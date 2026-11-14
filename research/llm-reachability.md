# How can a Hugging Face–hosted JS app reach the LLM?

Research for [#2](https://github.com/spowers42/reachy-emt-history/issues/2) (map: #1). Researched 2026-11-14 against primary sources: Pollen/HF source code, official docs, and live probes of the endpoints (CORS preflights run with `curl` on 2026-11-14). Anything not verified is marked **Unverified**.

## Context: what the app actually is

- A Reachy Mini JS app is a **static** page on a Hugging Face Space (`sdk: static`); there is no app server. ([APP_CREATION_GUIDE §11][guide], [javascript-sdk.md][jssdk])
- The browser reaches the robot over **WebRTC** (signaling via `pollen-robotics-reachy-mini-central.hf.space`, then peer-to-peer). JS apps "need internet and an HF account" and "run only while a browser tab is open". ([AGENTS.md][agents], [javascript-sdk.md][jssdk])
- Audio: the robot mic arrives as a `MediaStreamTrack` in the browser (`audioTrack` event); the app pushes audio to the robot speaker with `setOutgoingAudioTrack(track)`. So **the browser is the audio hub**, and whatever does STT/LLM/TTS has to be reachable from the browser. ([javascript-sdk.md][jssdk], [minimal-conversation README][mc-readme])
- Static-Space **secrets are not secret**: "For Static Spaces, both [variables and secrets] are available through client-side JavaScript in `window.huggingface.variables`." Docker/Gradio Spaces (which can hold secrets server-side) "require a paid plan to create: PRO". ([HF Spaces overview][spaces])
- The host shell signs the user in with HF OAuth and seeds the token in `sessionStorage.hf_token`. Default scopes are only `openid profile`; extra scopes (e.g. `inference-api`) are added with `hf_oauth_scopes`. ([guide §11.1][guide], [HF Spaces OAuth][oauth])

## Feasibility table

| Backend | Reachable from the HTTPS Space? | Auth / key handling | Model choice & system prompt | Cost | Verdict |
|---|---|---|---|---|---|
| **A. Pollen/HF realtime backend ("Hugging Voice", deployed)** — what `tfrere/minimal-conversation` and `reachy_mini_conversation_app` use | **Yes, verified.** Browser `fetch` POST to allocator `https://pollen-robotics-reachy-mini-realtime-url.hf.space/session` → `connect_url` → `wss://…/realtime`. Allocator CORS reflects any Origin (probed with arbitrary origins); no origin allowlist found in allocator source. WS-handshake Origin check: **Unverified**. | Visitor's HF OAuth token as `Bearer`; no other key. Without a token: `401 {"code":"verified_hf_token_required"}`. | **Instructions + tools: yes** (`session.update` with `instructions`, `tools`, `voice`). **Model: no** — "the model is chosen server-side by the session allocator"; client `model` param is stripped. Voices: fixed list (default `Aiden`). Allocator may A/B between two backends. | Free to the user as far as documented (no key needed). Quotas/limits: **Unverified**. | **Best default for the spoken Patient.** Zero infra, works today. Cannot pick MedGemma. |
| **B. Self-hosted, voice: HF open-source `speech-to-speech` server** (`speech-to-speech serve`, same protocol as A; LLM stage can point at any OpenAI-compatible endpoint) on a LAN box, LLM = vLLM / llama.cpp via `--llm_backend chat-completions` | Only if exposed as **`wss://` with a browser-trusted cert** (see "LAN from an HTTPS page" below). Plain `ws://192.168.x.x` from an HTTPS page is mixed content. | Server has **no auth of its own** ("performs no authentication and no throttling"), so put it behind a gateway or keep it on a trusted network. | **Full control**: any served model (e.g. MedGemma via vLLM), any prompt. Same client code as A; minimal-conversation already has a `local` mode (`VITE_HF_REALTIME_CONNECTION_MODE=local` + `VITE_HF_REALTIME_WS_URL`), though as a build-time setting — we would make it a runtime setting. | Your hardware. | **Supported alternative** for MedGemma / offline-ish use. Needs a TLS story. |
| **C. Self-hosted, text only:** vLLM / llama.cpp OpenAI-compatible `/v1/chat/completions` called straight from the browser | Same TLS caveat as B. CORS: **llama.cpp** reflects any Origin by default (`--cors-origins` default `*`). **vLLM** "Cross-origin browser access is disabled by default" — must pass `--allowed-origins` with the Space origin. | `--api-key` on either server; the key is then in the browser (fine for the owner on their own LAN). | Full control. | Your hardware. | **Feasible** for Case generation / Debrief text. Doesn't do voice by itself. |
| **D. HF Inference Providers router** `https://router.huggingface.co/v1/chat/completions` (OpenAI-compatible) | **Yes, verified**: CORS `Access-Control-Allow-Origin: *`. | Visitor's HF OAuth token, if the Space adds `hf_oauth_scopes: [inference-api]`. No secret in the page. | Many models. **`google/medgemma-27b-text-it` is live** (provider: featherless-ai) as of 2026-11-14; `medgemma-4b-it`/`27b-it` have no provider. MedGemma is gated (`gated: auto`); whether gating matters for routed calls is **Unverified**. | **Billed to the visitor.** Free users get **no** monthly credits (must buy credits); PRO gets $2/month. | **Good text path**, but it puts a cost on each Trainee. |
| **E. Frontier API + user's own key, from the browser** (OpenAI, Anthropic, Gemini, OpenRouter) | **Yes, CORS verified** for all four (preflight 200/204 with allow-origin). Anthropic needs the `anthropic-dangerous-direct-browser-access: true` header (sent by the SDK's `dangerouslyAllowBrowser`). OpenAI's SDK needs `dangerouslyAllowBrowser` too. | Key lives in the browser. **Only acceptable as BYO-key** (user pastes their own key; keep it in `sessionStorage`/memory). Never ship a key as a Space secret: static-Space secrets are public. | Full model choice; any prompt. | User's own account. | **Feasible as an opt-in BYO-key setting.** Not a default. |
| **F. Frontier API + key held server-side** (Docker Space or another proxy holding the secret) | Yes (same origin or CORS you control). | Key safe server-side; you then need your own abuse controls (e.g. check HF OAuth identity). | Full. | Your account pays for everyone; Docker Space needs PRO. | Feasible but adds a server, cost, and abuse surface. **Not recommended for v1.** |

## LAN from an HTTPS page: the details (B and C)

- **Mixed content.** An HTTPS page cannot normally load `http://`/`ws://`. Chrome's **Local Network Access (LNA)** permission, launched in Chrome 142, exempts requests "known to be local" from mixed-content blocking when the host is a private IP literal, a `.local` name, or `fetch(…, {targetAddressSpace: "local"})`, and the user approves the prompt. ([Chrome LNA blog][lna-blog], [WICG LNA explainer][lna-explainer])
- **WebSockets:** the Chrome post says WebSocket connections to the local network "are not yet gated on the LNA permission"; the explainer says they *should* behave like fetch. Whether `ws://192.168.x.x` from an HTTPS page is **mixed-content exempt** in current Chrome is **Unverified**. Assume it is blocked.
- **Iframes:** LNA is limited to top-level secure documents by default; subframes need the `local-network-access` permission-policy feature delegated to them. ([explainer][lna-explainer]) The Reachy app runs inside the host shell's same-origin iframe, and HF's `huggingface.co/spaces/...` page wraps the Space in another (cross-origin) iframe. Whether either iframe delegates LNA is **Unverified**. Opening the `*.static.hf.space` URL directly avoids the HF wrapper, but not the host-shell iframe.
- **Other browsers:** LNA is a Chrome feature. Pollen says JS apps should be mobile-first ([AGENTS.md][agents]), and Safari/iOS are not covered by LNA. **Unverified** for Chrome Android.
- **CORS still applies** to normal cross-origin requests; LNA only replaces the old PNA preflight. Set `--allowed-origins` (vLLM) or `--cors-origins` (llama.cpp) to the Space origin.
- **Robust option:** give the LAN server a **real HTTPS/WSS endpoint with a publicly trusted certificate**, so mixed content and LNA are not involved. Options: a reverse proxy (Caddy/nginx) with a Let's Encrypt DNS-01 cert on a domain you own; Tailscale HTTPS (`*.ts.net`, still only reachable on the tailnet); or a tunnel (Cloudflare Tunnel / ngrok). A tunnel exposes the server publicly, so add auth because `speech-to-speech` and keyless llama.cpp have none. **Self-signed certs** only work after each device manually trusts the cert, so treat them as a dev-only option.

## Facts about Pollen's conversation stack (backend A/B)

- It is Hugging Face's open **`speech-to-speech`** engine (Apache-2.0): a VAD → STT → LLM → TTS cascade with an **OpenAI-Realtime-compatible** `/v1/realtime` WebSocket. Pollen calls it "Hugging Voice". ([Pollen blog][pollen-blog], [HF blog][hf-blog], [s2s realtime README][s2s-rt])
- Realtime events we can rely on: `session.update` deep-merges `instructions`, `tools`, `voice`, and turn detection; `conversation.item.create` injects `input_text` or `function_call_output` without triggering generation; `response.create` accepts **per-response `instructions`** and `tool_choice` overrides. The README also mentions text replies (`output_text`). ([s2s realtime README][s2s-rt], [session.py][s2s-session]) These map well to our needs: a Patient persona prompt, an "end interview" tool call, and switching to Instructor via a new `instructions`. Text-only output against the *deployed* backend: **Unverified**.
- "Models are chosen on the server, not by the client." ([s2s realtime README][s2s-rt]) Self-hosted LLM backends: `responses-api` (`/v1/responses`) or `chat-completions` (`/v1/chat/completions`), configured with `--responses_api_base_url`, `--responses_api_api_key`, and `--model_name`. Local defaults are Qwen3.5-4B (transformers/MLX). `serve` binds `127.0.0.1` unless you pass `--host 0.0.0.0`; the default port is 8765. ([s2s LLM README][s2s-llm], [s2s README][s2s])
- `reachy_mini_conversation_app` (Python) uses the same backend: `HF_REALTIME_CONNECTION_MODE=deployed|local` and `HF_REALTIME_WS_URL`, with prompts in `profiles/<name>/profile.md`. It is a Python app, so it is not directly reusable in our JS app, but it confirms the protocol. ([conversation app][rca])
- **Can a new third-party Space use backend A? (origin allowlist)** The minimal-conversation README warns that the session fails "if … your origin is not in its CORS allow-list", and its source says the allocator "must therefore CORS-allow this Space's origin (the deployed pollen allocator already does for the mobile/website origins)". ([mc-readme][mc-readme], [mc-rt][mc-rt]) The evidence points to **no origin allowlist on the HTTP step today**:
  - The allocator's `app.py` contains no CORS middleware and no Origin check.
  - It forwards only `User-Agent`, `Content-Type`, `Accept`, `Authorization`, `X-Reachy-Mini-Authorization`, and `X-Forwarded-For` upstream, so the upstream allocator never sees `Origin`. ([allocator app.py][alloc-app])
  - Live probes on 2026-11-14 got `Access-Control-Allow-Origin` echoed for every origin tried: `https://someone-else-app.static.hf.space`, `https://example.com`, and `http://192.168.1.50:5173`. This held for both the preflight and the actual `POST` (which returned 401 without a token), so the echoing apparently comes from the HF Spaces proxy layer.
  - The gate that remains is the **HF token**: a missing token gets `verified_hf_token_required`.
  - **Unverified:** (i) whether the per-session realtime WebSocket checks the `Origin` handshake header (WebSockets have no CORS, but a server can reject on Origin). (ii) Whether Pollen intends third-party use, or may add an allowlist later; their own README describes one. We did not run an authenticated end-to-end session from a new origin, because that needs a real HF token. **First prototype step:** deploy a stub Space and open a session. If it fails, backend B is the fallback, using the same client code.
- **Model / system prompt on backend A:** the system prompt is **settable** (`instructions` in `session.update`, plus per-response `instructions` on `response.create`). The model is **not**: it is chosen server-side, and the client strips a `model` query param from the connect URL. ([mc-rt][mc-rt], [s2s-rt][s2s-rt])
- Which LLM the *deployed* backend runs, and its rate limits, are **not documented**. The allocator README says to keep backend URLs out of the repo and shows A/B splitting. ([allocator README][alloc])

## Recommended default path

1. **Patient voice loop → backend A (deployed HF realtime)**, copying `src/huggingface-realtime.ts` from `tfrere/minimal-conversation`. Benefits: no infra, no keys, honors our system prompt and tools, and Pollen already tests it on Reachy. The Case (frozen at Session start) is passed in `instructions`.
2. **Hide the backend behind one setting: the realtime WebSocket URL.** "Deployed" (default) or "Custom `wss://` URL" pointing at a self-hosted `speech-to-speech serve` backed by vLLM/llama.cpp (e.g. MedGemma). This makes backend B a configuration change, not a code fork. Document a TLS recipe (Tailscale HTTPS or a reverse proxy with a real cert) instead of relying on Chrome LNA.
3. **Case generation and Debrief:** these are one-shot, longer, structured text tasks where model quality matters most. Two candidates, to be settled by the model-quality ticket:
   - (a) do them in the same realtime session (text response with per-response `instructions`). No extra backend, but we can't choose the model on A.
   - (b) a separate OpenAI-compatible **text** endpoint selectable in settings: HF router with OAuth `inference-api` (D), a LAN vLLM/llama.cpp (C), or a BYO frontier key (E).
   v1 suggestion: start with (a), and build the text-endpoint interface as an OpenAI-compatible `chat/completions` client so C/D/E all plug in.
4. **Do not** ship any API key as a Space secret. A server-side proxy (F) is out of scope for v1.

## Open questions surfaced

- Deployed HF realtime backend: which LLM, rate/usage limits, terms for third-party apps, and what makes a token "verified" (`verified_hf_token_required`)? Ask Pollen.
- Does the deployed backend reliably do text-only responses and long single-turn generations (Case JSON, Debrief)? Needs a quick prototype.
- Can the deployed STT handle EMS vocabulary (drug names, OPQRST terms)? This affects the model-quality ticket.
- LNA inside the host-shell iframe and the HF wrapper, and `ws://` mixed-content behavior: only worth testing if we want plain-HTTP LAN servers. Otherwise require `wss://`.
- Transcript privacy: with A, D, or E, Trainee speech and transcripts leave the LAN for HF or a provider. This feeds the privacy ticket.

[agents]: https://github.com/pollen-robotics/reachy_mini/blob/main/AGENTS.md
[guide]: https://github.com/pollen-robotics/reachy_mini/blob/main/ts/APP_CREATION_GUIDE.md
[jssdk]: https://github.com/pollen-robotics/reachy_mini/blob/main/docs/source/SDK/javascript-sdk.md
[mc-readme]: https://huggingface.co/spaces/tfrere/minimal-conversation/blob/main/README.md
[mc-rt]: https://huggingface.co/spaces/tfrere/minimal-conversation/blob/main/src/huggingface-realtime.ts
[alloc]: https://huggingface.co/spaces/pollen-robotics/reachy-mini-realtime-url/blob/main/README.md
[alloc-app]: https://huggingface.co/spaces/pollen-robotics/reachy-mini-realtime-url/blob/main/app.py
[rca]: https://github.com/pollen-robotics/reachy_mini_conversation_app
[s2s]: https://github.com/huggingface/speech-to-speech
[s2s-rt]: https://github.com/huggingface/speech-to-speech/blob/main/src/speech_to_speech/api/openai_realtime/README.md
[s2s-session]: https://github.com/huggingface/speech-to-speech/blob/main/src/speech_to_speech/api/openai_realtime/handlers/session.py
[s2s-llm]: https://github.com/huggingface/speech-to-speech/blob/main/src/speech_to_speech/LLM/README.md
[hf-blog]: https://huggingface.co/blog/local-reachy-mini-conversation
[pollen-blog]: https://pollen-robotics.com/reachy-mini/blog/open-realtime-voice/
[spaces]: https://huggingface.co/docs/hub/spaces-overview
[oauth]: https://huggingface.co/docs/hub/spaces-oauth
[ip-pricing]: https://huggingface.co/docs/inference-providers/pricing
[lna-blog]: https://developer.chrome.com/blog/local-network-access
[lna-explainer]: https://github.com/WICG/local-network-access/blob/main/explainer.md
[vllm]: https://docs.vllm.ai/en/latest/cli/serve.html
[llamacpp]: https://github.com/ggml-org/llama.cpp/blob/master/tools/server/README.md
[anthropic-sdk]: https://github.com/anthropics/anthropic-sdk-typescript/blob/main/src/client.ts
[openai-sdk]: https://github.com/openai/openai-node/blob/main/src/client.ts

Additional sources: minimal-conversation transport code ([huggingface-realtime.ts][mc-rt]); HF Inference Providers pricing ([pricing][ip-pricing]); vLLM `serve` flags ([vllm][vllm]); llama.cpp server CORS section ([llama.cpp][llamacpp]); browser flags in the Anthropic ([client.ts][anthropic-sdk]) and OpenAI ([client.ts][openai-sdk]) SDKs.

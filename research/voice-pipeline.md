# Voice pipeline for a Reachy Mini JS app

Research for issue #5, "How does a Reachy Mini JS app do spoken conversation?" (map: #1).

Researched 2026-11-14 against these source snapshots:

| Source | Revision |
|---|---|
| `pollen-robotics/reachy_mini` (SDK, daemon, docs) | `caa919e` (2026-11-13); npm `latest` = `1.14.0` |
| `tfrere/minimal-conversation` HF Space | HEAD at time of clone; pins `@pollen-robotics/reachy-mini-sdk@1.11.0` |
| `pollen-robotics/reachy_mini_conversation_app` | `e901925` (2026-11-04) |
| `huggingface/speech-to-speech` | `d1dc31d` (2026-11-13) |
| W3C Web Speech API draft | https://webaudio.github.io/web-speech-api/ |

The tables below use these short names for source files:

- **SDK** means `reachy_mini/ts/lib/reachy-mini.ts`.
- **JS-SDK doc** means `reachy_mini/docs/source/SDK/javascript-sdk.md`.
- **Guide** means `reachy_mini/ts/APP_CREATION_GUIDE.md`.
- **MC** means the `tfrere/minimal-conversation` source.
- **RMCA** means `reachy_mini_conversation_app`.
- **S2S** means `huggingface/speech-to-speech`.

---

## 1. The audio plumbing

The Reachy Mini daemon owns the mic and speaker. A browser app reaches them over a WebRTC peer connection that goes directly to the robot. The browser decodes Opus audio locally. Sources: `media-architecture.md`, "Unified Architecture" and "Web Access".

There is one bidirectional audio transceiver:

- **Robot mic to browser.** This arrives as a remote `MediaStreamTrack`, available from the SDK in three ways:
  - `robot.robotAudioTrack`
  - the `audioTrack` event
  - `handle.media.robotStream`, for apps embedded in the host shell

  Source: JS-SDK doc, Properties and Events tables; Guide §5 `RobotMedia`.
- **Browser to robot speaker.** Call `robot.setOutgoingAudioTrack(track)`, which runs `replaceTrack` on the audio sender. The track survives auto-reconnects. The call returns `false` if the robot offered no two-way audio (`micSupported`). Source: SDK, `setOutgoingAudioTrack` / `_applyOutgoingAudioTrack`.
- **Other playback paths.** These are not suited to streaming TTS:
  - `playSound(file)` plays a named file on the daemon. Source: SDK `playSound`, daemon `backend/abstract.py play_sound`.
  - `uploadAudio(blob)` + `playUploadedAudio(id)` / `cancelAudio()` play a whole clip, which must be 16 kHz mono 16-bit PCM WAV. The daemon does not transcode it. Source: JS-SDK doc, "Audio format".
  - `playMove(motion, { audioBlob })` plays a clip in sync with a motion on the daemon clock.

  These are whole-clip uploads over the data channel, so the full TTS audio has to exist before playback starts. That is fine for canned lines such as "let me come up with a scenario for you". For conversational replies, streaming through `setOutgoingAudioTrack` is the path.
- **Barge-in flush.** `robot.clearIncomingAudio()` sends `clear_incoming_audio`. The daemon then flushes the audio it has queued for the speaker, described in the daemon as "Flush incoming WebRTC audio queued for the speaker (barge-in)". Source: SDK `clearIncomingAudio`; `daemon/backend/abstract.py clear_incoming_audio`.
- **Mic tuning.** `robot.applyAudioConfig([...])` writes XVF3800 audio-board parameters (Wireless only). Pollen's conversation apps all apply the same tuned set at session start (AGC max gain, noise-suppression floors, and so on). Source: MC `src/audio-startup-config.ts`, which says it is kept in lockstep with RMCA `audio/startup_config.py`.
- **Echo cancellation.** The Reachy Mini audio card (XMOS) does AEC in hardware. The daemon adds software AEC (`webrtcdsp`) only on the fallback `autoaudiosrc` path, where there is no Reachy card. So on a Wireless, the mic track the browser receives should already have the robot's own speaker output removed. Source: `media/media_server.py _configure_audio`. I have not verified how well this works in practice.
- **Daemon-side buffering.** These settings shape latency:
  - The robot-side receive jitter buffer for the browser-to-speaker leg is set to 300 ms. The code comment says the default of 200 ms "underruns on a jittery Wi-Fi link".
  - The mic-to-browser leg has Opus in-band FEC enabled, budgeting for 20% packet loss.

  Source: `media/media_server.py`, `RX_JITTER_LATENCY_MS = 300` and `TX_OPUS_FEC_LOSS_PERC = 20`.
- **Signaling needs the internet.** The central signaling server is a Hugging Face Space, and the Guide lists "Offline mode" as unsupported ("The central is required for every session"). A JS app therefore always needs internet for session setup, even if every speech model runs on the LAN. After setup, media flows peer to peer. Sources: Guide, "What's not supported"; `media-architecture.md`.

> Do not use `getUserMedia()` to read robot audio, because it opens the laptop's own mic. Source: `AGENTS.md`, "Media flow".

---

## 2. STT options: where the robot mic track can go

| Option | How it works | Local/offline? | Notes |
|---|---|---|---|
| **A. HF realtime backend** (Pollen's default; used by MC) | Resample the mic track to 16 kHz PCM16 in Web Audio. Stream it as base64 `input_audio_buffer.append` events over a WebSocket that speaks the OpenAI Realtime protocol. The server runs VAD, STT, the LLM and TTS. | **Cloud** by default. The `pollen-robotics-reachy-mini-realtime-url.hf.space/session` allocator, authorized with the HF token, returns a per-session `connect_url`. | The model is chosen server-side, so the app sends no model name. The app also does not control which LLM plays the Patient. Source: MC `huggingface-realtime.ts`, `resolveHfRealtimeWebSocketUrl`; MC README, "Authentication" and "Settings". |
| **B. Self-hosted `speech-to-speech` server** (same protocol as A) | The same client code as A, with `VITE_HF_REALTIME_CONNECTION_MODE=local` and `VITE_HF_REALTIME_WS_URL=ws://host:8765/v1/realtime`. | **LAN / local.** The default pipeline is Silero VAD → Parakeet TDT → Qwen3.5-4B → Qwen3-TTS. The README says it "runs entirely on your computer… no API key". | The LLM stage can be pointed at **any OpenAI-compatible server** (llama.cpp, vLLM, a hosted provider) with `--llm_backend chat-completions` and `--responses_api_base_url …`. STT and TTS are also swappable: Whisper, faster-whisper, Kokoro, Pocket TTS, or OpenAI-compatible `/v1/audio/*` endpoints. Sources: S2S README, "Realtime API", the backend table, and "LLM backends"; MC `huggingface-realtime.ts` env config; RMCA README, "Configuration". |
| **C. Browser Web Speech API with the track** | `new SpeechRecognition().start(robotMicTrack)` | **Depends.** With `processLocally = false` (the default), the spec says speech "might be sent to a remote server". With `processLocally = true`, recognition MUST happen on the device; if it can't, the browser fires `service-not-allowed`. `SpeechRecognition.available({langs, processLocally})` and `install()` manage on-device language packs. | The Pollen docs say `start(track)` needs Chrome 135+. I could not confirm which Chrome version shipped `processLocally` (a blink-dev post refers to it as "launched last year", meaning 2025). The spec supports `phrases` contextual biasing with a 0–10 boost, which could help with medical vocabulary, though browsers may reject it with `phrases-not-supported`. This option is effectively Chromium-only, and gives only STT, so it must be paired with separate LLM and TTS stages. Sources: JS-SDK doc, `audioTrack` event; `AGENTS.md`, "Media flow"; W3C Web Speech API draft, SpeechRecognition IDL. |
| **D. Client-side ASR model** (for example transformers.js with Whisper or Moonshine) | Capture the track with Web Audio or `MediaRecorder`, segment it with a VAD, and run ASR in the page through WASM or WebGPU (`device: 'webgpu'`). | **Fully local** in the browser after the model download. | transformers.js lists `automatic-speech-recognition` and Moonshine as supported, and describes WebGPU as "still experimental in many browsers". You have to build VAD, endpointing and barge-in yourself. Performance on the companion device is unmeasured. Sources: `AGENTS.md` ("a client-side ASR model"); transformers.js docs index. |

---

## 3. TTS options: getting speech out of the robot speaker

| Option | Path to speaker | Local? | Notes |
|---|---|---|---|
| **Realtime backend audio** (A or B above) | `response.output_audio.delta` PCM16 at 16 kHz goes to an `AudioBufferSourceNode` schedule, then into a `MediaStreamAudioDestinationNode` track. MC then calls `replaceTrack` on the robot's audio sender; the public equivalent is `setOutgoingAudioTrack`. | Cloud (A) or LAN (B) | Streams chunk by chunk, so the first audio plays before the whole reply has been synthesized. The voice list on the deployed backend is fixed: Aiden (default), Ryan, Dylan, Eric, Ono_Anna, Serena, Sohee, Uncle_Fu, Vivian. Source: MC `PcmOutputTrack`, `routeRealtimeToRobot`, `HF_AVAILABLE_VOICES`. |
| **Browser `speechSynthesis`** | **None.** The spec's `SpeechSynthesis` and `SpeechSynthesisUtterance` IDL has no MediaStream, track or output-device member, so the audio comes out of the companion device's speaker, not Reachy's. | Varies by browser and OS | Unsuitable if the voice must come from the robot. Source: W3C Web Speech API draft, SpeechSynthesis IDL. |
| **Client-side TTS model** (for example transformers.js `text-to-speech`, Kokoro) | Synthesize PCM, play it through Web Audio into a `MediaStreamDestination`, then `setOutgoingAudioTrack(track)`. | Fully local | Same streaming path as the realtime option. CPU/GPU cost in the browser is unmeasured. Source: transformers.js docs index; SDK `setOutgoingAudioTrack`. |
| **Cloud or LAN TTS over HTTP** (OpenAI-compatible `/v1/audio/speech`, and others) | Fetch audio, decode it, then either use the Web Audio → `setOutgoingAudioTrack` path, or for whole clips use `uploadAudio` + `playUploadedAudio` (which needs 16 kHz mono WAV). | Either | The whole-clip path adds upload time before playback starts. |

---

## 4. Turn-taking and barge-in

**How minimal-conversation does it.** Source: MC `buildHfSessionConfig` and `handleMessage`.

- It sends a session config with `turn_detection: { type: "server_vad", interrupt_response: true }`, so the server decides when the user's turn has ended and cancels the reply when the user talks over it.
- A client state machine maps server events to UI and motion:
  - `input_audio_buffer.speech_started` → user-speaking
  - `speech_stopped` / `response.created` → processing
  - first `output_audio.delta` → ai-speaking
  - `response.done` plus a 250 ms drain pad → listening
- It does **not** call `clearIncomingAudio()` or stop audio already scheduled in the browser when the user starts speaking. On barge-in it relies on the server to stop sending deltas. Audio that is already queued still plays out, both in the browser schedule and in the daemon's 300 ms jitter buffer. This is my reading of the code; I have not tested it on hardware.
- It also keeps the robot's head wobbling and antennas moving in time with the voice output, through `head-wobbler.ts` and `antennas.ts`.

**How the Python conversation app does it.** This is more robust. Source: RMCA `audio/local_vad.py`, `huggingface_realtime.py _interrupt_playback`, `console.py clear_audio_queue`.

- It runs **local Silero VAD** (ONNX, 16 kHz, 512-sample chunks) alongside server VAD. Settings: 160 ms of speech before it counts, 400 ms of silence to end a turn, and a 64 ms barge-in window.
- When a barge-in looks likely, it **ducks** playback gain to 0.25.
- On a confirmed interruption, it sets gain to 0, sends `response.cancel`, drops the queued output, and calls the media layer's `clear_player`. In the WebRTC client, that flushes the send chain and then POSTs `/api/media/clear_incoming_audio`.
- `LOCAL_VAD_ENABLED` and `LOCAL_BARGE_IN_ENABLED` both default to on.

**How a JS app could do the same:**

1. On `speech_started` (or a local VAD hit), stop or disconnect the scheduled `AudioBufferSourceNode`s.
2. Call `robot.clearIncomingAudio()`.
3. Send `response.cancel`, and optionally `conversation.item.truncate`. S2S supports both.

Silero VAD can also run in the browser through onnxruntime-web, but I have not checked a specific package.

**What this app needs on top.** The Interview ends only when the Trainee says an ending phrase or presses the in-app button (map #1). With server VAD, every pause triggers an LLM reply. So:

- Ending-phrase detection has to run on user transcripts. The `conversation.item.input_audio_transcription.completed` event provides them, and MC already surfaces it as a `transcript` event.
- Tool calls are another route: MC already wires `response.function_call_arguments.done` to tools.

---

## 5. Latency

- **Instrumented, but no published numbers.** S2S records per-turn timings: `vad_decision`, `stt`, `llm`, `tts_ttfa` (time to first audio), `e2e` and `hold`. It attaches them to `response.done` metadata under `speech_to_speech.turn_latency`. RMCA parses this metadata and can show it in its UI (Settings → Developer options → Show response latencies). S2S has a coverage guide in `docs/response-latency.md`. I found **no published end-to-end figures** in these sources, so the numbers need to be measured on our setup. Sources: RMCA `_parse_turn_latency`; S2S README, "Realtime API".
- **Fixed costs I could identify on the JS path:**
  - MC's input `ScriptProcessorNode` buffer of 4096 samples, which is about 85–93 ms at 44.1/48 kHz.
  - MC's 40 ms output start lead.
  - The daemon's 300 ms receive jitter buffer on the speaker leg.
  - Network round trips: browser → backend → browser → robot. With a cloud backend, the audio crosses the WAN twice per turn, plus the Wi-Fi hops to the robot.
- **Turn-end wait.** Server VAD silence detection adds time before every reply, set by S2S `--thresh` and its silence settings.
- **LLM choice.** S2S's README example notes "reasoning disabled for low voice latency" for Gemma via the HF router. A reasoning model playing the Patient would add noticeable delay.

---

## 6. Local/offline vs cloud, by piece

| Piece | Cloud | LAN / self-hosted | In-browser |
|---|---|---|---|
| Session signaling | **Required** (HF central) | — | — |
| Mic/speaker transport | — | WebRTC peer-to-peer to the robot | — |
| VAD | HF realtime (server VAD) | S2S Silero / FireRed | Possible (Silero ONNX); not provided by the SDK |
| STT | HF realtime; Chrome Web Speech with `processLocally=false` | S2S: Parakeet, Whisper, faster-whisper, Qwen3-ASR, … | Chrome Web Speech with `processLocally=true`; transformers.js Whisper/Moonshine |
| LLM (Patient/Instructor) | HF realtime (server-chosen); a frontier API through S2S `chat-completions` | S2S → llama.cpp / vLLM (for example MedGemma) | — (not considered) |
| TTS | HF realtime voices; hosted TTS APIs | S2S: Qwen3-TTS, Kokoro, Pocket, … | transformers.js / Kokoro → `setOutgoingAudioTrack` |

Implications for the map:

- The **only path Pollen already ships** is option A (HF realtime). It is cloud-based, the LLM is chosen server-side, and the voice list is fixed. That conflicts with the map's goal of choosing our own LLM (MedGemma or a frontier model).
- **Option B, the S2S server on a LAN or remote GPU host**, keeps MC's client code almost unchanged and lets us plug in an OpenAI-compatible LLM. It is the most direct way to get "our own LLM" with voice.
- **Options C and D** keep STT in the browser, but then the app has to build its own STT → LLM → TTS orchestration and turn-taking.

---

## 7. Caveats and discrepancies found

- **MC uses private SDK internals.** It reaches into `robot._pc` (`getReceivers` / `getTransceivers` / `replaceTrack`). The current SDK has public alternatives: `robotAudioTrack` / `handle.media.robotStream` and `setOutgoingAudioTrack`. MC pins SDK `1.11.0`. Checking the published type definitions on unpkg: `1.11.0` has `clearIncomingAudio` but not `setOutgoingAudioTrack` or `robotAudioTrack`, while `1.14.0` (npm `latest` on 2026-11-14) has all three. That explains why MC uses the private fields. The Guide's migration note still says `enableMicrophone: true` on `mountHost` enables bidirectional audio, but the JS-SDK doc says `enableMicrophone` is deprecated and ignored. Trust the JS-SDK doc and SDK source.
- **CORS on the allocator may block a new Space.** MC's README says the HF realtime allocator must CORS-allow the calling Space's origin, and that the orb errors "if… your origin is not in its CORS allow-list". **It is unconfirmed whether a new third-party Space origin is allowed.**
- **Deployed Spaces must be public.** MC's README says private static Spaces fail to load in the HF iframe wrapper because the `hf_jwt` cookie is blocked by its SameSite policy.
- **Python barge-in has not been ported to JS.** RMCA is a Python app that runs on the robot. Its local-VAD and barge-in logic is the reference behavior, but it does not exist in JS yet.

# Model candidates for Patient, Case generation, and Debrief

Research for issue #3 (map #1). Researched 2026-11-14. Every claim cites a primary source: a model card, official docs, a paper, or measured benchmark data. Where a value is my own estimate, it says so.

## TL;DR

- **MedGemma can be self-hosted, but it isn't the right model to drive the spoken Patient on Strix Halo.** Its best text model is a 27B dense Gemma 3 model. Dense ~30B models decode at only ~11 tok/s at Q4 on Strix Halo. Google also says MedGemma "has not been evaluated or optimized for multi-turn applications." It fits better in the **Case generation** and **Debrief** roles, which tolerate latency and benefit most from medical knowledge.
- **On Strix Halo, MoE models are the right choice for the Patient.** The measured llama.cpp speeds are ~55 tok/s for Gemma 4 26B-A4B (Q4), ~60 tok/s for Qwen3.6-35B-A3B (Q4) and ~57 tok/s for gpt-oss-120b (MXFP4). The Patient's job is staying in character, not medical reasoning, so a strong general model is the right tool there.
- **Strix Halo works today.** llama.cpp runs on both Vulkan (RADV) and ROCm, and Vulkan is the most stable. vLLM runs too, but only through community-maintained builds and with gaps (no native FP8). Use **llama.cpp's OpenAI-compatible server**. vLLM is the alternative to try later.
- **Frontier APIs are the low-risk baseline**, for Debrief grading especially. Claude Haiku 5.5 or Sonnet 5.5 cost cents per Session (estimate below). The Case and transcript leave the box, but they are synthetic, low-stakes data.

## The three jobs and what each needs

| Job | Latency sensitivity | What matters most |
|---|---|---|
| **Patient** (multi-turn spoken role-play) | **High.** Every Trainee question waits on it. Replies are short (~20–80 tokens). | Staying in character, revealing only what is asked, never volunteering the diagnosis, consistency with the frozen Case. Medical knowledge comes from the Case, not the model. |
| **Case generation** (once per Session) | Medium. Reachy covers the wait ("let me come up with a scenario…"), but more than ~30 s gets awkward. | Medical plausibility (vitals vs condition, meds vs history), structured (JSON) output, variety, Environment and Provider Level conditioning. |
| **Debrief** (once per Session) | Low to medium. Can be streamed or spoken incrementally. | Grading against SAMPLE/OPQRST and NREMT expectations, Differential critique, judgment, long-context reading of the transcript. |

## Comparison table

Speeds are llama.cpp `llama-bench` results on a Ryzen AI Max+ 395 (Fedora 43, kernel 6.19, llama.cpp b9187/9193, May 2026) from kyuz0's Strix Halo toolboxes dataset [S1]. **tg** is single-stream generation (tg128); **pp** is prompt processing (pp512). "@32k" is the value with 32k tokens already in context. MedGemma was not in that dataset, so its row is **my estimate** from the dense Gemma 4 31B row, scaled by file size (decode is memory-bandwidth-bound).

| Model | License | Size (total / active) | Context | Role-play / instruction-following | Medical evals | vLLM / llama.cpp, incl. Strix Halo | Strix Halo speed (llama.cpp, Vulkan RADV) |
|---|---|---|---|---|---|---|---|
| **MedGemma 27B text-it** (Gemma 3) | HAI-DEF terms (custom, gated) [M2] | 27B dense | 128K in, 8K out [M1] | Instruction-tuned. "Not been evaluated or optimized for multi-turn" [M1]. More prompt-sensitive than Gemma 3 [M3]. | MedQA 87.7 (0-shot), MedMCQA 74.2, MMLU-Med 87.0 [M1] | Gemma3 architecture: supported in vLLM and llama.cpp. Community GGUF exists (unsloth) [M4]. | **Estimate:** ~12 tok/s tg at Q4, ~6 at Q8; pp ~300 |
| **MedGemma 1.5 4B-it** (Gemma 3, multimodal) | HAI-DEF terms [M3] | 4B dense | 128K in, 8K out [M3] | Same multi-turn caveat [M3] | MedQA 69.1, MedMCQA 59.8 [M3] | Same as above. GGUF exists [M4]. | Fast (small), but weak on medical knowledge |
| **Gemma 4 26B-A4B-it** | Apache 2.0 [G1] | 25.2B / 3.8B MoE | 256K [G1] | Native system role, thinking toggle [G1]. Not yet tested for role-play (no IFEval published). | MedXpertQA-MM 58.1 [G1] | vLLM tested on gfx1151 [S2]. llama.cpp OK [S1]. | **tg 54.7 (Q4) / 45.5 (Q8)**, pp ~1300; tg 42 @32k |
| **Gemma 4 31B-it** | Apache 2.0 [G1] | 30.7B dense | 256K [G1] | As above | MedXpertQA-MM 61.3 [G1] | vLLM tested [S2]. llama.cpp OK [S1]. | tg 11.4 (Q4) / 6.3 (Q8), pp ~300 |
| **Qwen3.6-35B-A3B** | Apache 2.0 [Q1] | 35B / 3B MoE | 262K native [Q1] | Thinking on by default; `enable_thinking: false` turns it off [Q1]. No role-play evals published. | None on the card | vLLM ≥0.19, SGLang; llama.cpp via GGUF [Q1]. vLLM tested on gfx1151 [S2]. | **tg 60.4 (Q4) / 46.3 (Q8)**, pp ~1100; tg 49 @32k |
| **gpt-oss-120b** | Apache 2.0 [O1] | 117B / 5.1B MoE, MXFP4 | 131K [O2] | Reasoning effort low/med/high, always reasons [O1] | HealthBench 57.6, Hard 30.0 (high effort), "nearly matches o3" [O2] | vLLM, llama.cpp, Ollama [O1]. vLLM tested on gfx1151 at TP=1 [S2]. | **tg 56.6**, pp 720; tg 43 @32k, 35 @64k (59 GB) |
| **gpt-oss-20b** | Apache 2.0 [O1] | 21B / 3.6B MoE | 131K [O2] | As above | HealthBench 42.5, Hard 10.8 [O2] | As above | tg 79.8, pp 1690 |
| **Qwen3.5-122B-A10B** | Apache 2.0 [Q2] | 122B / 10B MoE | (see card) | General | None checked | AWQ tested in vLLM on gfx1151 [S2] | tg 22.3 (Q5), pp 310 (86 GB) |
| **Baichuan-M2-32B** (Qwen2.5 fine-tune) | Apache 2.0 [B1] | 32B dense | Not stated | Trained against a virtual-patient system, but plays the *doctor* [B1] | HealthBench 60.1, Hard 34.7 [B1] | vLLM ≥0.9, SGLang; GPTQ-4bit available [B1] | Not measured. A dense 32B is likely ~10 tok/s at Q4 (estimate). |
| **Baichuan-M3-235B** (Qwen3-235B-A22B fine-tune) | Apache 2.0 [B2] | 235B MoE (~22B active per its base) | Not stated | Trained for proactive **history taking** (SCAN-bench History Taking station) [B2] | HealthBench-Hard 44.4, #1 on SCAN-bench Clinical Inquiry (self-reported) [B2] | vLLM/SGLang; FP8, GPTQ-INT4 [B2] | INT4 is ~26% of BF16 ≈ ~120 GB [B2]: **does not realistically fit** in 128 GB alongside the OS and KV cache |
| **Claude Haiku 5.5 / Sonnet 5.5 / Opus 5.5** (API) | Commercial API | n/a | 1M [A1] | Strong instruction following, system prompts, structured outputs [A1] | Not checked here | n/a (hosted) | Hosted. Network RTT plus generation. |

Other frontier APIs (OpenAI GPT-5.x, Google Gemini; MedGemma is also on Vertex AI Model Garden [M5]) are viable through OpenAI-compatible or native SDKs. I did not research their current pricing or latency for this ticket.

## Strix Halo (Ryzen AI Max+ 395, Radeon 8060S, gfx1151) servability

- **ROCm officially supports gfx1151** on Linux, but only with KFD kernel fixes. You need Linux ≥ 6.18.4 (Fedora 43+, Ubuntu 26.04, or Ubuntu 24.04 HWE 6.17.0-19+). ROCm 7.2.x and 7.11–7.14 are listed as stable on those kernels, and 7.9/7.10 as unsupported [R1].
- **Memory:** by default the GPU can map only about 50% of RAM (GTT). AMD recommends keeping the BIOS VRAM carve-out small (e.g. 0.5 GB) and raising the TTM page limit (the `amd-ttm --set <GB>` tool from `amd-debug-tools`) [R1]. Without this, a 128 GB box can't load gpt-oss-120b (59 GB) plus KV cache comfortably.
- **llama.cpp:** works on both Vulkan (Mesa RADV) and ROCm. The toolbox maintainer recommends Vulkan RADV as "most stable and compatible… for all models" [S3]. In the measurements, Vulkan matches or beats ROCm on token generation for the quantized MoE models above, and ROCm wins prompt processing for BF16 [S1].
- **vLLM:** runs on gfx1151 through community toolbox images on stable ROCm. Tested models include Gemma 4 26B-A4B/31B, gpt-oss-20b/120b, Qwen3.6-35B-A3B (incl. AWQ) and Qwen3.5-122B AWQ [S2]. Caveats: there is **no native FP8** on gfx1151 (stock vLLM rejects FP8 checkpoints; an experimental Triton path costs ~1.5× BF16 memory), AITER MoE kernels are disabled on gfx1x, and current benchmarks for that image "have not yet been run" [S2]. Single-stream decode in vLLM was *slower* than llama.cpp in the historical numbers (Llama-3.1-8B BF16: 13.8 tok/s) [S2]. **For one Trainee at a time, llama.cpp's server is the better fit.** vLLM's strength is batching many users.

## Latency for spoken conversation

These are my estimates, built from the measured throughput above. Turn latency ≈ STT + (new-prompt tokens / pp) + (reply tokens / tg) + TTS start. llama.cpp keeps the KV cache between turns, so each turn only prefills the new Trainee utterance plus the previous reply (~100–200 tokens).

| Patient model on Strix Halo | ~50-token reply | Verdict |
|---|---|---|
| MoE ~3–5B active (Gemma 4 26B-A4B, Qwen3.6-35B-A3B, gpt-oss-120b @ low effort) | ~1 s generation (+ any reasoning tokens) | Good for conversation, especially with streamed TTS |
| Dense 27–31B at Q4 (MedGemma 27B, Gemma 4 31B) | ~4–5 s | Sluggish for back-and-forth |
| Dense 27–31B at Q8 | ~8 s | Too slow |

- **Turn off thinking for the Patient.** Gemma 4 and Qwen3.6 have off switches [G1][Q1]. gpt-oss always reasons, even at "low" [O1], which adds hidden tokens before the first spoken word.
- **Case generation** (~1–2k output tokens): ~20–40 s on a fast MoE; ~2–3 min on a dense 27B at Q4 (estimate). That is acceptable only if Reachy fills the time.
- **Debrief** reads the whole transcript (~3–8k tokens). That is a few seconds of prefill on MoE and ~10–30 s on dense models, plus generation. Streaming it into speech hides most of the wait.

## Frontier API cost (rough)

Anthropic list prices per 1M input/output tokens: Haiku 5.5 $0.10/$0.50, Sonnet 5.5 $2/$10, Opus 5.5 $4/$20 [A1]. A Session of ~20 Patient turns resending ~3k tokens each, plus a Debrief, is roughly 60–80k input and 3–5k output tokens. That works out to **under $0.01 on Haiku 5.5** and **~$0.15–0.20 on Sonnet 5.5** without prompt caching. Prompt caching cuts the input cost substantially. These are my estimates.

## Licensing notes

- **MedGemma (HAI-DEF terms):** weights carry a custom license. **Hosted API/web access counts as "Distribution"**, which brings obligations: pass the use restrictions and the agreement downstream, and seek regulatory authorization where applicable. Use must not risk Google being treated as a medical-device manufacturer. The terms neither mention nor forbid educational simulators [M2]. This is fine for a private trainer, but needs reading before any public HF Space that serves MedGemma.
- Gemma 4, Qwen3.6/3.5, gpt-oss, and Baichuan-M2/M3 are **Apache 2.0** [G1][Q1][Q2][O1][B1][B2].

## Recommended shortlist per job

**Patient**
1. **Gemma 4 26B-A4B-it** (self-hosted, Q4/Q8 GGUF, thinking off). Native system role, Apache 2.0, ~45–55 tok/s on Strix Halo.
2. **Qwen3.6-35B-A3B** (thinking off). Fastest measured, Apache 2.0.
3. **API:** Claude Haiku 5.5 or Sonnet 5.5 as the hosted baseline to compare role-play quality against.
- Not recommended: MedGemma 27B as the Patient. It is too slow on this box and has the multi-turn caveat.

**Case generation**
1. **gpt-oss-120b** (medium/high reasoning). Best measured health score among the models that fit (HealthBench 57.6) and fast on Strix Halo.
2. **MedGemma 27B text-it**. Best medical-QA scores and the user's preferred family. A run of ~2 min is acceptable only if Reachy covers the wait, or if Cases are pre-generated in the background.
3. **API:** Claude Sonnet 5.5 or Opus 5.5 with structured outputs for the Case JSON.

**Debrief**
1. **API: Claude Sonnet 5.5 or Opus 5.5.** Grading quality matters most here and latency least. It also gives a quality ceiling to compare local models against.
2. **gpt-oss-120b** (high reasoning), self-hosted.
3. **MedGemma 27B text-it** or **Baichuan-M2-32B** (HealthBench 60.1, Apache 2.0), both dense and slow but tolerable for a single Debrief. Baichuan-M3 is the only one trained specifically on history-taking evaluation, but it doesn't fit in 128 GB.

**Suggested starting configuration:** one llama.cpp server on Strix Halo with **Gemma 4 26B-A4B** for the Patient and **gpt-oss-120b** for Case and Debrief (~16–26 GB + 59 GB of weights; fits after raising the TTM limit). Keep an API key as a switchable backend so the Model-quality-evaluation work can compare them.

## Open questions surfaced

- None of these models publish **role-play or character-consistency evals**. The Patient choice has to come from our own Model quality evaluation (map: "Not yet specified").
- **MedGemma speed on Strix Halo is an estimate.** Running `llama-bench` on the actual box is a 10-minute test worth doing.
- **HAI-DEF terms** and a public HF Space: does serving MedGemma from the Space count as Distribution under the terms? That needs a decision if MedGemma stays in the plan.
- Does **Pollen's conversation API** (mentioned in the map) let us pick the model? I didn't research it here.
- Running two models at once (Patient plus Case/Debrief) on one box: with ~128 GB, check memory headroom for KV cache at long context.

## Sources

- [M1] MedGemma 27B text-it model card — https://huggingface.co/google/medgemma-27b-text-it
- [M2] Health AI Developer Foundations terms of use — https://developers.google.com/health-ai-developer-foundations/terms
- [M3] MedGemma 1.5 4B-it model card — https://huggingface.co/google/medgemma-1.5-4b-it
- [M4] Community GGUFs: https://huggingface.co/unsloth/medgemma-27b-text-it-GGUF, https://huggingface.co/unsloth/medgemma-1.5-4b-it-GGUF (architecture `Gemma3ForCausalLM`, via HF API)
- [M5] Google-Health/medgemma README (Hugging Face / Model Garden links) — https://github.com/Google-Health/medgemma ; MedGemma collection — https://huggingface.co/collections/google/medgemma-release-680aade845f90bec6a3f60c4 ; tech report arXiv:2507.05201
- [G1] Gemma 4 model card — https://ai.google.dev/gemma/docs/core/model_card_4 ; https://huggingface.co/google/gemma-4-26B-A4B-it
- [Q1] Qwen3.6-35B-A3B model card — https://huggingface.co/Qwen/Qwen3.6-35B-A3B
- [Q2] Qwen3.5-122B-A10B — https://huggingface.co/Qwen/Qwen3.5-122B-A10B (license via HF API)
- [O1] gpt-oss-120b model card — https://huggingface.co/openai/gpt-oss-120b
- [O2] gpt-oss model card paper, arXiv:2508.10925 (Table 3 HealthBench; 131,072 context) — https://arxiv.org/html/2508.10925
- [B1] Baichuan-M2-32B model card — https://huggingface.co/baichuan-inc/Baichuan-M2-32B
- [B2] Baichuan-M3-235B model card — https://huggingface.co/baichuan-inc/Baichuan-M3-235B ; paper arXiv:2602.06570
- [R1] AMD ROCm, RDNA3.5 system optimization (Strix Halo kernel/ROCm/TTM) — https://rocm.docs.amd.com/en/docs-7.14.1/reference/system-optimization/rdna3-5.html
- [S1] kyuz0/amd-strix-halo-toolboxes `docs/results.json` (llama-bench on Ryzen AI Max+ 395) — https://github.com/kyuz0/amd-strix-halo-toolboxes/blob/main/docs/results.json
- [S2] kyuz0/amd-strix-halo-vllm-toolboxes README — https://github.com/kyuz0/amd-strix-halo-vllm-toolboxes
- [S3] kyuz0/amd-strix-halo-toolboxes README — https://github.com/kyuz0/amd-strix-halo-toolboxes
- [A1] Claude models and pricing (Anthropic docs) — https://platform.claude.com/docs/en/about-claude/models/overview , https://platform.claude.com/docs/en/about-claude/pricing

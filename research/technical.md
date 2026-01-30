# QuickRail Technical Feasibility Research

**Date:** January 2026
**Purpose:** Pre-seed technical due diligence for AI-powered European rail booking platform

---

## 1. ElevenLabs Voice API Latency Benchmarks

### Production Numbers

| Metric | Value | Source |
|--------|-------|--------|
| Flash v2.5 Model Inference | 75ms | [ElevenLabs Docs](https://elevenlabs.io/docs/developers/best-practices/latency-optimization) |
| End-to-End TTFB (US) | 135ms | [ElevenLabs Docs](https://elevenlabs.io/docs/developers/best-practices/latency-optimization) |
| End-to-End TTFB (EU) | 150-200ms | [ElevenLabs Docs](https://elevenlabs.io/docs/developers/best-practices/latency-optimization) |
| Third-Party Benchmark (US) | 350ms | [Smallest.ai Benchmark 2025](https://smallest.ai/blog/tts-benchmark-2025-smallestai-vs-elevenlabs-report) |
| Third-Party Benchmark (India) | 527ms | [Smallest.ai Benchmark 2025](https://smallest.ai/blog/tts-benchmark-2025-smallestai-vs-elevenlabs-report) |

### Key Considerations
- 75ms is **model inference only** - real-world adds network + application latency
- WebSocket streaming recommended for lowest latency
- `chunk_length_schedule` parameter controls buffer/quality tradeoff
- EU customers need dedicated EU tech stack (contact sales) for 150-200ms TTFB
- Voice selection impacts latency: Default > Synthetic > Instant Voice Clone

**Verdict:** Achievable sub-300ms TTS for EU users with proper configuration.

---

## 2. Claude API Response Times

### Current Benchmarks

| Model | First Token Latency | Per-Token Latency | Tokens/sec |
|-------|---------------------|-------------------|------------|
| Claude 4.5 Sonnet | 2.0s | 0.015s | ~67 |
| Claude 4.5 Sonnet (Streaming) | 300-360ms | - | 20-30 |
| Claude 3.5 Sonnet | ~1.0s | - | ~80 |
| GPT-5.2 (comparison) | 0.55s | 0.010s | 187 |

Sources: [AI Multiple LLM Benchmark](https://research.aimultiple.com/llm-latency-benchmark/), [GPT for Work Tracker](https://gptforwork.com/tools/openai-api-and-other-llm-apis-response-time-tracker)

### Route Decomposition Optimization
- **Use streaming**: First token in 300-360ms vs 2s for complete response
- **Prompt caching**: Reduces TTFT significantly for repeated query patterns
- **Model selection**: Claude 3.5 Sonnet offers 2x speed over Opus with 80 tok/s
- **Translation tasks**: Claude 4.5 Sonnet shows 2s first-token, 0.015s per-token

**Verdict:** With streaming, route decomposition can start responding in ~300ms. Full parsing of a typical route query (Paris to Berlin, stopping in Frankfurt) can complete in 500-800ms.

---

## 3. EU Train Provider APIs

### Tier 1: Official APIs with Developer Access

| Provider | API Type | Access | Documentation | Notes |
|----------|----------|--------|---------------|-------|
| **Deutsche Bahn** | REST | Free, 60 req/min | [developer-docs.deutschebahn.com](https://developer-docs.deutschebahn.com/apis) | Timetables, stations, disruptions. Poorly documented. CC BY 4.0 license. |
| **SNCF** | REST (Navitia) | Free, 90k/month | [SNCF Open Data](https://ressources.data.sncf.com/explore/dataset/api-sncf/) | Schedules + real-time. No direct booking. |

### Tier 2: Restricted/Partnership Access

| Provider | Access Model | Requirements |
|----------|--------------|--------------|
| **Trenitalia** | No public API | Via Lyko or direct partnership |
| **Renfe** | Limited Open Data | Via Lyko; direct requires ATOUT France registration + 10k EUR guarantee |
| **OBB** | OSDM-compliant | Partnership required |
| **SBB** | OSDM-compliant | Partnership required |

### Tier 3: Aggregator APIs (Recommended for MVP)

| Provider | Coverage | Integration Time | Model |
|----------|----------|------------------|-------|
| **[Trainline Partner Solutions](https://tps.thetrainline.com/our-products/global-api/)** | 270+ operators, 40 countries | Contact sales | B2B, RESTful |
| **[SilverRail (SilverCore)](https://docs.silverrailtech.com/)** | All major EU carriers | Contact sales | Aggregated API, OSDM-native |
| **[Lyko](https://lyko.tech/en/portfolio/train-api/)** | DB, SNCF, Trenitalia, Renfe, SNCB | 3 months | Commission sharing |
| **[AllAboard](https://allaboard.eu/api-solution)** | European multi-carrier | Contact sales | White-label |

### OSDM Standard (Emerging)
- **What:** Open Sales and Distribution Model - EU rail interoperability standard
- **Status:** 6+ railways implementing by end 2025; PKP, Rail Market NSW live in 2025
- **Benefit:** Single API spec across carriers once adopted
- **Source:** [osdm.io](https://osdm.io/)

**Verdict:** For pre-seed MVP, use **Trainline or SilverRail aggregator API**. Avoids 6+ separate integrations. OSDM adoption accelerating but not universal yet.

---

## 4. Multi-Language Inference Considerations

### Performance by Language

| Model | MMMLU (Multilingual Reasoning) | Notes |
|-------|--------------------------------|-------|
| Gemini 3 Pro | 91.8% | Best multilingual |
| Claude Opus 4.5 | 90.8% | Strong multilingual |
| Claude Opus 4.1 | 89.5% | - |
| Llama 3.1 | Optimized | Open source option |

Source: [Vellum LLM Leaderboard](https://www.vellum.ai/llm-leaderboard)

### Latency Implications
- **Translation tasks add ~200-500ms** depending on model choice
- **Input tokenization**: Non-Latin scripts (Cyrillic, etc.) tokenize less efficiently
- **Recommendation**: Keep prompts in English internally; translate only user-facing I/O
- **Caching**: Cache common multilingual phrases for voice responses

### Supported Languages (Critical EU Markets)
- English, German, French, Italian, Spanish, Dutch, Polish, Swedish
- All major models support these; Claude/GPT have best quality

**Verdict:** Minimal latency penalty for EU languages. Keep internal processing in English.

---

## 5. Architecture for Sub-3s Total Latency

### Target Latency Budget

| Stage | Target | Method |
|-------|--------|--------|
| ASR (Speech-to-Text) | 150ms | Deepgram streaming |
| LLM (Route Decomposition) | 300-500ms | Claude streaming + caching |
| Rail API (Parallel Queries) | 800-1200ms | Parallel async, aggregator API |
| TTS (Voice Response) | 200-300ms | ElevenLabs Flash + EU stack |
| Network/Overhead | 200-300ms | Edge deployment, WebSocket |
| **Total** | **1650-2450ms** | **Under 3s achievable** |

### Architecture Patterns

```
User Voice Input
       |
       v
[WebRTC/WebSocket] -----> [Edge Node (EU)]
       |                        |
       v                        v
[Deepgram Streaming ASR]  [ElevenLabs EU TTS]
       |                        ^
       v                        |
[Claude 3.5 Sonnet]            |
  - Route parsing              |
  - Intent extraction          |
  - Streaming response --------+
       |
       v
[Rail API Gateway]
  - Trainline/SilverRail
  - Parallel segment queries
  - Response caching
```

### Critical Optimizations

1. **Speculative Processing**
   - Start LLM inference on ASR interim results (80-90% confidence)
   - 200-400ms savings when correct

2. **Streaming Everything**
   - ASR: Stream partial transcripts
   - LLM: Stream first token in 300ms
   - TTS: Begin playback while generating

3. **Parallel Rail Queries**
   - Decompose multi-segment routes immediately
   - Query all segments simultaneously
   - Aggregate responses

4. **Response Caching**
   - Cache common routes (Paris-London, Berlin-Munich)
   - Pre-synthesize frequent TTS phrases
   - Cache rail schedules (15-min TTL)

5. **Infrastructure**
   - EU edge deployment (Frankfurt, Amsterdam)
   - WebSocket persistent connections
   - Dedicated GPU instances for LLM (Groq LPU optional for speed)

Sources: [Introl Voice AI Infrastructure Guide](https://introl.com/blog/voice-ai-infrastructure-real-time-speech-agents-asr-tts-guide-2025), [Cresta Engineering Blog](https://cresta.com/blog/engineering-for-real-time-voice-agent-latency), [Medium: Solving Voice AI Latency](https://medium.com/@reveorai/solving-voice-ai-latency-from-5-seconds-to-sub-1-second-responses-d0065e520799)

---

## 6. Risk Assessment

| Risk | Severity | Mitigation |
|------|----------|------------|
| Rail API latency variance | HIGH | Aggressive caching, aggregator API SLAs |
| Claude TTFB > 2s (non-streaming) | MEDIUM | Always use streaming mode |
| ElevenLabs EU latency without sales contact | MEDIUM | Budget for enterprise tier |
| OSDM adoption slower than expected | LOW | Aggregator APIs abstract this |
| Multi-language accuracy | LOW | Major EU languages well-supported |

---

## 7. Recommendations

### MVP Stack
- **ASR:** Deepgram (150ms streaming)
- **LLM:** Claude 3.5 Sonnet (streaming, 300ms TTFT)
- **Rail API:** Trainline Partner Solutions or SilverRail
- **TTS:** ElevenLabs Flash v2.5 (EU enterprise tier)
- **Infra:** AWS EU-West or GCP Europe, WebSocket connections

### Cost Estimates (Monthly, 10k conversations)
- ElevenLabs: $99-$330 (Pro/Scale)
- Claude API: ~$500-1000 (depends on usage)
- Rail API: Partnership/commission model
- Infrastructure: $500-1000

### Next Steps
1. Contact Trainline/SilverRail for API partnership terms
2. Contact ElevenLabs for EU enterprise access
3. Build latency proof-of-concept with synthetic rail data
4. Validate sub-3s end-to-end with real user testing

---

**Conclusion:** Sub-3s latency is technically achievable with current infrastructure. Primary challenge is rail API integration complexity - solved by aggregator APIs. Voice and LLM components are mature and performant.

# Voice Stack Research: STT/TTS for QuickRail

## Executive Summary

For a voice-first EU rail booking system targeting sub-500ms response, the recommended MVP stack is:
- **STT**: Deepgram Nova-3 (EU endpoint)
- **TTS**: Cartesia Sonic-3 (GDPR compliant)

## Provider Comparison

### Speech-to-Text (STT)

| Provider | Latency | Accuracy | Pricing | EU Hosting | Notes |
|----------|---------|----------|---------|------------|-------|
| **Deepgram Nova-3** | <300ms | 93%+ (6.84% WER) | $0.0077/min batch, ~$0.014/min streaming | YES (api.eu.deepgram.com) | Best EU option, GA Jan 2026 |
| **AssemblyAI Universal** | 300ms P50 | 92%+ | $0.15/hr streaming ($0.0025/min) | NO | Immutable transcripts, no EU endpoint |
| **ElevenLabs Scribe v2** | 150ms | 90%+ | Included in plans | YES (Netherlands) | Good but TTS-focused company |
| **OpenAI Whisper API** | 4-7s+ | 95%+ | $0.006/min | NO | Too slow for real-time |
| **Whisper Self-Hosted** | <500ms | 95%+ | Infra only (~$276/mo GPU) | Self-managed | Requires ML ops team |

### Text-to-Speech (TTS)

| Provider | Latency (TTFA) | Quality | Pricing | EU Hosting | Notes |
|----------|----------------|---------|---------|------------|-------|
| **Cartesia Sonic-3** | 40-90ms | Excellent | $0.038/1K chars | GDPR compliant | Fastest, purpose-built for real-time |
| **ElevenLabs Flash** | ~75ms | Best-in-class | $0.050/1K chars | YES (Enterprise) | Higher quality, slight premium |
| **ElevenLabs Turbo** | 250-300ms | Excellent | $0.030/1K chars | YES (Enterprise) | Good quality/latency balance |
| **PlayHT** | ~300ms | Good | $0.020/1K chars | Unknown | Budget option, higher latency |
| **LMNT** | Ultra-low | Good | Opaque pricing | Unknown | English-only, limited languages |

## Latency Budget Analysis (Target: <500ms)

```
Voice Input:     ~50ms  (mic capture + VAD)
Network (EU):    ~30ms  (regional hosting)
STT:            ~150ms  (Deepgram streaming)
LLM:            ~200ms  (streamed, first token)
TTS:             ~50ms  (Cartesia TTFA)
Audio Start:     ~20ms  (buffer + playback init)
────────────────────────
TOTAL:          ~500ms  (achievable with streaming)
```

## EU Data Residency Summary

| Provider | EU Endpoint | GDPR | Data Residency |
|----------|-------------|------|----------------|
| Deepgram | api.eu.deepgram.com | Yes | AWS EU regions |
| ElevenLabs | Netherlands (Enterprise) | Yes | Enterprise only |
| Cartesia | Via DPF/SCCs | Yes | GDPR compliant |
| AssemblyAI | None | Via DPA | US-based processing |

## OpenAI Whisper: API vs Self-Hosted

| Factor | API | Self-Hosted (faster-whisper) |
|--------|-----|------------------------------|
| Latency | 4-7s+ | <500ms with GPU |
| Cost (500 hrs/mo) | $180 | $276+ infra |
| Real-time | No native support | Requires chunking |
| EU data control | No | Full control |
| Ops overhead | Zero | Significant |
| **Recommendation** | Not for real-time | Only with ML team |

## Sub-500ms Optimization Techniques

1. **Streaming everything**: Start TTS before LLM finishes, process partial transcripts
2. **Regional deployment**: Co-locate STT/LLM/TTS in same EU region
3. **Connection pooling**: Persistent WebSockets, pre-warmed instances
4. **Codec optimization**: Use Opus codec end-to-end
5. **Aggressive endpointing**: 200-300ms silence detection
6. **Speculative execution**: Pre-generate common responses

## MVP Recommendation

### Primary Stack (EU-First, Sub-500ms)

| Component | Provider | Why |
|-----------|----------|-----|
| **STT** | Deepgram Nova-3 (EU) | Native EU endpoint, <300ms, best accuracy |
| **TTS** | Cartesia Sonic-3 | 40-90ms TTFA, GDPR compliant, best latency |
| **Fallback TTS** | ElevenLabs Flash | Higher quality for complex responses |

### Cost Estimate (10K minutes/month)

```
Deepgram STT (streaming): $140/mo
Cartesia TTS:             $380/mo (est. 1M chars)
────────────────────────────────
Total:                    ~$520/mo
```

### Implementation Priority

1. **Week 1**: Deepgram EU WebSocket integration + VAD
2. **Week 2**: Cartesia streaming TTS integration
3. **Week 3**: End-to-end latency optimization
4. **Week 4**: Fallback handling + monitoring

## Key Trade-offs

| Choice | Pros | Cons |
|--------|------|------|
| Deepgram over AssemblyAI | EU endpoint, lower latency | Slightly lower accuracy |
| Cartesia over ElevenLabs | 2x faster, cheaper | Slightly less natural voices |
| API over self-hosted | Zero ops, instant scaling | Less control, ongoing costs |

## References

- [Deepgram EU Endpoint GA](https://deepgram.com/learn/deepgram-eu-endpoint-now-generally-available)
- [Deepgram Pricing](https://deepgram.com/pricing)
- [ElevenLabs Latency Optimization](https://elevenlabs.io/docs/developers/best-practices/latency-optimization)
- [AssemblyAI Streaming STT](https://www.assemblyai.com/products/streaming-speech-to-text)
- [Cartesia vs PlayHT](https://cartesia.ai/vs/cartesia-vs-playht)
- [Cartesia GDPR Compliance](https://cartesia.ai/blog/gdpr-compliance)
- [Voice AI Latency Guide](https://vapi.ai/blog/speech-latency)
- [300ms Rule for Voice AI](https://www.assemblyai.com/blog/low-latency-voice-ai)
- [Whisper API vs Self-Hosted Pricing](https://brasstranscripts.com/blog/openai-whisper-api-pricing-2025-self-hosted-vs-managed)

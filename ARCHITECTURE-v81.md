# EchoChiasm v81 — Future-Proof Foundation

v81 is intentionally a low-risk architectural release. It does not redesign the v71/v80 UI, voice system, driving microphone behavior, Bible reader, Study reader, or Dropbox workflow.

## Stable boundaries introduced

- AI Capability API v1: first-party features call `callAI()`, not a vendor-specific model function.
- Optional local/native adapter: a future iOS/Android wrapper can expose `window.EchoChiasmLocalAI = { isAvailable(), request() }`.
- Current routing is unchanged by default: EchoChiasm shared gateway, or Anthropic BYO.
- Cloud gateway version 1: the relay has an explicit API/provider boundary. Anthropic remains the only active server adapter in v81.
- Library schema v1: new Dropbox exports identify `schemaVersion`, `contentType`, and `appVersion`; all legacy v80 JSON remains import-compatible.
- IndexedDB schema v2: adds an additive metadata store and explicit migration version while never rewriting existing study records during upgrade.
- Service worker v81: navigation/index uses network-first with cached fallback; static Bible/app assets remain offline cache-first.
- Rate-limit storage boundary: the server limiter is abstracted behind a replaceable adapter. v81 keeps the proven in-memory implementation.

## Non-goals in v81

v81 does not enable Apple Foundation Models, Siri, Gemini Nano, ChatGPT, or another local model by itself. It creates the stable interface a native/mobile adapter can plug into later without rewriting EchoChiasm.

v81 also does not introduce a new persistent rate-limit dependency. That should be added only after selecting and testing a deployment-compatible persistent store.

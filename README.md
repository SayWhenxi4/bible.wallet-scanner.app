# EchoChiasm — Standalone PWA

A bilingual (English/Spanish) Bible study tool with a built-in Bible reader.
Browse or search the full text, tap verses to select a passage, send it
straight into EchoChiasm, and get study notes back — structure (chiasm /
focal climax / inclusio), Hebrew or Greek word insight, and application.

This is a fully static app — `index.html`, `manifest.json`, `sw.js`, two
icons, and a `bible/` folder with the bundled Bible text. No backend, no
server for the Bible reader. The EchoChiasm generation step calls the
Anthropic API directly from the browser using your own API key, the same
bring-your-own-key pattern Wallet-Tracer uses for its other API keys. You
paste your key into Settings once; it's stored in this browser's
localStorage on this device only.

## Bible text and licensing

The bundled Bible text is public domain, so it can ship directly in the app
with no license fees and no attribution requirements:

- **English:** King James Version (1769 standard text)
- **Spanish:** Reina-Valera 1909 (confirmed public domain revision — not the
  1960 revision, which is still under copyright)

Both are sourced from `scrollmapper/bible_databases` on GitHub, a long-
standing open dataset (originally distributed via the CrossWire/SWORD
project). 66 books, 31,102 verses, verified matching in both languages.

This is not the NKJV or RVR1960 you've been using in conversation — those
are copyrighted and can't be bundled. If you want those specific
translations inside the app later, that needs the live-API-lookup approach
(a Bible API with your own key) instead of bundled text — ask any time and
I can add it as a second option alongside the bundled KJV/RV1909.

## 1. Get an Anthropic API key

1. Go to https://console.anthropic.com and sign in (or create an account —
   separate from your claude.ai login).
2. Add billing under **Settings → Billing**. Usage is pay-as-you-go; each
   EchoChiasm generation is roughly 800–1200 tokens, so cost per use is a
   fraction of a cent.
3. Go to **Settings → API Keys → Create Key**. Copy it immediately — it's
   only shown once.

## 2. Deploy alongside wallet-scanner.app

Same flow as Wallet-Tracer:

- Push this folder (`echochiasm-pwa/`) to a GitHub repo.
- Connect it to Netlify (or add it as a new site under your existing
  Netlify account — doesn't have to share the wallet-scanner.app repo).
- Point a subdomain at it if you want it under your existing domain, e.g.
  `echochiasm.wallet-scanner.app` — add that as a custom domain in Netlify's
  site settings, then add the matching CNAME record wherever
  wallet-scanner.app's DNS is managed.
- Or just deploy it as its own Netlify `*.netlify.app` address if you'd
  rather keep it separate — either works, nothing in the code assumes a
  particular domain.

## 3. Enter your key on first use

Open the deployed app, tap the gear icon (top right), paste your API key,
tap **Test connection** to confirm it works, then **Save**. That's it — no
redeploy needed to add or change the key later, since it lives in the
browser, not in the code.

## 4. Install to home screen

- **iPhone (Safari):** tap Share → "Add to Home Screen."
- **Android (Chrome):** tap the ⋮ menu → "Add to Home Screen" or "Install app."

It launches full-screen with its own icon, no browser chrome.

## 5. Bump the cache on updates

`sw.js` caches the app shell for offline load, and now also opportunistically
caches the Bible JSON files the first time each language is opened, so
repeat reading works fully offline. Whenever you change `index.html`, bump
`CACHE_NAME` in `sw.js` (currently `echochiasm-shell-v2`) so returning
visitors get the update instead of a stale cached copy — same rule you
already follow for Wallet-Tracer's `sw.js`.

## Notes on the key and safety

- The key is stored in `localStorage`, in plain text, on whatever device
  you enter it on. It is not encrypted at rest by the browser. Anyone with
  physical access to your unlocked device and browser devtools could read
  it — same exposure as any other bring-your-own-key field in Wallet-Tracer.
- The key is sent directly from your browser to `api.anthropic.com` on each
  request — it never passes through any server of ours.
- If a key ever leaks or you want to rotate it, revoke it at
  console.anthropic.com → API Keys and generate a new one; paste the new
  key into Settings the same way.
- This app has no login and no multi-user support — it's a personal tool
  for whoever has the device, same as Wallet-Tracer.


## v67 hands-free commands
- Highlight verse N / Highlight verses N through M (persistent yellow highlight)
- Remove highlight verse N
- Backup to Dropbox / Save to Dropbox
- Load from Dropbox / Restore from Dropbox
Dropbox voice commands reuse the existing sync buttons and require the account to already be connected once in Settings.


## v68 hands-free polish
- Closed-form driving commands execute as soon as speech recognition finalizes them instead of waiting for the full Ask AI silence window.
- Added voice commands: `Repeat`, `Repeat chapter`, and `Undo repeat` / `Turn off repeat`.
- `Repeat chapter` loops Bible reading from verse 1; Study repeat loops the generated Study notes.

## v70 theme + touch polish
- Fixed the Inverted-theme stacking regression that could leave the visible Menu, book, chapter, and Search controls untappable on iOS Safari.
- Top navigation and the Bible/Study audio control bar now fade after idle time but remain tappable on the first touch.
- Reworked Light and Dark to use the same media-app glass/rainbow visual family as Inverted while preserving Bible text, saved highlights, active-reading highlights, Study notes, sheets, and controls.
- Service-worker cache bumped to `echochiasm-shell-v70`.

## EchoChiasm AI Relay v76

EchoChiasm AI is the default connection mode. Ordinary users do not need an Anthropic API key. The shared key is used only inside the Netlify Function at `netlify/functions/echo-ai.js`.

### Netlify setup

1. Deploy this entire folder/ZIP to Netlify. Do not deploy only `index.html`.
2. In Netlify, open Site configuration → Environment variables.
3. Add `ANTHROPIC_API_KEY` with the shared Anthropic API key.
4. If Anthropic reports that the key is identity-linked, also add `ANTHROPIC_WORKSPACE_ID` with that key's Anthropic workspace ID. Ordinary API keys can leave this unset.
5. Redeploy after changing environment variables, then use Settings → AI Connection → Check EchoChiasm AI.
4. Redeploy the site after changing environment variables.
5. Open EchoChiasm → Settings → AI Connection → EchoChiasm AI → Check EchoChiasm AI.

The v76 check reports a safe diagnostic category instead of hiding every upstream failure behind one generic message. It never displays the API key or Anthropic's raw account-specific error text. Typical results include:

- `ANTHROPIC_AUTH` — the key was rejected.
- `ANTHROPIC_PERMISSION` — the key/account lacks permission.
- `ANTHROPIC_BILLING` — billing or credits likely need attention.
- `ANTHROPIC_MODEL` — the configured model is unavailable to the account.
- `ANTHROPIC_RATE_LIMIT` — Anthropic is throttling the account.
- `ANTHROPIC_REQUEST` — Anthropic rejected the request format.
- `NETWORK_ERROR` — the Netlify Function could not reach Anthropic.
- `OK` — relay, key, model access, and billing are working.

The health check deliberately uses a minimal Messages API request with no prompt-caching fields. Normal study requests use prompt caching, but v76 automatically retries once without caching if Anthropic rejects the cache syntax. This makes caching an optimization rather than a single point of failure.

Optional environment variables:
- `ECHO_AI_ENABLED=false` — emergency kill switch.
- `ECHO_ANTHROPIC_MODEL=claude-sonnet-5` — server-side model selection. If omitted, the relay uses `claude-sonnet-5`.
- `ECHO_ALLOWED_ORIGINS=https://bible.wallet-scanner.app` — comma-separated allowed origins.
- `ECHO_APP_TOKEN` — optional casual-bot filter. If you set it, it must match the `ECHO_APP_TOKEN` constant in `index.html`. This value is visible in client code and is not a true secret.
- `ECHO_GLOBAL_PER_HOUR=800`, `ECHO_IP_PER_HOUR=100`, `ECHO_IP_PER_DAY=400` — best-effort in-memory throttles.

The relay only accepts fixed EchoChiasm tasks. The browser cannot supply a model name or system prompt. Keep a hard Anthropic spend limit as the financial backstop.

Advanced users can still choose “My Anthropic API Key” and use direct BYO access.


## v77 relay fix
- Claude Sonnet 5 adaptive thinking is explicitly disabled for EchoChiasm requests. This preserves the app's prior direct-response behavior and prevents tiny health-check output budgets from being consumed by default adaptive thinking.
- The Settings health diagnostic may show Anthropic's sanitized error message (max 240 characters) when an upstream request is rejected. API keys and long opaque identifiers are redacted.
- Relay version: v77-sonnet5.

## v81 Future-Proof Foundation

v81 preserves the existing UI, Bible reader, voice/driving microphone behavior, Study mode, Dropbox flow, shared relay, and Anthropic BYO behavior. It adds a provider-neutral AI capability router, an optional native/on-device AI adapter hook, Library Schema v1, IndexedDB Schema v2, an explicit AI Gateway API v1, and safer service-worker versioning. See `ARCHITECTURE-v81.md`.

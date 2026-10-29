# Universal Translator

A voice-to-voice, two-way conversation translator. One person speaks Bangla, the other speaks any of 60+ languages. Each side taps a mic, the app transcribes the speech, translates it with AI and reads the translation aloud.

**বাংলা সারসংক্ষেপ:** দুজন মানুষ দুই ভাষায় কথা বললে অ্যাপটি কথা শুনে অনুবাদ করে এবং সেই ভাষায় পড়ে শোনায়। ওপরে বিদেশির প্যানেল (ভাষা বেছে নেওয়া যায়), নিচে বাংলাভাষীর প্যানেল, মাঝখানে চ্যাট ইতিহাস।

## Features

- Dark glassmorphism UI with an animated splash screen
- Split layout: foreigner on top, Bangla speaker at the bottom, chat history in the middle
- 60+ languages with a searchable picker (your last choice is remembered)
- Live transcript, listening animation and a translating spinner
- Automatic spoken translation (`speechSynthesis`) and a replay button on every bubble
- Backend on Cloudflare Workers AI, so no API key is needed

## Files

| File | Purpose |
|---|---|
| `index.html` | The whole frontend (HTML, CSS and JS in one file) |
| `worker.js` | Cloudflare Worker that translates text with Workers AI |

## How it works

```
Mic → speech recognition → text → POST /api/translate → Worker → Workers AI (m2m100)
                                                                      ↓
                           speechSynthesis ← translated text ← JSON response
```

Request:

```json
{ "text": "আপনি কেমন আছেন?", "source": "bn", "target": "en" }
```

Response:

```json
{ "translation": "How are you?" }
```

## Setup

### 1. Deploy the Worker

1. Create a free account at [dash.cloudflare.com](https://dash.cloudflare.com).
2. Go to **Workers & Pages → Create → Start with Hello World!**, name it `translator` and deploy.
3. Click **Edit code**, replace everything with the contents of `worker.js` and deploy.
4. Go to **Settings → Bindings → Add binding → Workers AI** and set the variable name to exactly `AI`.
5. Deploy again so the binding takes effect.

Optional, if you deploy with Wrangler instead of the dashboard:

```toml
name = "translator"
main = "worker.js"
compatibility_date = "2026-10-01"

[ai]
binding = "AI"
```

### 2. Point the frontend at your Worker

In `index.html`, set:

```js
const API_URL = "https://translator.YOUR-SUBDOMAIN.workers.dev/api/translate";
```

### 3. Host the frontend over HTTPS

Browsers only allow microphone access on HTTPS (or `localhost`), so opening the file directly will not work. Use any static host:

- Cloudflare Pages (**Upload your static files**)
- GitHub Pages
- Netlify or Vercel

### 4. Lock down CORS

Once your site is live, edit `worker.js`:

```js
const ALLOWED_ORIGIN = "https://your-site.pages.dev";
```

Redeploy. Until you do this, any website can call your Worker.

## Adding a language

Add one line to the `L` list in `index.html`:

```js
["code", "English name", "Native name", "speech-locale", "🏳️"]
```

Add `, 1` at the end for right-to-left scripts. Then add the code to the `SUPPORTED` list in `worker.js` (and to `CODE_MAP` if the app code differs from the M2M100 code).

## Limitations

- **Speech recognition** works in Chrome, Edge and Safari. Firefox does not support it.
- **Spoken output** needs a voice for that language installed on the device. Many phones have no Bangla voice, so the text still appears but is not read aloud.
- **Translation quality** from M2M100 is good for major languages and weaker for smaller ones.
- **Free tier:** Workers AI has a daily usage limit. Heavy traffic may need a paid plan.
- Input is capped at 1000 characters per message.

## Troubleshooting

| Problem | Fix |
|---|---|
| "API not connected yet" in chat | `API_URL` still has the placeholder value |
| "Microphone blocked" | Allow mic access in the browser's site settings and use HTTPS |
| "Couldn't translate: Server returned 502" | The `AI` binding is missing or misnamed; redeploy after adding it |
| CORS error in the console | `ALLOWED_ORIGIN` does not match your site's URL |
| Translation shows but no sound | No voice installed for that language on the device |

## Roadmap

- Wrap as an Android app with native speech APIs
- Optional Claude-powered translation for higher quality on smaller languages

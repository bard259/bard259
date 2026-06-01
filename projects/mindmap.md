# mindmap

LLM-powered interactive knowledge mind-map. Type any topic and explore it by double-clicking nodes to recursively expand into sub-topics. Each expansion calls GPT-4.1-mini with your stated perspective and learning goal to generate contextually appropriate sub-categories.

Cross-platform: runs in browser, iOS, and Android from a single codebase (Expo / React Native).

## Architecture

```
Frontend (Expo / React Native)
  └── components/MindMap.js     SVG node+edge graph, tap/double-tap/pan/zoom
        └── services/mindmap.js  OpenAI API calls with rate limiter + demo fallback

Backend (Node.js / Express)
  └── server.js                  POST /api/mindmap → OpenAI Responses API (strict JSON schema)
  └── public/index.html          Standalone vanilla-JS web version (no Expo required)
```

## Quick start

### Web app (Express backend)

```bash
# Create .env
echo "OPENAI_API_KEY=sk-..." > .env

npm install
node server.js
# Open http://localhost:8080
```

### Mobile / Expo app

```bash
# Create .env
echo "EXPO_PUBLIC_OPENAI_API_KEY=sk-..." > .env

npm install
npx expo start
# Press w for web, i for iOS simulator, a for Android
```

> **Note:** If you don't have an OpenAI key, click **Demo** in the app to explore pre-built finance and technology trees.

## Features

- **Recursive expansion** — double-click any node to generate 3 sub-topics
- **Collapse/expand** — double-click an already-expanded node to collapse it
- **Ancestry memory** — each expansion receives the full path from root so context is preserved
- **Perspective controls** — set your background ("undergraduate student", "finance professional") and purpose ("exam prep", "investment research") to tailor explanations
- **Rate limiter** — 10 expansions per session with graceful error messaging
- **Demo mode** — works without an API key for finance and technology topics

## Project structure

```
mindmap/
├── components/
│   └── MindMap.js         Core SVG mind-map component (pan, zoom, tap, expand)
├── services/
│   └── mindmap.js         OpenAI integration, rate limiting, demo fallback
├── public/
│   └── index.html         Standalone vanilla-JS web version
├── server.js              Express API server
├── App.js                 Expo entry point
└── app.json               Expo configuration
```

## Roadmap

- [ ] **Fix API key exposure** — `EXPO_PUBLIC_` prefix bakes the OpenAI key into the browser bundle; move all AI calls to `server.js` and remove the client-side key
- [ ] **Save and export** — download the current tree as JSON (re-importable), PNG screenshot, or formatted PDF outline
- [ ] **Edit and delete nodes** — long-press to rename or prune any node
- [ ] **Persistence** — save trees in localStorage (web) / AsyncStorage (mobile) so sessions survive refresh; optional backend sync
- [ ] **Sharing** — generate a read-only shareable URL for any saved tree
- [ ] **Expand demo mode** — pre-built trees for more topics (history, science, programming)
- [ ] **Alternative LLM backend** — the server is LLM-agnostic; add a flag to route through Claude (`claude-haiku-4-5`) and compare explanation quality

## License

MIT

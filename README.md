# Amoeba — Deployable Multiplayer Web Game

Real-time party game where players submit prompts, one prompt is chosen, everyone answers anonymously, and players take turns **guessing who wrote which answer**. Correct guesses **capture** the author into the guesser's team; wrong guesses pass the turn to the accused.

## Monorepo Layout
```
amoeba-deployable/
  server/    # Node + Express + Socket.IO (Dockerfile included)
  client/    # Vite + React (configurable server URL via env)
```

## Quick Start (Local Dev)
Terminal A — Server:
```bash
cd server
npm install
npm run dev
# -> http://localhost:4000
```

Terminal B — Client:
```bash
cd client
npm install
npm run dev
# -> http://localhost:5173 (auto-connects to ws://localhost:4000)
```

Open the client URL on multiple devices/browsers and share the same **room code**.

---

## Deploy

### Option A — Server on Render (free) + Client on Vercel (free)

**Server (Render):**
1. Create a new **Web Service** from this `server/` folder (Node).
2. Environment:
   - **Build Command:** `npm ci`
   - **Start Command:** `node index.js`
   - **PORT:** Render sets it automatically.
3. Add a custom domain later if you wish (optional).

**Client (Vercel):**
1. Import the repo, select the `client/` folder as the project root.
2. Add Environment Variable:
   - `VITE_SERVER_URL=https://<your-render-app>.onrender.com`
3. Deploy. The client will connect to your server over WebSocket.

### Option B — Server with Docker anywhere (Fly.io/Render/VPS)

Build & run the server container:
```bash
cd server
docker build -t amoeba-server .
docker run -p 4000:4000 -e PORT=4000 amoeba-server
```

Then build and host the client (any static host):
```bash
cd client
# set server URL for the build
echo "VITE_SERVER_URL=https://YOUR_SERVER_URL" > .env.production
npm install
npm run build
# deploy ./dist to Netlify/Vercel/Static S3/etc.
```

---

## Game Flow
- **Lobby**: players join with a username; the first joiner is host.
- **Prompt Submit**: everyone submits one prompt; server picks one randomly.
- **Answer Submit**: everyone submits one answer to the chosen prompt.
- **Reveal/Guessing**:
  - Answers shown anonymously.
  - One player randomly chosen to start.
  - On your turn: select an answer → choose a username → submit guess.
  - **Correct**: author joins your team; you go again.
  - **Wrong**: turn passes to the accused.
- **Round End**: when one team has everyone.

## Notes
- This in-memory server is great for small rooms. For persistence or scaling, use Redis/Postgres and a Socket.IO adapter.
- Make sure your server’s CORS allows your client origin (already permissive by default here).

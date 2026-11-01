# FRAME//FATE — Hybrid AI Movie Simulator

A single-player, GitHub Pages-ready branching story game. The browser game always has its original predefined story scenes. When you connect your own AI endpoint, it can generate new chapters, dialogue, decisions and endings dynamically. If AI is unavailable or the provider limit is reached, the game falls back to its built-in story.

## What is included

- 8 genres, character roles, difficulty levels, time periods and presentation styles.
- Existing branching scenes, character stats, inventory, story history and endings.
- Hybrid engine: built-in scenes work without an API; AI scenes are optional.
- Google Gemini API integration through a Cloudflare Worker proxy.
- API key stored as a Cloudflare Worker secret, never in the public website.
- Local browser saves and responsive layout.

## Free architecture

- **Frontend:** GitHub Pages (free for public repositories).
- **Backend proxy:** Cloudflare Workers Free. Cloudflare currently documents 100,000 Worker requests/day on its Free plan; account limits and terms can change.
- **AI provider:** Google Gemini API. Free-tier model access and rate limits depend on the model and account and can change. It is not unlimited, and the game remains playable when the limit is reached.

The Worker calls the Gemini `generateContent` REST endpoint and requests JSON output. The default model is `gemini-2.5-flash`; you can change `GEMINI_MODEL` in `wrangler.toml` to a model available to your Google AI Studio account.

## Part A — Deploy the game to GitHub Pages

1. Create a **public** GitHub repository, for example `frame-fate`.
2. Upload the contents of this folder to the repository root. Keep `index.html` at the root.
3. Open **Settings → Pages**.
4. Under Build and deployment, select **Deploy from a branch**, choose `main` and `/(root)`, then Save.
5. Your site will be available at `https://YOUR-USERNAME.github.io/frame-fate/` after GitHub finishes publishing.

## Part B — Create a Gemini API key

1. Open [Google AI Studio](https://aistudio.google.com/).
2. Create an API key in the API Keys section.
3. Copy it somewhere private. Do not paste it into `index.html`, GitHub, a public issue, or a commit.

Create an API key from the API Keys section. Keep it private. Gemini free-tier model access and rate limits can change; check the [official Gemini pricing and limits](https://ai.google.dev/gemini-api/docs/pricing).

## Part C — Deploy the secure API proxy to Cloudflare Workers

You need Node.js and npm on your computer.

1. Sign in at [Cloudflare Dashboard](https://dash.cloudflare.com/) and open **Workers & Pages**.
2. Install Wrangler in a terminal:

   ```bash
   npm install -g wrangler
   ```

3. Open a terminal in this project folder (the folder containing `worker.js` and `wrangler.toml`) and sign in:

   ```bash
   wrangler login
   ```

4. Open `wrangler.toml` and replace `YOUR-USERNAME` in `ALLOWED_ORIGIN` with your GitHub username. Use only the origin, for example `https://myname.github.io`, with no `/frame-fate/` path.
5. Save your Gemini key as a Worker secret:

   ```bash
   wrangler secret put GEMINI_API_KEY
   ```

   Paste the key when Wrangler prompts you. It will not be included in the repository.

6. Deploy the Worker:

   ```bash
   wrangler deploy
   ```

7. Copy the `workers.dev` URL printed by Wrangler and add `/api/story` to it. Example:

   `https://frame-fate-story.YOUR-SUBDOMAIN.workers.dev/api/story`

## Part D — Connect the game to your Worker

1. Open your published GitHub Pages game.
2. In **03 / Story Engine**, leave **Enable AI when connected** selected.
3. Paste your Worker URL into **Your AI Worker URL**.
4. Start a new movie. The opening is built in; after your first decision, the AI can continue with generated scenes and choices.
5. Your Worker URL is saved in that browser. Your Gemini API key is not stored in the browser.

If you leave the Worker URL blank or select **Offline stories only**, the predefined game works as usual.

## Protect your public AI endpoint

The Worker checks the browser origin, validates the request size and story fields, limits prompt/output sizes, and keeps the provider key server-side. **CORS is not authentication**: a public endpoint can still be called outside a browser by someone who knows its URL and spoofs headers. Before sharing the game widely, add a Cloudflare rate-limiting rule for `/api/story`:

- Cloudflare Dashboard → **Security rules → Create rule → Rate limiting rules**.
- Match URI Path equals `/api/story`.
- Count by IP address.
- A sensible starting point for a small public game is 10 requests per minute per IP, with a short block period.

Cloudflare's available rule fields depend on plan; the Free plan supports path matching and IP counting. Review [Cloudflare rate-limiting rules](https://developers.cloudflare.com/waf/rate-limiting-rules/) and adjust the threshold if real players are blocked. Keep an eye on Gemini API usage and revoke/replace the key if you suspect abuse.

## Local testing

- Open `index.html` directly to test the built-in offline game.
- To test AI locally, deploy the Worker with `ALLOWED_ORIGIN` set to your local development origin (for example `http://localhost:8000`), run a local static server, and use the local Worker URL. Restore the GitHub Pages origin before deploying the public Worker.

## Important notes

- AI generation requires internet access and your own Gemini API key. The API provider may impose rate limits or change its free offering.
- Cloudflare Workers Free is subject to its published quotas and terms.
- Save data is stored in the current browser's local storage. It does not sync between devices.
- This project does not include accounts, multiplayer, cloud saves, or a database.
- Do not commit `.env` files or API keys. `GEMINI_API_KEY` belongs only in Worker Secrets.

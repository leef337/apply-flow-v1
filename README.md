# Easy + Agentic apply (prototype v1)

Single-file interactive prototype for the FINN Jobb apply step. Drop into any
static host (GitHub Pages, Netlify Drop, Vercel) — no build step.

## What's in here

- `index.html` — the entire prototype (React + Babel standalone, all CSS
  inlined). Open it in a browser, or serve it.

## Views

- **Prototype** — iOS clickable prototype of the apply flow.
- **Opportunity tree** — outcome → opportunities → hypotheses → solutions.
- **Impact–Effort** — each prototype solution plotted by effort and impact.
- **Storymap** — activities → backbone → MVP / 1.0 / 2.0 release slices.

## Publishing on GitHub Pages

1. Create a new public repo on GitHub (e.g. `apply-flow-v1`).
2. Upload `index.html` (and this `README.md` if you like) to the repo root.
3. Repo → Settings → Pages → "Deploy from a branch" → `main` / `/ (root)`.
4. After ~1 min, your prototype is live at
   `https://<your-username>.github.io/<repo-name>/`.

## Sharing without GitHub

Drag `index.html` to [app.netlify.com/drop](https://app.netlify.com/drop) for
an instant URL — no account required.

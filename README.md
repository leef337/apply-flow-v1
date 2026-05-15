# FINN Jobs — Easy + Agentic apply (prototype v1)

Single-file interactive prototype for the FINN Jobb apply step. Hosted on
GitHub Pages alongside two small asset folders.

## Files in this repo

```
.
├── index.html                # the prototype (React + Babel standalone)
├── README.md                 # this file
├── assets/
│   └── politiet-logo.png     # the Politiet company logo used on the job ad
└── fonts/
    ├── FINNTypeWebStrippet-Light.ttf
    ├── FINNTypeWebStrippet-Regular.ttf
    ├── FINNTypeWebStrippet-Medium.ttf
    └── FINNTypeWebStrippet-Bold.ttf
```

All references in `index.html` are relative, so the folders need to live next
to it. If you move `index.html` somewhere else, move the two folders with it.

## Views

- **Prototype** — iOS clickable prototype of the apply flow. Use the dev panel
  on the right to switch User state and Version (MVP / 1.0 / 2.0).
- **Opportunity tree** — outcome → opportunities → hypotheses → solutions.
- **Impact–Effort** — every prototype solution plotted by effort and impact.
- **Storymap** — activities → backbone → MVP / 1.0 / 2.0 release slices.

## Version dropdown

- **MVP** — what the prototype currently ships. No Job Match pill, no
  Tailored CV option in the picker.
- **1.0** — adds the Job Match pill + "What is Job Match?" link, plus the
  Tailored CV option.
- **2.0** — Magic Apply. Tap Apply Now on the job ad and the application
  auto-fills row by row, then sends itself.

## Publishing on GitHub Pages

1. Public repo, drag all the files in matching this folder structure.
2. Settings → Pages → Deploy from `main` / `/ (root)` → Save.
3. URL lands at `https://<your-username>.github.io/<repo-name>/`.

## Sharing without GitHub

Drag the whole folder onto [app.netlify.com/drop](https://app.netlify.com/drop)
for an instant URL — keeps the folder layout intact.

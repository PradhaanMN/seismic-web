# 🕸️ Seismic Web — Deployment Guide
## Git → GitHub → Netlify (Step-by-Step)

---

## Prerequisites

| Tool | Version | Install |
|------|---------|---------|
| Git  | ≥ 2.40  | https://git-scm.com |
| GitHub account | — | https://github.com |
| Netlify account | — | https://netlify.com (free) |

---

## Step 1 — Initialise the Local Git Repository

Open a terminal (PowerShell / Bash) inside the project folder:

```bash
cd D:\projects\seismic-web        # navigate to project root

git init                          # initialise empty repo
git add .                         # stage all files
git commit -m "feat: initial Seismic Web simulation"
```

Verify the file tree looks like this:

```
seismic-web/
├── index.html        ← UI + PyScript DSP engine
├── styles.css        ← dark oscilloscope theme
├── pyscript.toml     ← PyScript package config
├── netlify.toml      ← Netlify deployment config
└── DEPLOYMENT.md     ← this guide
```

---

## Step 2 — Create a GitHub Repository

1. Go to https://github.com/new
2. Fill in:
   - **Repository name**: `seismic-web`
   - **Description**: `Bio-inspired perimeter intrusion detection simulation`
   - **Visibility**: Public  *(required for free Netlify deployments)*
3. **Do NOT** tick "Add README" or "Add .gitignore" (your repo is already initialised)
4. Click **Create repository**

GitHub will show you a URL like:
```
https://github.com/<your-username>/seismic-web.git
```

---

## Step 3 — Push to GitHub

Back in your terminal:

```bash
# Link local repo to GitHub remote
git remote add origin https://github.com/<your-username>/seismic-web.git

# Rename default branch to 'main' (modern convention)
git branch -M main

# Push and set upstream tracking
git push -u origin main
```

Refresh the GitHub page — you should see all five files.

---

## Step 4 — Deploy to Netlify

### Option A — Netlify Dashboard (easiest)

1. Log into https://app.netlify.com
2. Click **"Add new site"** → **"Import an existing project"**
3. Choose **GitHub** as the Git provider
4. Authorise Netlify to access your GitHub
5. Select the **`seismic-web`** repository
6. In the **Build settings** form:
   | Field | Value |
   |-------|-------|
   | Branch to deploy | `main` |
   | Build command | *(leave blank)* |
   | Publish directory | `.` |
7. Click **"Deploy seismic-web"**

Netlify will assign a URL like `https://seismic-web-abc123.netlify.app`.

---

### Option B — Netlify CLI

```bash
# Install Netlify CLI globally
npm install -g netlify-cli

# Login to Netlify
netlify login

# From inside your project folder:
netlify init           # connects to your Netlify account + GitHub repo
netlify deploy --prod  # deploy to production
```

---

## Step 5 — Verify the Deployment

1. Open the Netlify URL in a browser.
2. You should see the **Seismic Web** loading screen.
3. Pyodide will initialise (takes ~8–15 s on first load; cached thereafter).
4. Click any trigger button — the canvas should animate and the FFT plot should appear.

> **Tip:** Open DevTools → Console to see PyScript logs if something goes wrong.

---

## Step 6 — Custom Domain (Optional)

In the Netlify dashboard:
1. Site settings → **Domain management** → **Add custom domain**
2. Enter your domain and follow the DNS instructions.
3. Netlify provisions a free **Let's Encrypt** TLS certificate automatically.

---

## Updating the Site

Every `git push` to `main` automatically triggers a new Netlify build:

```bash
git add .
git commit -m "fix: improve TDOA accuracy"
git push
```

Netlify re-deploys within ~10 seconds.

---

## File Reference

| File | Purpose |
|------|---------|
| `index.html` | Complete UI + all PyScript/Python DSP logic |
| `styles.css` | Dark oscilloscope CSS theme |
| `pyscript.toml` | Declares `numpy` & `matplotlib` for Pyodide |
| `netlify.toml` | Sets COOP/COEP headers (required for Pyodide) |

---

## Architecture Notes

```
Browser
  └─ index.html
       ├─ styles.css          (loaded normally)
       ├─ PyScript core.js    (loaded from CDN)
       │    └─ Pyodide WASM   (downloads ~8MB on first visit)
       │         ├─ numpy
       │         └─ matplotlib
       │
       ├─ <script type="py"> — Python DSP Engine
       │       Signal Gen → FFT → Classify → TDOA → Plot
       │
       └─ <script> — JavaScript Canvas Renderer
               Grid → Sensor dots → Wave animation → Alert marker
```

### Cross-Origin Headers (Critical!)

Pyodide requires `SharedArrayBuffer`, which is only available in
**cross-origin isolated** contexts. The `netlify.toml` sets:

```
Cross-Origin-Opener-Policy:   same-origin
Cross-Origin-Embedder-Policy: require-corp
```

Without these headers, Pyodide will silently fall back to a slower
single-threaded mode or fail entirely.

---

## Troubleshooting

| Symptom | Fix |
|---------|-----|
| Infinite loading screen | Open DevTools → check for COOP/COEP header errors |
| `SharedArrayBuffer is not defined` | Add the two CORS headers in `netlify.toml` |
| `ModuleNotFoundError: numpy` | Confirm `pyscript.toml` is in the root and has `packages = ["numpy", "matplotlib"]` |
| Blank canvas after trigger | Open DevTools console — Python exceptions are logged there |
| Slow first load (~15 s) | Normal — Pyodide downloads ~8 MB of WASM. Subsequent loads use browser cache. |

---

*Generated for Seismic Web v2.0 — February 2026*

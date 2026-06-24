# Filecoin Pay — Fee & Burn Calculator

An interactive model of Filecoin Pay fees and FIL burn under network-growth scenarios. It reconstructs the FIP-1249 FPV growth models (decaying, peak & settle, Wasabi) and overlays the fee structure for **mainnet v1.2.x**, **v1.3.0**, and a **redesign** with a non-FIL/non-USDFC surcharge.

It is a single, self-contained HTML file. The only external dependency is the Chart.js charting library, loaded from a CDN.

## Run it locally

Open `index.html` in any modern browser (double-click). An internet connection is needed for the Chart.js CDN.

## Publish it on GitHub Pages

1. Create a new GitHub repository (e.g. `filecoin-pay-fee-calculator`).
2. Add `index.html` (and optionally this `README.md`) to the repo — drag-and-drop via **Add file → Upload files** in the GitHub web UI, or push with git:

   ```bash
   git init
   git add index.html README.md
   git commit -m "Filecoin Pay fee calculator"
   git branch -M main
   git remote add origin https://github.com/<your-username>/<repo>.git
   git push -u origin main
   ```

3. In the repo: **Settings → Pages → Build and deployment** → Source: **Deploy from a branch**, Branch: **main** / **/(root)** → **Save**.
4. Wait ~1 minute. The calculator is live at:

   ```
   https://<your-username>.github.io/<repo>/
   ```

   Share that URL with anyone. Because the file is named `index.html`, it loads at the site root automatically.

## Other one-click hosting options

- **Netlify Drop** — drag `index.html` onto <https://app.netlify.com/drop> for an instant public URL.
- **Vercel** / **Cloudflare Pages** — connect the repo, or drag the file in.
- **Just send the file** — `index.html` opens in any browser locally (still needs internet for the chart library).

## Want it fully offline / zero-dependency?

The chart library is the only thing loaded from the network. It can be inlined into the file so the calculator works with no internet at all — ask and it can be bundled.

## Methodology & data

See `Filecoin-Pay-Fee-Redesign.md` for the full fee/burn taxonomy, the real mainnet + calibnet figures behind the defaults, and the modelling assumptions.

- FIP-1249 explorer: <https://irenegia.github.io/FIP-discussion-1249-design-explorer/subnet-weights-explorer.html>
- Filecoin Data Portal: <https://filecoindataportal.xyz>

*All figures are illustrative projections. The dataset-per-volume and proof-fee-per-dataset links are simplifying proxies you can tune in the sidebar.*

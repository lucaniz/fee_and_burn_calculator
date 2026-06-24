# Filecoin Pay — Fee &amp; Burn Redesign

> An interactive model and supporting analysis of fee-driven FIL burn in the **Filecoin Onchain Cloud (FOC)**, built to inform a redesign of the Filecoin Pay fee structure.

![status](https://img.shields.io/badge/status-research%20preview-3fb6f7)
![type](https://img.shields.io/badge/build-static%20single%20file-22c55e)
![deps](https://img.shields.io/badge/dependencies-Chart.js%20(CDN)-a78bfa)
![scope](https://img.shields.io/badge/scope-FOC%20%2F%20PDP%20only-f59e0b)

---

## Overview

Filecoin Pay charges several fees, but only a subset actually burns FIL — and the component doing most of the burning is being removed in the v1.3.0 upgrade. This project quantifies the current fee/burn picture from on-chain data, proposes a treatment for fees paid in non-FIL / non-USDFC tokens, and ships an interactive calculator that projects fee revenue and FIL burn under network-growth scenarios.

The calculator is a single, self-contained HTML page. It reconstructs the FPV (Filecoin Pay Volume) growth models from the FIP-1249 scenario explorer and overlays the fee structure for three regimes — **mainnet v1.2.x**, **v1.3.0**, and a configurable **redesign** — so the burn impact of each lever can be compared side by side.

## Key findings

| Finding | Detail |
|---|---|
| The 0.5% network fee is economically marginal | Across all of mainnet history it has produced ≈ **2 USDFC** in fees. |
| The createDataSet ("sybil") fee carries the burn | ≈ **74.8 USDFC** (≈ **97%** of the auction pool), converted to **52.7 FIL burned** across 32 Dutch auctions. |
| v1.3.0 removes that burn | The createDataSet fee drops to 0.025 USDFC and is **paid to the storage provider instead of burned**, cutting fee-driven burn by roughly **7–18×** at a given network size. |
| The proof fee is a separate, direct FIL burn | ≈ **1.31 FIL** all-time, at 0.00023 FIL/TiB per `provePossession` — **PDP only**, not the legacy sealed-sector network. |
| Non-FIL / non-USDFC tokens do not accrue FIL value | They only burn FIL if their auction pool is claimed, which for illiquid tokens may never happen — the motivation for a payment-token surcharge. |

Full reasoning, data, and the redesign proposal are in [`Filecoin-Pay-Fee-Redesign.md`](./Filecoin-Pay-Fee-Redesign.md).

## Live demo

Once deployed (see [Deployment](#deployment)), the calculator is available at:

```
https://<your-username>.github.io/<repo>/
```

## Repository contents

| File | Description |
|---|---|
| [`index.html`](./index.html) | The interactive fee &amp; burn calculator (single file, no build step). |
| [`Filecoin-Pay-Fee-Redesign.md`](./Filecoin-Pay-Fee-Redesign.md) | The analysis: fee/burn taxonomy, on-chain figures, non-FIL surcharge proposal, scenario findings. |
| `README.md` | This file. |

## The calculator

Three control groups drive the model:

1. **Network size (FPV)** — choose a growth model (Decaying, Peak &amp; settle, Wasabi, or Custom per-quarter), set the starting volume, FIL price, and the PDP data stored.
2. **Fee regime** — switch between v1.2.x / v1.3.0 / Redesign presets and tune the network fee, createDataSet fee and its destination (burned vs. paid to SP), and the proof-fee rate.
3. **Token mix &amp; non-FIL surcharge** — set the share of volume settled in FIL / USDFC / other tokens, and the surcharge, claim rate, and auction efficiency applied to them.

Outputs update live: FIL burn by category, a cumulative regime comparison, the FPV trajectory, and the FIP-1249 block-reward split for macro context. Every parameter is documented in an in-app glossary (the **"Parameters &amp; growth models explained"** panel at the top of the page).

## Quick start

Open `index.html` in any modern browser. No build step or server is required; an internet connection is used only to load the Chart.js library from a CDN.

## Deployment

### GitHub Pages

1. Create a repository and add `index.html` (and this `README.md`).
2. Push, or upload via **Add file → Upload files** in the web UI.
3. Go to **Settings → Pages → Build and deployment**, set **Source: Deploy from a branch**, **Branch: `main` / `/(root)`**, and save.
4. The site goes live at `https://<your-username>.github.io/<repo>/` within ~1 minute.

```bash
git init
git add index.html README.md Filecoin-Pay-Fee-Redesign.md
git commit -m "Filecoin Pay fee & burn calculator"
git branch -M main
git remote add origin https://github.com/<your-username>/<repo>.git
git push -u origin main
```

### Other static hosts

- **Netlify Drop** — drag `index.html` onto <https://app.netlify.com/drop> for an instant URL.
- **Vercel** / **Cloudflare Pages** — connect the repository or upload the file.

### Embedding in Notion

Paste the deployed URL into a Notion page, or use a `/embed` block to render the calculator live inside the document.

## Methodology

- **Growth models** reproduce the FIP-1249 "Block-reward split — Scenario Explorer" (decaying, peak-and-settle, Wasabi) and add a custom per-quarter path.
- **Fee structure** follows the Filecoin Pay / FWSS / PDPVerifier mechanics documented in the Filecoin Onchain Cloud protocol context.
- **Defaults** are anchored to observed mainnet data (Nov 2025 – Jun 2026); e.g., the proof-fee branch reproduces the measured ≈ 0.52 FIL/quarter at the current ≈ 25 TiB actively proven.
- **USDFC → FIL conversion** approximates the Dutch-fee-auction as `FIL burned ≈ pool ÷ FIL price`, with adjustable auction efficiency.

## Scope

This models the **Filecoin Onchain Cloud (FOC / PDP) subsystem only** — Filecoin Pay volume, PDP-stored data, and the PDP proof fee. It is **not** the whole Filecoin network: the proof fee is charged exclusively on `PDPVerifier` proofs, never on legacy sealed-sector (WindowPoSt) storage. The block-reward panel is the single network-wide mechanism, and it is gated by FOC volume.

## Caveats

All figures are **illustrative projections**. The dataset-per-volume and stored-data-per-FPV relationships are simplifying proxies exposed as tunable inputs. The mainnet fee auction is a small sample (32 claims, ≈ 76 USDFC of lifetime pool): relative category shares are robust, absolute projections are not. Validate against current on-chain data before any figure informs a decision.

## Data sources &amp; references

- Filecoin Onchain Cloud protocol context &amp; indexed event data (mainnet &amp; calibnet).
- Filecoin Data Portal — <https://filecoindataportal.xyz>
- FIP-1249 "Block-reward split — Scenario Explorer" — <https://irenegia.github.io/FIP-discussion-1249-design-explorer/subnet-weights-explorer.html>
- filecoin-services releases (v1.0.0 → v1.3.0) — <https://github.com/FilOzone/filecoin-services>

## License

No license is set yet. Add a `LICENSE` file before publishing (MIT is a common choice for tools like this).

# Filecoin Pay — Fee &amp; Burn: Analysis and Fee Calibration

*Working analysis · data through 24 Jun 2026 · mainnet (v1.2.1) and calibnet (v1.3.0) side by side*

---

## 1. Summary

This document maps which Filecoin Pay fees burn FIL, quantifies each from on-chain data, and works through two calibration questions:

1. **How much to augment the proof fee and/or the payment (network) fee** to produce a target FIL burn.
2. **How much to charge for fees paid in tokens other than FIL/USDFC.**

Three facts frame the calibration:

- The **0.5% network fee scales with volume.** Across mainnet history so far it has produced ≈ 2 USDFC, because settled volume is tiny; under the FIP-1249 growth scenarios it becomes the dominant percentage-based source.
- On mainnet (v1.2.x) the **0.1 USDFC createDataSet ("sybil") fee** has funded ≈ 97% of the fee-auction pool (≈ 74.8 USDFC → 52.7 FIL burned). **Under v1.3.0 this fee is paid to the storage provider, not burned** (verified below).
- The **proof fee** is a separate, direct FIL burn (≈ 1.31 FIL all-time at 0.00023 FIL/TiB), and it is **PDP-only** — it never touches legacy sealed-sector storage.

## 2. Assumptions

- **Fee auctions close.** Every `burnForFees` auction pool is eventually cleared, so accrued fees in any token *do* convert to a FIL burn. Tokens differ only in the **efficiency and timeliness** of that conversion: native FIL burns immediately at par; USDFC clears near par on the ≈ weekly Dutch auction; other tokens clear at a discount and/or less frequently. This document does not assume any pool sits unclaimed indefinitely.
- **Scope is FOC / PDP**, not the whole Filecoin network (see §7). Figures are denominated in USDFC and FIL as indexed; USDFC ≈ US$1.
- Network growth follows the FIP-1249 FPV models; FPV = settled deal-payment value per quarter (not the fee).

## 3. What gets burned, and what is "a fee"

Only **FIL** is ever truly burned. USDFC (and any other token) fees accumulate in a per-token pool and become a FIL burn when that pool is auctioned via `burnForFees` — the winning bidder sends FIL, which is burned to `f099`.

### 3a. Direct FIL burns

| Item | Mechanism | Rate / size |
|---|---|---|
| **Proof fee** | `provePossession` burns a per-TiB FIL fee from the SP's wallet | 0.00023 FIL/TiB default (`Fees.sol`, governable) |
| **Auction burn** | `burnForFees` — bidder sends FIL to claim the accumulated pool; FIL burned | market-priced via Dutch auction |
| **Network fee on FIL rails** | 0.5% taken at settlement on FIL-denominated rails, burned directly | 0.5% |
| **Gas** | EIP-1559 base-fee burn on every transaction | network-determined |

PDPVerifier also holds a refundable 0.1 FIL cleanup deposit per dataset — locked, not burned, unless cleanup never runs.

### 3b. USDFC fees that feed the auction

| Item | Mechanism | Rate / size | Fate |
|---|---|---|---|
| **Settlement network fee** | 0.5% of gross at `settleRail` on USDFC rails | 0.5% | → auction pool (USDFC) |
| **createDataSet fee** | v1.2.x: 0.1 USDFC burn rail → contract; v1.3.0: 0.025 USDFC to SP | 0.1 → 0.025 USDFC | v1.2.x: burned via pool · v1.3.0: not burned |

### 3c. Fees that are not burned

Operator commission (0 today), storage streaming rate (2.5 USDFC/TiB/month FWSS), v1.3.0 per-operation fees (all to the SP), and CDN/cache-miss egress (to FilBeam/SP).

## 4. Current burn by category (on-chain data)

### Mainnet — v1.2.1, 5 Nov 2025 → 24 Jun 2026

| Category | Token | Amount | Notes |
|---|---|---|---|
| Gas (base protocol) | FIL | 278.4 FIL | 1.57M txs; not a Filecoin Pay fee |
| Proof fee | FIL | 1.31 FIL | 59,489 proofs, direct SP burn |
| Auction burn (`burnForFees`) | FIL | 52.69 FIL | 32 claims; claimed 69.47 USDFC from the pool |
| Legacy createDataSet burn | FIL | 50.6 FIL | 506 datasets × 0.1 FIL, pre-23 Mar — discontinued |

**Auction-pool sources (the ≈ 76.5 USDFC behind the 52.7 FIL burn):**

| Source | USDFC | Share |
|---|---|---|
| createDataSet / sybil fee (748 × ≈ 0.0995) | ≈ 74.43 | **97.3%** |
| 0.5% network fee — streaming | 0.99 | 1.3% |
| 0.5% network fee — one-time payments | 1.09 | 1.4% |

### Calibnet — v1.3.0 (test network, real mechanics)

Proof fee ≈ 2.46 FIL (7.69M proofs at 12×/day). FIL-denominated rails settle and burn directly (0.0025 FIL on 0.50 gross). Operator commission is non-zero in test. The createDataSet burn rail is gone (see §5).

## 5. Effect of v1.3.0 on fee-driven burn

**Verified from calibnet data.** The v1.2.x sybil-burn sink (a single payee receiving exactly 0.0995 USDFC per createDataSet) ran from 2026-03-20 and **stopped on 2026-06-08 — the calibnet v1.3.0 upgrade date.** After the upgrade there is no 0.0995 burn sink; post-upgrade one-time payments are small (≈ 0.002–0.003 USDFC) and spread across many SP/FilBeam addresses. This confirms the v1.3.0 mechanic: the createDataSet fee (0.025 USDFC) is **accrued to the storage provider via the PDP rail's lifecycle reserve, not burned.**

Consequence for fee-driven burn: under v1.3.0 the auction pool no longer grows from createDataSet. The two remaining burn-generating fees are the **network (payment) fee** — USDFC → auction → FIL, scaling with settled volume — and the **proof fee** — FIL, direct, scaling with stored data. The calibration below sizes both.

## 6. Calibration 1 — augmenting the proof fee and/or payment fee

The two levers behave differently:

- **Payment (network) fee** — a percentage of settled volume, paid by clients, in USDFC; reaches FIL via the auction (efficiency `e_usdfc ≈ 1`). Per-quarter FIL burn: `f_pay × FPV × e_usdfc ÷ P_FIL`.
- **Proof fee** — FIL per TiB per proof, paid by SPs, burned directly (no auction dependency). Per-quarter FIL burn: `r_proof × TiB × proofs_per_day × 90`.

To hit a target burn `T` (FIL/quarter), invert either:

```
f_pay   = T × P_FIL ÷ (FPV × e_usdfc)
r_proof = T ÷ (TiB × proofs_per_day × 90)
```

### Worked example

A representative, internally-consistent point: **≈ 2 PiB stored (2,048 TiB), all storage volume** at 2.5 USDFC/TiB/month ⇒ FPV ≈ **15,360 USDFC/quarter**; FIL = $0.70; 1 proof/day.

| Lever / rate | FIL burned per quarter |
|---|---|
| **Payment fee** 0.5% | 109.7 |
| Payment fee 1.0% | 219.4 |
| Payment fee 1.5% | 329.1 |
| Payment fee 2.0% | 438.9 |
| **Proof fee** 0.00023 FIL/TiB | 42.4 |
| Proof fee 0.00046 (2×) | 84.8 |
| Proof fee 0.00069 (3×) | 127.2 |
| Proof fee 0.00115 (5×) | 212.0 |

Reading: at scenario-scale volume the **payment fee** is the larger lever (it scales with dollar volume), while the **proof fee** is the more *certain* lever (FIL-denominated, burned directly, no auction step). At this network size, a 5× proof fee (212 FIL/q) costs SPs ≈ $148/quarter against ≈ $15.4k/quarter of storage revenue — i.e. ≈ 1% of revenue — so the proof fee has substantial headroom as an SP-borne cost.

### Replacing the old per-dataset burn

The retired sybil fee burned a **flat 0.1 USDFC per dataset**, independent of size. Replacing that with the percentage payment fee requires a rate that depends entirely on per-dataset volume — `f = 0.1 ÷ (lifetime settled USDFC per dataset)`:

| Avg dataset | Lifetime volume (12 mo) | Payment fee to match 0.1 USDFC/dataset |
|---|---|---|
| 0.1 TiB | 3 USDFC | **3.3%** |
| 1 TiB | 30 USDFC | 0.33% |
| 10 TiB | 300 USDFC | 0.033% |

So a percentage fee replaces the per-dataset burn cleanly only for large datasets; for small datasets it would need an impractically high rate. If per-dataset sybil-resistance / burn is a goal in the v1.3.0 world, a **small flat burnable per-dataset fee** (separate from the 0.025 USDFC SP fee) is the efficient instrument; the percentage fee is better understood as a volume-scaling burn that matters at scale.

These three levers — payment-fee %, proof-fee rate, and an optional burnable per-dataset fee — are the inputs in the calculator's Fee-regime panel.

## 7. Calibration 2 — charging for non-FIL / non-USDFC tokens

Under the auction-closure assumption, a non-FIL/non-USDFC token's fees **do** burn FIL, but the pool clears at a **discount** (a Dutch auction for a less-liquid token settles below par) and on a **longer cadence**. Let `e_other` be the realised conversion efficiency (FIL value burned ÷ fee value accrued) for that token, and `e_usdfc` the same for USDFC (≈ par).

To make a non-FIL token contribute the **same FIL burn per dollar of fee** as USDFC, set its fee so that `f_other × e_other = f_base × e_usdfc`:

```
f_other       = f_base × (e_usdfc ÷ e_other)
surcharge(pp) = f_base × (e_usdfc ÷ e_other − 1)
```

### Surcharge for parity (base fee 0.5%, e_usdfc = 100%)

| Other-token clearing efficiency `e_other` | Multiplier | Resulting fee | Surcharge |
|---|---|---|---|
| 90% | 1.11× | 0.56% | +0.06 pp |
| 75% | 1.33× | 0.67% | +0.17 pp |
| 70% | 1.43× | 0.71% | +0.21 pp |
| 60% | 1.67× | 0.83% | +0.33 pp |
| 50% | 2.00× | 1.00% | +0.50 pp |
| 40% | 2.50× | 1.25% | +0.75 pp |

The surcharge is governed by one estimate: how deeply that token's fee-auction clears below par. A liquid, near-par token warrants only a few basis points; an illiquid token clearing at a 40–50% discount warrants roughly doubling the fee. The clearing efficiency can be set per token at approval time and revised from observed auction outcomes (realised FIL-per-dollar at each token's auctions).

Two refinements, both small relative to the discount:

- **Time value of the delay.** If a token's pool clears every `D` weeks versus USDFC's ≈ 1, the burn is delayed; at discount rate `ρ` the cost ≈ `ρ × (D − 1) ÷ 52`. Fold it into `e_other`.
- **FIL as the benchmark.** Native FIL converts at 100% instantly. It can be priced at — or slightly below — the USDFC baseline (e.g. 0.4% vs 0.5%), since it is the ideal case; USDFC sits at the baseline; everything else carries the parity surcharge above.

In the calculator, this maps directly to the **USDFC auction conversion efficiency**, **Other-token auction conversion efficiency**, and **Surcharge on other tokens** sliders, so a given clearing-efficiency assumption can be checked against the resulting burn.

## 8. Sources

- Filecoin Onchain Cloud protocol context &amp; indexed event data (mainnet &amp; calibnet) — `pdp_proof_fee_paid`, `fp_burn_for_fees`, `fp_rail_settled`, `fp_one_time_payment`, `fp_rail_created`, `pdp_data_set_created`, `tx_meta`.
- Filecoin Data Portal — <https://filecoindataportal.xyz>
- FIP-1249 "Block-reward split — Scenario Explorer" — <https://irenegia.github.io/FIP-discussion-1249-design-explorer/subnet-weights-explorer.html>
- filecoin-services releases (v1.0.0 → v1.3.0) — <https://github.com/FilOzone/filecoin-services>

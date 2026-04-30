# OkinawaTrader — Visual Demo

A walk through the OkinawaTrader iOS app captured live from the iPhone 17 Simulator
on commit `dffc318` (April 30, 2026). Every screenshot below is a real frame from
the running app, not a mockup.

OkinawaTrader is a SwiftUI commodity-hedging and compliance platform for Indian
food-import merchants. It pairs an on-chain LMSR automated market maker for six
commodities with a dual-ledger IFSCA reporting flow and six stakeholder role views
(Merchant, Liquidity Provider, Operator, Supplier, Auditor, Banking Partner).

---

## 1. Welcome / cold launch

![Welcome screen](../screenshots/welcome_hero.png)

The cold-launch hero positions the platform around three guardrails the rest of
the app keeps coming back to:

- **Policy-driven compliance** — FEMA, IFSCA and RBI rule sets are wired into
  every flow, not bolted on at the end.
- **Real-time mandi pricing** — LMSR pools for six Indian commodities give a
  spot price and a marginal-cost curve at every block.
- **End-to-end procurement** — purchase orders, hedges, fulfilment and
  settlement are tracked on one timeline.

Two entry points: `Get Started` opens the onboarding flow, `Browse Markets First`
drops the user straight into a read-only market view (no auth needed).

---

## 2. Onboarding — identity and jurisdiction

![Onboarding stage 1 of 5](../screenshots/onboarding_identity.png)

Stage 1 of a five-stage onboarding (`Identity → Role → Details → Confirm`). Two
things are decided here that ripple through the rest of the app:

- **Jurisdiction.** India (INR) vs United States (USD) selects the KYC regime
  and the currency formatting used everywhere downstream.
- **Identity provider.** Sign in with Apple is the recommended path; an escape
  hatch — `Browse markets without signing in` — keeps the public mandi view
  available without an account.

Once identity is verified the user is funnelled into role selection (Merchant,
LP, Operator, Supplier, Auditor, or Banking Partner), then role-specific KYC.

---

## 3. Today's Mandi — public market view

![Today's Mandi market view](../screenshots/market_today_mandi.png)

The unauthenticated market view. Six commodities track live LMSR spot prices,
quoted in ₹/quintal:

| Commodity | Grade | Spot |
|---|---|---|
| Maize | Feed grade | ₹2,100 |
| Wheat | Atta & Maida | ₹2,349 |
| Sugar | Refined M-grade | ₹3,602 |
| Steel | TMT & packaging | ₹55,029 |
| Rice | Basmati & non-basmati | ₹3,154 |
| Urad Dal | Black Gram | ₹7,885 |

The `LIVE • feed • test` chip in the header reflects the data source — the
in-process mock feed in this build, swappable to the live IFSCA-attested feed in
production. `Enter Platform` upgrades the session to an authenticated role view.

---

## 4. Commodity drill-in — Maize

![Maize commodity detail](../screenshots/market_maize_detail.png)

Tapping a row opens the per-commodity sheet. The header carries the indicative
mandi rate (₹2,100/qtl for Maize), with a price-history chart that fills in as
live ticks arrive. The empty state — *"Not enough data yet. Live ticks are
accumulating"* — is what a fresh-boot Simulator session looks like before the
mock feed has produced enough samples to plot.

The collapsed `Liquidity detail` row is the entry point into the LMSR pool
internals.

---

## 5. LMSR pool internals

![Maize LMSR pool detail expanded](../screenshots/market_maize_lmsr.png)

Expanding `Liquidity detail` exposes the full LMSR state for this commodity.
This is the same data structure the LP and Operator role views are built on top
of:

| Field | Value | What it means |
|---|---|---|
| Spot (INR) | ₹2K | `spotPrice` from `Models.swift` — instantaneous market price |
| TVL | ₹14.6 Cr | Total value locked across both sides of the pool |
| Depth `b` | 50000 | LMSR liquidity parameter — controls price sensitivity |
| Q filled | 0 | Net quantity traded so far |
| Sat | 50.0 % | Saturation of the pool against its cap |
| Cap | ₹2,520 | Per-trade ceiling enforced by `OperatorPolicy` |
| 100u impact | +0.050 % | Marginal price shift from a 100-unit buy |
| 1K impact | +0.500 % | Marginal price shift from a 1,000-unit buy |
| Open int | 0 L | Open interest in lots |

The `executionCost(delta:)` and `priceImpact(delta:)` helpers in `Models.swift`
are what produce these numbers; the same code path runs both in the
`MockBlockchainService` (default in Debug) and against the on-chain
`LMSRAMMPool.sol` contract on Sepolia.

---

## 6. Liquidity Provider — onboarding capital

![LP onboarding — capital commitment](../screenshots/lp_onboarding_capital.png)

The LP role's KYC and capital-commitment screen. LPs sign a capital pledge and
nominate which pools they will provide depth into; the platform uses this to
seed the on-chain `LMSRAMMPool` and to compute their pro-rata share of fees and
slippage P&L.

---

## 7. Liquidity Provider — pool dashboard

![LP dashboard — pools and P&L](../screenshots/lp_dashboard_pools.png)

The post-onboarding LP dashboard. Each row is a pool the provider is exposed
to, with running P&L, current saturation, and quick actions to deposit or
withdraw. This is the screen LPs spend the most time on day-to-day.

---

## How the screenshots were captured

- **Device:** iPhone 17 Simulator, iOS 26.2
- **Build:** `dffc318` on `main`, Debug configuration
- **Chain mode:** `MockBlockchainService` (in-process LMSR, no Sepolia round-trips)
- **Feed:** mock feed; `LIVE • feed • test` chip indicates the data source

Each frame was captured with the Simulator's `Cmd+S` shortcut and stored under
[`/screenshots`](../screenshots/). To regenerate the set:

1. Open `OkinawaTrader.xcodeproj` in Xcode.
2. Select the `iPhone 17` simulator destination.
3. Run the app (`Cmd+R`).
4. Walk through Welcome → Onboarding (or `Browse Markets First`) → Today's Mandi
   → tap a commodity → expand `Liquidity detail`.
5. Press `Cmd+S` at each stop, then move the PNGs from `~/Desktop` into
   `/screenshots`.

---

## Where to go next

- Architecture overview and role model: [`/README.md`](../README.md)
- LMSR math and pool mechanics: [`Models.swift`](../Models.swift) — see
  `CommodityPool`, `executionCost`, `priceImpact`
- Compliance ledger: [`Services/IFSCALedgerService.swift`](../Services/IFSCALedgerService.swift)
- Stakeholder feature matrix:
  [`STAKEHOLDER_FEATURES_AND_DEPLOYMENT.md`](../STAKEHOLDER_FEATURES_AND_DEPLOYMENT.md)
- Mainnet roadmap: [`MAINNET_ROADMAP.md`](../MAINNET_ROADMAP.md)

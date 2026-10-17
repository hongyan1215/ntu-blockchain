---
theme: seriph
title: Perp DEX Liquidation Mechanisms
info: |
  Comparative analysis of GMX, dYdX v4, and Hyperliquid liquidation engines.
  Topic J — Bitcoin & Cryptocurrency, Spring 2026.
class: text-center
highlighter: shiki
lineNumbers: false
drawings:
  persist: false
transition: slide-left
mdc: true
fonts:
  sans: 'Inter'
  serif: 'IBM Plex Serif'
  mono: 'JetBrains Mono'
---

<style>
@import './style.css';
</style>

<div class="absolute inset-0 cover-bg flex flex-col items-center justify-center">

<div class="text-xs tracking-[0.4em] opacity-60 mb-4">NTU · BITCOIN & CRYPTOCURRENCY · SPRING 2026</div>

# <span class="text-white">Liquidation Mechanisms<br/>in Perp DEXes</span>

<div class="text-2xl mt-4 opacity-90">
  <span class="text-orange-400">GMX</span>
  &nbsp;·&nbsp;
  <span class="text-purple-300">dYdX v4</span>
  &nbsp;·&nbsp;
  <span class="text-green-400">Hyperliquid</span>
</div>

<div class="text-lg opacity-70 mt-3">A comparative study of design, safety, and the UX trade-off</div>

<div class="mt-16 text-base">
  <b>黃泓諺</b> &nbsp;Hong-Yan Huang &nbsp;·&nbsp; R14922156
</div>
<div class="text-sm opacity-60 mt-1">Topic J · May 2026</div>

</div>

---

# Why this matters

<div class="text-lg leading-relaxed">

Perp DEXes settle **tens of billions** in daily notional. The liquidation engine decides:

- Who pays for losing trades
- How fast bad debt is closed
- **Who eats the loss when markets gap**

</div>

<div class="mt-8 grid grid-cols-3 gap-6">

<div class="p-5 rounded-lg bg-orange-50 border-l-4 border-orange-500">
  <div class="text-2xl mb-2">🚀</div>
  <div class="font-bold">UX</div>
  <div class="text-sm opacity-75">high leverage, low fees, partial liquidations</div>
</div>

<div class="p-5 rounded-lg bg-blue-50 border-l-4 border-blue-500">
  <div class="text-2xl mb-2">🛡️</div>
  <div class="font-bold">Safety</div>
  <div class="text-sm opacity-75">conservative MM, deep insurance, fast oracles</div>
</div>

<div class="p-5 rounded-lg bg-green-50 border-l-4 border-green-500">
  <div class="text-2xl mb-2">🌐</div>
  <div class="font-bold">Decentralization</div>
  <div class="text-sm opacity-75">permissionless keepers, no admin pause</div>
</div>

</div>

<v-click>

<div class="mt-8 text-xl text-center italic opacity-80">
  No platform wins all three. This talk maps the trade-offs.
</div>

</v-click>

---

# Vocabulary in 30 seconds

<div class="text-base">

| Term             | What it means                                                        |
| ---------------- | -------------------------------------------------------------------- |
| **Mark price**   | Price used for PnL & liquidation checks (≠ last trade)               |
| **IM / MM**      | Initial / Maintenance margin                                         |
| **Liquidator**   | Actor that submits the closing tx for a reward                       |
| **Insurance fund** | Pooled capital that absorbs shortfalls                             |
| **ADL**          | Auto-deleveraging — force-close profitable positions to clear bad debt |

</div>

---
layout: section
---

# Part 1
## The Architectural Landscape

---
class: diagram-slide
---

# Three architectures, three philosophies

<div class="grid grid-cols-3 gap-4 mt-4">

<div class="border-t-4 border-orange-500 bg-orange-50 rounded p-2">

<div class="text-center font-bold text-orange-700 mb-1">🟠 GMX v2 · Pool-counterparty</div>

```mermaid {scale: 0.6}
flowchart TB
    G1[Trader] --> G2[Smart contracts<br/>Arbitrum / Avalanche]
    G4[Chainlink<br/>Data Streams] --> G2
    G5[Keeper network] --> G2
    G2 --> G3[GM / GLV<br/>liquidity pool]
    classDef pool fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    classDef oracle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    class G3 pool
    class G4 oracle
```

</div>

<div class="border-t-4 border-purple-500 bg-purple-50 rounded p-2">

<div class="text-center font-bold text-purple-700 mb-1">🟣 dYdX v4 · App-chain CLOB</div>

```mermaid {scale: 0.6}
flowchart TB
    D1[Trader] --> D2[Cosmos SDK<br/>validator set]
    D4[Validator oracle<br/>vote extensions] --> D2
    D5[Insurance fund] --> D2
    D2 --> D3[On-chain<br/>order book]
    classDef clob fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    classDef oracle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    classDef pool fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    class D3 clob
    class D4 oracle
    class D5 pool
```

</div>

<div class="border-t-4 border-green-600 bg-green-50 rounded p-2">

<div class="text-center font-bold text-green-700 mb-1">🟢 Hyperliquid · CLOB + Vault</div>

```mermaid {scale: 0.6}
flowchart TB
    H1[Trader] --> H2[HyperCore L1<br/>validators]
    H4[Weighted CEX<br/>median oracle 3s] --> H2
    H5[HLP backstop<br/>vault] --> H2
    H2 --> H3[On-chain<br/>order book]
    classDef clob fill:#e8f5e9,stroke:#2e7d32,stroke-width:2px
    classDef oracle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    classDef pool fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    class H3 clob
    class H4 oracle
    class H5 pool
```

</div>

</div>

---

# Architecture cheat-sheet

|              | <span class="gmx">GMX v2</span>     | <span class="dydx">dYdX v4</span>      | <span class="hl">Hyperliquid</span>   |
| ------------ | ---------------------------- | -------------------------------- | --------------------------------- |
| Venue        | Arbitrum / Avalanche L2      | Cosmos SDK app-chain             | HyperCore L1                      |
| Matching     | **No book** — AMM vs pool    | On-chain CLOB                    | On-chain CLOB + HLP vault         |
| Counterparty | GM / GLV LPs                 | Other side + insurance fund      | Other side + HLP                  |
| Oracle       | Chainlink Data Streams       | Validator vote-extensions        | Weighted CEX median, 3 s          |
| Liquidator   | Permissionless keeper bots   | Liquidation daemon (any node)    | Anyone (market) + HLP (backstop)  |

---

# Leverage and margin

|                | <span class="gmx">GMX</span> | <span class="dydx">dYdX v4</span>    | <span class="hl">Hyperliquid</span>  |
| -------------- | --------------- | -------------------- | ------------------------------------ |
| Max leverage   | up to 100×      | 1–50× (tier-based)   | 3–40× (asset-tiered)                 |
| Modes          | Pool collateral | Cross + isolated     | Cross + isolated                     |
| MM rule        | 0.25–1 % floor  | Fixed MMF per market | $\text{MMR} = \text{IMR}/2$, by tier |

<v-click>

<div class="mt-6 p-4 rounded bg-green-50 border-l-4 border-green-600">
🔍 <b>Tier deep-cut (HL):</b> BTC 40× → drops to 20× above $150M OI per account.<br/>
Aggressive UX, but structural defense against whale cascades.
</div>

</v-click>

---
layout: section
---

# Part 2
## The Liquidation Waterfalls

---
class: diagram-slide
---

# Liquidation waterfall — side by side

<div class="grid grid-cols-3 gap-3 mt-2">

<div class="p-2 rounded border-t-4 border-orange-500 bg-orange-50">
<div class="text-orange-700 font-bold text-center mb-1">GMX</div>

```mermaid {scale: 0.42}
flowchart TD
    A([Undercollateralized]) --> B[Keeper closes<br/>at oracle price]
    B --> C[GM pool absorbs PnL]
    C --> D{pendingPnL ratio<br/>exceeds cap?}
    D -- No --> E([Settled])
    D -- Yes --> F[ADL profitable peers<br/>synthetic only]
    F --> E
    classDef adl fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef bs fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    class F adl
    class C bs
```

</div>

<div class="p-2 rounded border-t-4 border-purple-500 bg-purple-50">
<div class="text-purple-700 font-bold text-center mb-1">dYdX v4</div>

```mermaid {scale: 0.42}
flowchart TD
    A([Undercollateralized]) --> B[Liquidation order<br/>onto CLOB]
    B --> C{Matched at<br/>oracle ± 1.5× MMF?}
    C -- Yes --> E([Settled, fees → IF])
    C -- No --> F[Insurance Fund<br/>covers shortfall]
    F --> H[Deleveraging<br/>if IF depleted]
    H --> E
    classDef adl fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef bs fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    class H adl
    class F bs
```

</div>

<div class="p-2 rounded border-t-4 border-green-600 bg-green-50">
<div class="text-green-700 font-bold text-center mb-1">Hyperliquid</div>

```mermaid {scale: 0.42}
flowchart TD
    A([Undercollateralized]) --> B[Tier 1: market liq<br/>≤20%, 30 s cooldown]
    B --> C{Back above MM?}
    C -- Yes --> E([Settled])
    C -- No --> D[Tier 2: HLP backstop<br/>absorbs position]
    D --> G[Tier 3: ADL by<br/>PnL × leverage]
    G --> E
    classDef adl fill:#ffebee,stroke:#c62828,stroke-width:2px
    classDef bs fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    class G adl
    class D bs
```

</div>

</div>

<div class="caption caption-red mt-2 text-xs">
👁️ <b>Hyperliquid's red path:</b> Tier 1 Market Liq → 30 s cooldown → Tier 2 HLP backstop → Tier 3 ADL
</div>

---

# Liquidation price formulas

<div class="grid grid-cols-3 gap-4 text-sm">

<div class="p-4 rounded bg-orange-50 border-t-4 border-orange-500">
<div class="text-orange-700 font-bold mb-2">GMX</div>
Close at oracle adverse side:

- longs fill at `maxPrice`
- shorts fill at `minPrice`

Fee: <b>0.20 %</b> standard / 0.45 % synthetic
</div>

<div class="p-4 rounded bg-purple-50 border-t-4 border-purple-500">
<div class="text-purple-700 font-bold mb-2">dYdX v4</div>
Fillable price spread from oracle (sign $s$):

$$P_{\text{fill}} = P\bigl(1 - s\cdot\text{SMMR}\cdot\text{MMF}\cdot\text{BA}\cdot(1-Q)\bigr)$$

$s=+1$ long-liq · $s=-1$ short-liq

Penalty up to <b>1.5 %</b> of account value → IF
</div>

<div class="p-4 rounded bg-green-50 border-t-4 border-green-600">
<div class="text-green-700 font-bold mb-2">Hyperliquid</div>
On-chain liq price:

$$P_{\text{liq}} = P_{\text{mark}} - s\,\frac{m_{\text{avail}}}{|S|}\,\frac{1}{1 - l/l_{MM}}$$

No fee; MM buffer forfeited if HLP takes the position
</div>

</div>

---

# Oracles: the trust delegation

| Property      | <span class="gmx">GMX</span>            | <span class="dydx">dYdX v4</span>     | <span class="hl">Hyperliquid</span>           |
| ------------- | ------------------------- | --------------------- | ---------------------------- |
| Source        | Chainlink Data Streams    | Validator side-cars   | Weighted CEX median          |
| CEX list      | Provider-internal         | 7+ exchanges          | Binance 3×, OKX 2×, …        |
| Cadence       | Sub-second (pull)         | ~1 s (per block)      | **3 s**                      |
| Aggregation   | Provider-internal         | Validator vote weight | Stake-weighted median        |
| Defense       | Two-phase commit          | Governance + quorum   | OI cap + 1 % resting-order cap |

<v-click>

<div class="mt-4 p-3 rounded bg-blue-50 border-l-4 border-blue-500">
📌 Common theme: <b>avoid thin internal AMMs as oracle</b>. Lesson learned from 2021–2022 spot-AMM exploits.
</div>

</v-click>

---

# Extreme-volatility defenses

|                  | <span class="gmx">GMX</span>           | <span class="dydx">dYdX v4</span>          | <span class="hl">Hyperliquid</span>            |
| ---------------- | -------------------------------------------- | ---------------------- | ---------------------------------------- |
| Circuit breakers | Price-impact 0.5–10 %                        | None explicit          | OI caps + 1 % resting-order cap          |
| Delay auction    | 2-phase commit (~1 block)                    | None                   | 3-s oracle + 30-s liq cooldown            |
| Backstop         | GM pool (`MAX_PNL_FACTOR`)                   | Insurance fund         | HLP vault                                 |
| ADL              | Synthetic markets only                       | After IF exhausted     | After HLP exhausted                       |

---

# Funding rate at a glance

|              | <span class="gmx">GMX</span>            | <span class="dydx">dYdX v4</span>                    | <span class="hl">Hyperliquid</span>           |
| ------------ | ------------------------- | ---------------------------------- | ---------------------------- |
| Type         | Adaptive (OI imbalance)   | Premium + interest                 | Premium + interest           |
| Sample       | per-second continuous     | 1 min                              | **5 s**                      |
| Settle       | on position touch         | hourly                             | hourly (1/8 of 8h rate)      |
| Cap          | 15–300 % APR              | $600\% \cdot (\text{IMF}-\text{MMF})$/8h | 4 % per hour            |

---
layout: section
---

# Part 3
## GMX Deep-Dive

The liquidation flow that backs $1B+ in TVL

---
class: diagram-slide
---

# GMX v2 liquidation flow

```mermaid {scale: 0.85}
flowchart LR
    Monitor[Keeper monitors<br/>+ Chainlink Streams] --> Check{collateral &lt;<br/>0.25–1% notional?}
    Check -- No --> Monitor
    Check -- Yes --> Close[Close at oracle<br/>worst-side price]
    Close --> Fee[Deduct fee<br/>0.20–0.45%]
    Fee --> Split{Fee split}
    Split -- 63% --> LP[GM/GLV LPs<br/>absorb PnL]
    Split -- 37% --> Proto[Protocol +<br/>keeper subsidy]
    LP --> ADLCheck{pendingPnL &gt;<br/>MAX_PNL_FACTOR?}
    ADLCheck -- No --> Done([Complete])
    ADLCheck -- Yes --> ADL[ADL: force-close<br/>profitable peers] --> Done
    ADL --> Done

    classDef oracle fill:#e1f5ff,stroke:#0288d1,stroke-width:2px
    classDef pool fill:#fff4e1,stroke:#f57c00,stroke-width:2px
    classDef danger fill:#ffebee,stroke:#c62828,stroke-width:2px
    class Oracle oracle
    class LP,PoolUpdate pool
    class ADL,Check,ADLCheck danger
```

<div class="caption caption-amber">
🔍 <b>Spotlight:</b> &nbsp; Keeper monitors &nbsp;→&nbsp; commit oracle (block N) &nbsp;→&nbsp; execute (block N+1) &nbsp;→&nbsp; <b>63%</b> fee → GM LPs / <b>37%</b> → protocol &nbsp;→&nbsp; ADL guard
</div>

---

# Worked example — $1M long ETH @ 10×

<div class="grid grid-cols-2 gap-6 mt-2 text-sm">

<div>

**Setup**
- Entry: **$3,000**
- Drop to: **$2,730** (−9 %)
- PnL: **−$90,000**
- Collateral left ≈ $10,000 *(below 1 % floor)*

**Liquidation**
- Keeper triggers liquidation
- Fee: 0.20 % × $1M = **$2,000**
  - $1,260 → GM pool LPs (63 %)
  - $740 → protocol (37 %)
- Pool absorbs: **+$90,000** PnL (in WETH)
- Residual to trader: ≈ **$8,000**

</div>

<div class="flex flex-col justify-center">

<div class="p-5 rounded bg-amber-50 border-l-4 border-amber-500">
<div class="text-base font-bold mb-2">💡 GM pool ≈ short-vol, long-fee</div>
<div class="text-xs leading-relaxed">
Profits on liquidations; loses on sustained directional moves
(capped by <code>MAX_PNL_FACTOR</code> → ADL).
</div>
</div>

</div>

</div>

---

# GMX v1 → v2 evolution

| Aspect       | <span class="text-gray-500">GLP v1</span> | <span class="gmx">GM / GLV v2</span> |
| ------------ | --------------------- | -------------------------------------- |
| Pool         | Single shared basket  | Per-market isolated pairs              |
| Risk         | One pool, all assets  | Contagion contained per market         |
| Oracle       | Chainlink push feeds  | Chainlink **Data Streams** (signed pull) |
| ADL          | Limited               | `MAX_PNL_FACTOR_FOR_ADL` per market    |

<div class="mt-6 p-3 rounded bg-red-50 border-l-4 border-red-500">
<b>Lesson behind the upgrade:</b> Sep 2022 AVAX manipulation on v1 GLP.
</div>

---
layout: section
---

# Part 4
## Lessons from the Battlefield

---

# Real incidents → real design changes

| Date         | Incident                  | Mechanism that failed         | Modern fix                             |
| ------------ | ------------------------- | ----------------------------- | -------------------------------------- |
| Mar 2020     | "Black Thursday"          | Maker 0-bid liquidations      | Backstop bidders, min bid floors       |
| May 2022     | LUNA / UST                | Reflexive collateral spiral   | OI / asset caps                        |
| Sep 2022     | GMX v1 AVAX               | Thin spot + single pool       | Per-market isolation                   |
| Nov 2022     | FTX collapse              | Centralized custody           | Validated non-custodial perp DEX thesis |
| **Mar 2025** | **Hyperliquid JELLY**     | HLP took toxic inventory      | Asset onboarding + per-asset risk budgets |

---

# JELLY squeeze — the modern case

<div class="grid grid-cols-2 gap-8 mt-4">

<div class="p-5 rounded bg-red-50 border-l-4 border-red-500">

**What happened**

- Attacker self-traded a low-cap asset into liquidatable state
- HLP backstop absorbed the position *(as designed)*
- Underlying spot too illiquid for HLP to unwind
- HLP took a multi-million $ loss

</div>

<div class="p-5 rounded bg-blue-50 border-l-4 border-blue-500">

**What it teaches**

Insurance-fund primitives need:
- Asset onboarding due diligence
- Per-asset risk budgets
- Governance kill-switch

> *"Automated backstop" isn't automatic enough for adversarial new listings.*

</div>

</div>

---

# JELLY — quantitative attack model

<div class="grid grid-cols-2 gap-6 text-sm">

<div>

**Microstructure inequality** *(necessary condition)*

$$\begin{cases} l \cdot c \;>\; L_{\text{exit}} & \text{book can't absorb} \\[2pt] l \cdot c \;\lesssim\; V & \text{HLP rich enough to swallow} \end{cases}$$

<div class="mt-4 text-sm">

- For **BTC**: $L_{\text{exit}} \gg V$ → attack impossible
- For a **new listing**: inequality flips → HLP becomes the dump

</div>

</div>

<div>

**Expected extraction** *(dual-account model)*

$$\mathbb{E}[\text{extraction}] \approx \min\!\Bigl(N\tfrac{p_1-p_0}{p_0},\,V\Bigr) - c$$

Plugging JELLY numbers
($N \approx \$70$M, $4\times$ ramp, $V \approx \$240$M, $c \approx \$3.5$M):

$$\approx \min(\$210\text{M},\,\$240\text{M}) - \$3.5\text{M} \approx \boxed{\$206\text{M}}$$

<div class="text-xs opacity-75 mt-2">
Profit-side addresses exit clean USDC <i>before</i> HLP can unwind — the liquidity time-lag is the rent.
</div>

</div>

</div>

---
layout: section
---

# Part 5
## Open Problems & The Road Ahead

---

# Open risks (common to all three)

<div class="grid grid-cols-2 gap-x-8 gap-y-3 mt-4 text-base">

<div>1. <b>Oracle compromise</b> — Chainlink integrity vs validator collusion</div>
<div>2. <b>MEV / front-running</b> of liquidations — reduced but not eliminated</div>
<div>3. <b>ADL fairness</b> — being force-closed in profit is brutal UX</div>
<div>4. <b>Chain halt</b> — sovereign chains pause trading <i>and</i> liquidation</div>
<div>5. <b>LP / HLP impermanent risk</b> — short-vol decays in trends</div>
<div>6. <b>Stablecoin / bridge risk</b> — USDC depeg affects all three</div>
<div>7. <b>Cross-margin correlation</b> — MM ignores off-diagonal $\rho_{ij}$</div>
<div>8. <b>Funding-induced cascade</b> — capped funding drains margin even on flat mark</div>

</div>

---

# Cross-margin: the hidden correlation tax

<div class="grid grid-cols-2 gap-8 text-sm">

<div>

**Today's maintenance check** *(diagonal only)*:

$$\mathrm{MM}_{\text{req}} = \sum_i |S_i|\,P_i\,\mathrm{MMF}_i$$

**But equity variance is** $n\!\times\!n$:

$$\mathrm{Var}(E) = \sum_{i,j} S_iS_j\,\sigma_i\sigma_j\,\rho_{ij}$$

In crashes $\rho_{ij}: 0.5 \to >0.9$ — ten alt-coin longs $\ne$ ten independent risks.

</div>

<div>

**Why no on-chain fix yet?** Cost.

| Approach | Per-account cost |
|---|---|
| Full SPAN matrix $H$ | $O(n^2)$ / block |
| Validator amortised  | $O(n^3)$ |
| <span class="text-emerald-700"><b>Correlation-group haircut</b></span> | <span class="text-emerald-700"><b>$O(n+K)$</b></span> |

<div class="text-xs mt-3 opacity-80">
FTX / CME-SPAN-style: bucket assets into $K \ll n$ classes, assume $\rho=1$ inside, step-wise add-on on dominant net exposure.
</div>

</div>

</div>

---

# Where the design space is going

<div class="text-sm mb-3 opacity-75">Each idea pays a system-engineering price (in <span class="text-rose-600 font-bold">red</span>).</div>

<div class="space-y-3 text-sm">

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
🔐 <b>ZK-proven oracles</b> (Pyth Lazer, zkTLS) &nbsp;<span class="text-rose-600">· cost: 200 ms–2 s prover latency → increases effective oracle staleness on 3 s-cadence chains</span>
</div>

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
🏛 <b>Intent-based liquidation auctions</b> → MEV value back to user &nbsp;<span class="text-rose-600">· cost: window $\tau$ → bad debt $\Delta D \sim |S|\,v\,\tau$; linear in crashes</span>
</div>

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
🤝 <b>Hybrid CLOB + RFQ</b> for liquidated inventory &nbsp;<span class="text-rose-600">· cost: RFQ round-trip + mandatory CLOB fallback path = 2× engineering surface</span>
</div>

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
🧱 <b>Per-asset insurance tranches</b> — no monolithic HLP &nbsp;<span class="text-rose-600">· cost: fragmented capital → thin-asset vaults demand higher yields</span>
</div>

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
✅ <b>Formal verification</b> of margin invariants &nbsp;<span class="text-rose-600">· cost: protocol upgrades require re-proving; months per change</span>
</div>

<div class="p-3 rounded bg-gray-50 border-l-4 border-indigo-500">
⚡ <b>Programmable circuit breakers</b> — per-asset LULD-style bands &nbsp;<span class="text-rose-600">· cost: false-positive freezes legitimate hedging; vol-detector becomes governance risk</span>
</div>

</div>

---
layout: center
class: text-center
---

# Three answers to one question

<div class="grid grid-cols-3 gap-6 mt-8 text-left">

<div class="p-5 rounded-lg bg-orange-50 border-t-4 border-orange-500">
  <div class="text-orange-700 font-bold text-xl mb-2">GMX</div>
  <div class="text-sm">Isolate risk on willing LPs, trust Chainlink.</div>
</div>

<div class="p-5 rounded-lg bg-purple-50 border-t-4 border-purple-500">
  <div class="text-purple-700 font-bold text-xl mb-2">dYdX v4</div>
  <div class="text-sm">Rebuild CEX inside a sovereign chain, transparent IF.</div>
</div>

<div class="p-5 rounded-lg bg-green-50 border-t-4 border-green-600">
  <div class="text-green-700 font-bold text-xl mb-2">Hyperliquid</div>
  <div class="text-sm">Aggressive leverage backed by a community vault.</div>
</div>

</div>

<div class="mt-10 max-w-3xl mx-auto text-base opacity-85 leading-relaxed">
None dominates. The next generation will <b>mix primitives</b> — oracles from one, insurance from another, matching from a third — to <i>break</i> rather than <i>navigate</i> the UX-vs-safety trade-off.
</div>

<div class="mt-12 text-3xl">Questions?</div>

---
layout: end
---

# Thank you

Report · Slides · Script — all in the project repo

Citation map: [refs.md](../refs.md)

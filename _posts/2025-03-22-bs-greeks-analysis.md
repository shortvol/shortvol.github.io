---
layout: post
title: "My First Post"
date: 2025-03-22
---


# Black-Scholes Greeks: Detailed Analysis & Decomposition

**SPX Put Butterfly: +4 / −8 / +4**
Strikes: 5,875 / 5,600 / 5,325 | Expiry: July 18, 2025
Valuation Date: January 6, 2025

---

## 1. Market Data & Model Inputs

### 1.1 Underlying & Rate Assumptions

| Parameter | Value |
|---|---|
| Spot (S) | 5,974.00 |
| Risk-Free Rate (r) | 4.30% |
| Dividend Yield (q) | 1.30% |
| Valuation Date | January 6, 2025 |
| Expiry Date | July 18, 2025 |
| Days to Expiry | 193 |
| T (years) | 0.5288 |
| Contract Multiplier | $100 |

> *Note: The risk-free rate is approximated from the 6-month US Treasury bill yield. The dividend yield is an estimate of the S&P 500 trailing dividend yield. In practice, these can be more precisely extracted from market data using put-call parity on near-ATM options.*

### 1.2 Option Market Data (January 6, 2025)

| Leg | Strike | Price (pts) | Implied Vol | Contract $ |
|---|---|---|---|---|
| Upper Long Put | 5,875 | 174.45 | 15.27% | $17,445 |
| Short Put | 5,600 | 119.10 | 17.74% | $11,910 |
| Lower Long Put | 5,325 | 82.05 | 20.09% | $8,205 |

Implied volatilities are backed out from the market prices using Brent's root-finding method on the BS put pricing formula. The resulting skew is clearly visible: lower strikes carry progressively higher implied volatility, consistent with the well-documented equity volatility skew.

### 1.3 Structure Definition

The structure is a put butterfly scaled by 4, consisting of:

- **+4** Upper Long Puts (K = 5,875)
- **−8** Short Puts (K = 5,600)
- **+4** Lower Long Puts (K = 5,325)

The wing width is 275 points on each side, centered on the 5,600 strike. Maximum payoff at expiry occurs if spot settles exactly at 5,600.

---

## 2. Raw Black-Scholes Greeks (Index Point Basis)

This section presents the raw BS greeks on a per-index-point basis, before applying the $100 SPX contract multiplier. These are the direct outputs of the Black-Scholes formulas.

### 2.1 Core Formulas

All greeks share the common building blocks d₁ and d₂:

```
d₁ = [ln(S/K) + (r − q + σ²/2) · T] / (σ · √T)

d₂ = d₁ − σ · √T

N(x)  = standard normal CDF
N'(x) = standard normal PDF = (1/√(2π)) · exp(−x²/2)
```

| Greek | Formula (European Put) | Measures |
|---|---|---|
| **Delta** | e^(−qT) · [N(d₁) − 1] | Δ price / $1 spot |
| **Gamma** | e^(−qT) · N'(d₁) / (S · σ · √T) | Δ delta / $1 spot |
| **Vega** | S · e^(−qT) · √T · N'(d₁)  [÷100 for per 1 vol-pt] | Δ price / 1 vol-pt |
| **Theta** | See Section 5 for full decomposition | Δ price / 1 day |

### 2.2 Individual Option Greeks (Raw, per index point)

These values represent the change in the option price (in index points) for a 1-unit change in the respective input. Vega is shown per 1 percentage-point change in implied volatility. Theta is per 1 calendar day.

| Leg | K | IV | Delta | Gamma | Vega | Theta |
|---|---|---|---|---|---|---|
| Upper Long Put | 5,875 | 15.27% | −0.3611 | 0.000562 | 16.20 | −0.84 |
| Short Put | 5,600 | 17.74% | −0.2439 | 0.000406 | 13.58 | −0.76 |
| Lower Long Put | 5,325 | 20.09% | −0.1652 | 0.000284 | 10.77 | −0.65 |

### 2.3 Structure Greeks (Raw)

| Leg | Wt | wDelta | wGamma | wVega | wTheta |
|---|---|---|---|---|---|
| Upper Long Put | +4 | −1.4444 | +0.002248 | +64.78 | −3.35 |
| Short Put | −8 | +1.9509 | −0.003245 | −108.63 | +6.06 |
| Lower Long Put | +4 | −0.6608 | +0.001136 | +43.06 | −2.60 |
| **TOTAL** | | **−0.1543** | **+0.000139** | **−0.79** | **+0.10** |

The structure is nearly neutral across all greeks. The small positive theta (+0.10 pts/day) indicates the butterfly earns a modest amount of time decay, consistent with being net short the ATM belly options.

---

## 3. Dollar Greeks (×100 Contract Multiplier)

SPX options have a $100 multiplier, meaning each index point of option price equals $100. All values below represent the actual dollar P&L per contract for a 1-unit move in the relevant input.

### 3.1 Individual Option Dollar Greeks

| Leg | K | Delta ($) | Gamma ($) | Vega ($) | Theta ($) |
|---|---|---|---|---|---|
| | | *per $1 spot* | *per $1 spot* | *per 1 vol-pt* | *per day* |
| Upper Long Put | 5,875 | −$36.11 | +$0.0562 | +$1,620 | −$83.86 |
| Short Put | 5,600 | −$24.39 | +$0.0406 | +$1,358 | −$75.79 |
| Lower Long Put | 5,325 | −$16.52 | +$0.0284 | +$1,077 | −$65.11 |

### 3.2 How to Read These Numbers

Taking the Upper Long Put (K = 5,875) as an example:

**Delta = −$36.11:** If the S&P 500 rises by $1 (from 5,974 to 5,975), the put contract loses approximately $36.11 in value. The negative sign reflects the inverse relationship between puts and the underlying.

**Gamma = +$0.0562:** For each $1 the spot moves, the delta itself changes by $0.0562. Gamma also captures convexity: the P&L from a $1 move due to curvature is ½ × Gamma × (ΔS)² = ½ × 0.0562 × 1 = $0.028.

**Vega = +$1,620:** If implied volatility rises by 1 percentage point (from 15.27% to 16.27%), the contract gains approximately $1,620.

**Theta = −$83.86:** With each passing calendar day, the contract loses approximately $83.86 in time value, all else equal.

### 3.3 Structure Dollar Greeks (+4 / −8 / +4)

| Leg | Wt | wDelta ($) | wGamma ($) | wVega ($) | wTheta ($) |
|---|---|---|---|---|---|
| Upper Long Put | +4 | −$144 | +$0.2248 | +$6,478 | −$335 |
| Short Put | −8 | +$195 | −$0.3245 | −$10,863 | +$606 |
| Lower Long Put | +4 | −$66 | +$0.1136 | +$4,306 | −$260 |
| **TOTAL** | | **−$15.43** | **+$0.014** | **−$79** | **+$10.45** |

### 3.4 Structure P&L Summary (per 1-unit move)

| Driver | Move | P&L ($) |
|---|---|---|
| Spot (Delta) | +$1 | −$15.43 |
| Spot (Gamma) | +$1 (½Γ·1²) | +$0.007 |
| Vol (Vega) | +1 vol-point | −$79 |
| Time (Theta) | 1 calendar day | +$10.45 |

The butterfly collects approximately $10.45 per day in theta. Vega exposure is modest at −$79 per vol-point, and delta is nearly flat at −$15 per dollar of spot movement.

---

## 4. Vega: Formula & the ÷100 Convention

### 4.1 The Vega Formula

Vega measures the sensitivity of the option price to changes in implied volatility. The Black-Scholes vega formula is identical for puts and calls:

```
Vega = S · e^(−qT) · √T · N'(d₁)
```

where N'(d₁) is the standard normal probability density function (PDF):

```
N'(x) = (1 / √(2π)) · exp(−x² / 2)
```

### 4.2 Why Divide by 100?

The raw formula output gives the price change per 1.0 change in σ. Since σ is expressed as a decimal (e.g., 0.1527 for 15.27%), a change of 1.0 means volatility jumping from 15.27% to 115.27% — not a meaningful scenario.

Traders think in percentage points (e.g., vol moving from 15.27% to 16.27%, which is a 0.01 change in σ). To convert:

```
Vega (per 1 vol-point) = Raw Vega / 100
```

Concrete example using the Upper Long Put (K = 5,875):

| Measure | Value (pts) | Value ($×100) |
|---|---|---|
| Raw formula output | 1,620 | $162,000 |
| Per 1 vol-point (÷100) | 16.20 | $1,620 |

So when we say the Upper Long Put has a vega of 16.20, it means: if implied volatility rises by 1 percentage point (15.27% → 16.27%), the option price increases by approximately 16.20 index points, or $1,620 per contract.

This is a linear approximation. For large vol moves, higher-order effects (volga/vomma, the second derivative of price with respect to vol) become relevant.

### 4.3 Vega Across the Strikes

Vega decreases as strikes move further from the money:

| Leg | K | d₁ | N'(d₁) | Vega/1% |
|---|---|---|---|---|
| Upper Long Put | 5,875 | 0.3489 | 0.3754 | 16.20 |
| Short Put | 5,600 | 0.6886 | 0.3147 | 13.58 |
| Lower Long Put | 5,325 | 0.9688 | 0.2496 | 10.77 |

The key driver is N'(d₁), the normal PDF. It peaks when d₁ ≈ 0 (at-the-money) and falls as d₁ moves away from zero. Since all three strikes are below spot (OTM puts), d₁ is positive and increasing as strikes move further OTM, causing N'(d₁) and therefore vega to decline.

---

## 5. Theta: Decomposition & Rate Sensitivity

### 5.1 The Theta Formula (European Put)

Put theta consists of three distinct terms, each with a clear economic interpretation:

```
Term 1 (Time Decay) = −[S · e^(−qT) · N'(d₁) · σ] / (2√T)

Term 2 (Dividend)   = +q · S · e^(−qT) · N(−d₁)

Term 3 (Interest)   = −r · K · e^(−rT) · N(−d₂)

Theta (per year)     = Term 1 + Term 2 + Term 3
Theta (per day)      = Theta (per year) / 365
```

### 5.2 Interpreting Each Term

**Term 1 — Pure Time Decay (always negative):** This is the erosion of optionality as time passes. It is driven by N'(d₁), σ, and critically by 1/√T. The inverse square-root of time means decay accelerates dramatically as expiry approaches: an option with 7 days left decays roughly 4× faster per day than one with 100 days left. This term dominates theta for most options.

**Term 2 — Dividend Effect (positive for puts):** Holding a put delays your exposure to the underlying. Meanwhile, the underlying pays dividends you do not receive. For put holders, this works slightly in your favor because the forward price is lower than spot (due to dividends), supporting the put value. The magnitude is proportional to q (dividend yield).

**Term 3 — Interest Rate Effect (negative for puts):** A put gives the right to sell at K in the future. The present value of receiving K is reduced by the discount factor e^(−rT). As time passes, you are one day closer to receiving K, so discount shrinks, which is a positive effect. However, the formula accounts for the continuous cost of deferring that receipt, which nets negative. Higher rates amplify this cost.

### 5.3 Term-by-Term Decomposition ($ per contract per day)

| Leg | IV | Term 1 (Time Decay) | Term 2 (Dividend) | Term 3 (Interest) | Theta (Total) |
|---|---|---|---|---|---|
| Upper Long Put | 15.27% | −$64.07 | +$7.68 | −$27.47 | −$83.86 |
| Short Put | 17.74% | −$62.41 | +$5.19 | −$18.57 | −$75.79 |
| Lower Long Put | 20.09% | −$56.04 | +$3.51 | −$12.59 | −$65.11 |

### 5.4 Percentage Contribution by Term

| Leg | Term 1 % | Term 2 % | Term 3 % |
|---|---|---|---|
| Upper Long Put (K=5875) | 64.6% | 7.7% | 27.7% |
| Short Put (K=5600) | 72.4% | 6.0% | 21.5% |
| Lower Long Put (K=5325) | 77.7% | 4.9% | 17.5% |

Term 1 (pure time decay) dominates at 65–78% of total theta. The interest rate term (Term 3) contributes 18–28%, which is significant given r = 4.3%. As strikes move further OTM, Term 1's share increases while Term 3's decreases. This is because N(−d₂) shrinks for deep OTM puts — the probability of exercise drops, making the cost of deferring strike receipt less relevant.

### 5.5 Structure Theta Decomposition

| Leg | Wt | wTerm 1 | wTerm 2 | wTerm 3 | wTheta |
|---|---|---|---|---|---|
| Upper Long Put | +4 | −$256.29 | +$30.73 | −$109.88 | −$335.44 |
| Short Put | −8 | +$499.32 | −$41.51 | +$148.53 | +$606.33 |
| Lower Long Put | +4 | −$224.14 | +$14.06 | −$50.37 | −$260.45 |
| **TOTAL** | | **+$18.88** | **+$3.28** | **−$11.72** | **+$10.45** |

The structure's net theta of +$10.45/day breaks down as follows:

**+$18.88/day from pure time decay:** The 8 short belly options decay faster than the 8 long wings (short options are closer to ATM), so the structure is net collecting optionality erosion.

**+$3.28/day from dividends:** A small positive contribution from the net dividend exposure across the structure.

**−$11.72/day from interest rates:** At r = 4.3%, the cost of deferring strike receipt is material. This is the largest drag on the structure's theta, consuming over 60% of the time-decay benefit.

In a lower rate environment (e.g., r = 1%), Term 3 would shrink substantially, and the net theta would be closer to +$20/day. This highlights how the rate regime directly impacts the economics of butterfly positions.

### 5.6 What Drives Theta Most?

Theta is most sensitive to three inputs, in order of importance:

1. **Time to Expiry (T):** The 1/√T term in Term 1 means theta accelerates into expiry. This is the single most important driver.

2. **Implied Volatility (σ):** σ appears directly in the numerator of Term 1. Higher vol means more optionality value to lose per day. A 30% vol option decays roughly 2× faster than a 15% vol option.

3. **Moneyness (via N'(d₁)):** ATM options have the highest theta because N'(d₁) peaks at d₁ = 0. Deep ITM and OTM options have lower theta.

In summary: short-dated, high-vol, ATM options have the most theta — making them the primary candidates for theta harvesting strategies, but also the most exposed to adverse moves in the underlying.

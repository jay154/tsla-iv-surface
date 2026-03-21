# TSLA Implied Volatility Surface
## 5-Minute Presentation Script
### Quant Research Role — Recruiter Presentation

---

---

# SLIDE 1 — Motivation
## "Why Does Volatility Have a Surface?"

---

### Visual
> Single striking image: a smooth, curved 3D IV surface (magma colormap) with a flat gray plane cutting through it.
> Caption: *"Black-Scholes says flat. The market says otherwise."*

---

### Bullet Points (on slide)
- Options embed the market's **forward-looking volatility expectations**
- Black-Scholes assumes **one constant vol** for all strikes and maturities
- In reality, implied volatility varies — forming a **surface, not a plane**
- This project asks: *what does that surface tell us about market fear, regime, and positioning?*

---

### Speaker Notes (~60 seconds)
> "Every option price contains a hidden number — the implied volatility. It's the market's best guess at future uncertainty, baked into the price you pay. Black-Scholes, the foundational pricing model, assumes this number is constant across all strikes and maturities. But if you actually look at the data — as we did across four years of TSLA options — it's clearly not. The volatility forms a surface. It curves. It smiles. And the shape of that surface is rich with information about what the market is pricing in. That's the question this project answers."

---

---

# SLIDE 2 — Method
## "From 2.6 Million Quotes to 638K IVs"

---

### Visual
> Clean pipeline diagram (left to right):
> `Raw CSV (2.6M rows)` → `Liquidity Filter (647K)` → `BS Inversion (638K IVs)` → `IV Surface`
>
> Below: a small code snippet showing the Newton step (4–5 lines max)

---

### Bullet Points (on slide)
- **Dataset:** TSLA daily EOD options, 2019–2022 (Kaggle)
- **Cleaning:** 5-filter liquidity screen → 647K tradeable quotes
  - Bid > $0.50 | Volume > 0 | Spread < 20% | 7 < DTE < 365 | Strike within 40% ATM
- **IV Solver:** Hybrid Newton-Bisection (vectorized over NumPy arrays)
  - Newton-Raphson for speed; bisection fallback when vega < 1e-6
  - Arbitrage bounds check: rejects prices outside `(S − Ke^{−rT}, S)`
- **Output:** 638,197 IV estimates | Average IV: **66.70%**

---

### Speaker Notes (~60 seconds)
> "The pipeline has three stages. First, we load the raw data — 2.6 million option quotes. We apply a five-part liquidity filter: we remove penny options, zero-volume quotes, stale wide-spread quotes, and anything too deep OTM or near expiry where IV breaks down. That leaves 647,000 clean, tradeable quotes. Then, for each one, we invert the Black-Scholes formula numerically — we're asking: what volatility makes the model price match the market price? We use a hybrid Newton-Bisection solver, vectorized over NumPy arrays for efficiency. Newton-Raphson converges fast in well-behaved regions; bisection kicks in as a fallback. The result: 638,000 implied volatilities across the full dataset."

---

---

# SLIDE 3 — The Surface
## "The Market Smiles, Not Flat"

---

### Visual
> **Primary (large):** 3D IV surface vs. gray BS flat plane — the hero chart
> **Secondary (small, bottom row):** Side-by-side regime comparison — Dec 2020 (viridis) vs Dec 2022 (magma)

---

### Bullet Points (on slide)
- The gray plane = Black-Scholes (constant vol assumption)
- The colored surface = what the market actually prices
- **OTM puts elevated** → crash insurance premium (left wing)
- **Short maturities elevated** → near-term uncertainty higher
- **Regime shifts the shape**, not just the level:
  - *Dec 2020:* symmetric — two-way uncertainty around S&P 500 inclusion
  - *Dec 2022:* steep put-skew — defensive, post-crash fear

---

### Speaker Notes (~60 seconds)
> "Here's the core visual result. The gray plane is the Black-Scholes assumption — one flat volatility for everything. The colored surface is reality. It's immediately obvious they don't match. OTM puts on the left carry significantly higher IV — that's the market pricing crash insurance. Near-term options spike upward — higher uncertainty about imminent moves. The surface curves and smiles rather than lying flat. And when you compare two historical snapshots — December 2020, when TSLA was being added to the S&P 500, versus December 2022, after a 65% drawdown — you see that the *shape* of the surface changes with market regime. Not just the level. The shape tells you which direction the market is afraid of."

---

---

# SLIDE 4 — Insights
## "Skew, Term Structure, and a Rare Event"

---

### Visual
> **Top left:** Volatility skew chart (IV vs Strike, downward sloping) — Dec 2022, ~30 DTE
> **Top right:** Term structure chart (ATM IV vs Maturity) — Dec 2022
> **Bottom (full width):** Smile evolution chart — Aug 11 → Aug 28 → Aug 31, 2020

---

### Bullet Points (on slide)
**Skew** (top left):
- IV rises as strike falls — classic *put skew*
- Market charges more for OTM puts = **tail risk / crash protection premium**

**Term Structure** (top right):
- ATM IV declines with maturity — market expects near-term stress to resolve
- Inverted term structure = signal of acute, near-term fear

**Event Study — 2020 Stock Split** (bottom):
- Announced Aug 11 → retail aggressively buys OTM calls
- By Aug 28: smile flips to *right-skew* — calls more expensive than puts
- Aug 31 (post-split): **vol crush** — IV collapses as event resolves

---

### Speaker Notes (~60 seconds)
> "Decomposing the surface gives two key signals. The skew — how IV varies across strikes at fixed maturity — shows the classic put premium. OTM puts are expensive. That's the market paying for tail protection, sometimes called the variance risk premium. The term structure — how ATM IV varies across maturities — tells you the timing dimension: where on the curve IV is elevated signals when the market expects stress. Then there's the event study — the August 2020 stock split. This is the most striking result. Normally put skew dominates. But as the split date approached, retail traders piled into OTM calls so aggressively that the smile flipped — calls became more expensive than puts. A rare reverse skew. And then on August 31st, the day after the split, IV collapsed. A textbook vol crush. The surface captured all of this in real time."

---

---

# SLIDE 5 — Why It Matters
## "What the Surface Tells You That Black-Scholes Can't"

---

### Visual
> Clean summary graphic — two columns:
> | Black-Scholes Assumes | Market Reality |
> |---|---|
> | Constant volatility | Volatility surface |
> | No fat tails | Put skew = tail risk premium |
> | Static model | Dynamic, regime-sensitive |
> | One number | A landscape of information |

---

### Bullet Points (on slide)
**Practical applications of the IV surface:**
- **Options trading:** accurate pricing requires fitting to the surface, not a single vol
- **Risk management:** skew steepness is a real-time crash risk barometer
- **Macro signal:** term structure inversion flags acute near-term fear (like VIX term structure)
- **Event detection:** surface distortions reveal positioning and market stress before they fully materialize

**Why Black-Scholes still matters:**
- The framework is correct — *the assumption is wrong*
- IV is defined relative to BS — the surface quantifies exactly *how* and *where* BS breaks

**Extensions this work motivates:**
- Local volatility (Dupire) — fits any smile exactly
- Stochastic volatility (Heston, SABR) — models smile dynamics
- Jump-diffusion — accounts for the short-maturity spike

---

### Speaker Notes (~60 seconds)
> "So why does this matter beyond a pricing exercise? The IV surface is a live signal. The slope of the skew tells you how much the market is paying for crash protection right now — it's a real-time fear gauge. The term structure tells you *when* that fear is concentrated. And when the surface distorts around events — like we saw with the stock split — it reveals positioning and stress before it fully shows up in prices. Black-Scholes isn't wrong as a framework — it's wrong as an assumption. IV is defined *through* Black-Scholes, which is why the surface is so useful: it precisely maps where and how the model breaks. Extensions like Heston, SABR, or Dupire local vol are all attempts to model that surface dynamically. This project builds the empirical foundation that motivates all of them."

---

---

## Delivery Notes

| Slide | Time | Key Message |
|-------|------|-------------|
| 1 — Motivation | ~60s | BS assumes flat; market prices a surface |
| 2 — Method | ~60s | 2.6M quotes → 638K IVs via Newton-Bisection |
| 3 — The Surface | ~60s | Smiles, not flat; regime changes shape not just level |
| 4 — Insights | ~60s | Skew = crash fear; term structure = timing; split = rare reverse skew |
| 5 — Why It Matters | ~60s | Surface = live risk signal; motivates Heston/SABR/local vol |

**Total: 5 minutes**

---

## Recruiter Talking Points (if asked to go deeper)

- *"How did you handle the IV solver?"* → Newton-Raphson with bisection fallback; arbitrage bounds pre-filter; tol=1e-8
- *"Why use mid-price?"* → Eliminates bid-ask bounce; standard industry practice for IV computation
- *"What's the risk-free rate?"* → 3.9% — approximate 2019–2022 US Treasury average; sensitivity is low for short-dated options
- *"What would you build next?"* → Fit a Heston model to the surface; compute model vs. market surface residuals; build a vol regime classifier
- *"Why TSLA?"* → High liquidity, high vol, rich event history (split, S&P inclusion, earnings) — maximally informative for surface analysis

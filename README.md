# The Viscosity Ledger

An interactive damping-curve explorer for mountain bike forks. Set a baseline, build up to two
comparison setups, and see what oil choice and temperature do to a fork that nobody touched.

Single-file, zero-dependency HTML. Deploy by renaming to `index.html` and enabling GitHub Pages.

**Live:** https://postmillennium-mtb.github.io/the-viscosity-ledger/

---

## Why this exists

Manufacturer dyno charts show you what a clicker does. They don't show you what a chairlift does.

Two 10wt fork oils can be identical on the bottle, identical at 40 °C, and nearly **2× apart at
0 °C** — because they have different viscosity indices. Almost nobody checks VI when buying oil.
This tool exists to make that difference visible.

---

## What it does

- **Three comparison setups** (baseline + two), each with its own colour and line style, plotted together
- **Fork-cap dials** for LSC / HSC / LSR / HSR — drag to rotate, or use the arrows
- **13 damper fluids and 3 bath oils** with published cSt and VI figures
- **Ambient temperature** (−10 to 105 °F) plus heat soak, combining into actual oil temperature
- **Damping circuit controls** the clickers can't reach: free bleed, compression crossover,
  independent rebound crossover, cavitation ceiling
- **Negative tokens** down to −1.5 (Noken-style high-volume air cap), flattening the spring curve
- **Simulate 5 runs** — progressive heat build across a day, drawn as a fan of curves
- **3D surface** — compression force over shaft speed × oil temperature, with a cavitation plane
- **Conversation panel** — plain-prose comparison of the active setups, with an
  "explain it like I'm 15" version underneath
- **Under the Hood panel** — every formula, with confidence tiering

---

## The math

### Damping curve

Piecewise: linear below the crossover, power curve above it.

```
v ≤ v_knee :   F(v) = k_low · v
v > v_knee :   F(v) = F_knee + k_high · (v − v_knee)^n

where  F_knee = k_low · v_knee
```

`n` sets the character above the knee — `n < 1` digressive, `n = 1` linear, `n > 1` progressive.
This build runs **n = 0.82** compression, **0.85** rebound. Not currently exposed as a control.

### Coefficients

```
k_low  = (clicker term) × bleedF × μ^0.90
k_high = (tune term)    × μ^0.28

bleedF = 1 / (0.45 + (bleed/10) × 1.15)
μ      = ν(oil, T) / ν_ref          ν_ref = Motorex 10 at 20 °C

comp: clicker = 0.55 + (LSC/22) × 1.95 ,  tune = 0.6 + (HSC/6) × 1.4
reb : clicker = 0.50 + (LSR/19) × 1.60 ,  tune = 0.6 + (HSR/6) × 1.4
```

The two viscosity exponents are the load-bearing assumption. Low-speed flow is bleed-orifice
dominated and close to laminar, so force tracks viscosity nearly linearly (0.90). High-speed flow
through shim ports is closer to turbulent and much less viscosity-sensitive (0.28). That gap is why
heat guts low-speed support while barely touching big-hit control.

### Viscosity vs. temperature — ASTM D341 (Walther)

Solved from each oil's published 40 °C and 100 °C points:

```
log₁₀( log₁₀( ν + 0.7 ) ) = A − B · log₁₀( T )        T in kelvin

B = [ LL(ν₄₀) − LL(ν₁₀₀) ] / [ log₁₀(373.15) − log₁₀(313.15) ]
A = LL(ν₄₀) + B · log₁₀(313.15)
```

### Oil temperature

```
T_oil   = T_ambient + soak
soak(n) = 40 · ( 1 − e^(−n/2) )        n = run number
```

Each run adds less heat than the one before, approaching an equilibrium the damper can shed.

### Spring, bottom-out, friction

```
F_spring(x) = (psi/100) · (x / 170)^prog · 4000
prog        = max( 0.95 , 1.6 + tokens × 0.35 )

F_hbo(x)    = 1800 · ((x − 145)/25)^1.5 · speedMult · μ^0.4     x > 145 mm only
speedMult   = 0.5 slow / 1.0 med / 1.8 fast

F_stiction  = 8 + 26 · (ν_bath / 32)^0.45
```

Negative tokens push `prog` toward linear. The 0.95 floor keeps the curve physical.

### Cavitation

A flat ceiling at **1950 N**. Above it the model isn't describing anything real. It marks where a
fork-sized damper tends to run out of oil supply and go harsh — not a calculated limit.

---

## Confidence tiering

| Tier | What |
|---|---|
| **Verified** | Walther / ASTM D341 is the industry-standard viscosity–temperature relation. Oil cSt and VI figures are manufacturer-published. The low-speed / crossover / high-speed structure follows established damper theory. |
| **Reasoned** | Viscosity exponents 0.90 and 0.28. Digressive `n` of 0.82 / 0.85. Heat-soak time constant. Direction defensible, exact values not measured. |
| **Picked** | All absolute force magnitudes — the 4000 N spring scale, 1800 N HBO scale, 1950 N cavitation ceiling, bleed and stiction coefficients. Tuned to look plausible, not fitted to any dyno. |

**Read the shapes and the percentages, not the newtons.** The relationships — how a curve bends, how
far two oils diverge when cold, how much low speed fades relative to high speed — are the part worth
trusting. Absolute forces are illustrative.

No dyno data from any manufacturer was used. Nothing here has been validated against a real damper.

---

## Known gaps

- Seal and bushing friction is folded into one bath-oil term rather than modelled
- No negative spring or top-out behaviour
- Rebound and compression share a single bleed value
- Shaft-speed distribution across real terrain isn't represented, so the chart weights the whole
  velocity range equally even though a rider spends most of their time in a narrow band of it
- `n` (shim stack character) is hardcoded digressive and not adjustable

## Corrections made during development

An earlier build tied the rebound crossover to compression at a fixed **0.87** ratio. That number had
no source. There is no published rebound-to-compression knee ratio, because the two are set by
physically separate components — rebound has its own crossover gap and shim stack. The rod-area
argument actually points the other way: because the rod occupies part of the bore, a rebound stroke
displaces less oil per mm of travel, so reaching shim-crack flow takes a *higher* shaft velocity.
Rebound crossover is now an independent control defaulting to linked at 1.00.

---

## Data sources

Oil viscosity figures are collated from published suspension-fluid comparison tables and are
manufacturer-claimed rather than independently measured. The Motorex SuperGliss / Fox 20wt Gold
pairing rests on a single enthusiast teardown and is the weakest figure in the set.

Damper theory background: Shim ReStackor (restackor.com). Negative token concept: MRP Noken.

---

## Licence

Post Millennium Renaissance · postmillenniumrenaissance.com

# 02 — Survival Function, Hazard, and Cumulative Hazard

## 1. The big picture: one story, three views

Imagine a group of light bulbs switched on at the same time. We watch until they burn out.

| View | Question it answers | Name |
|---|---|---|
| **Survival** | "What fraction are **still working** at time *t*?" | **S(t)**, survival function |
| **Hazard** | "Among bulbs still working at *t*, how fast are they **failing right now**?" | **h(t)**, hazard function |
| **Cumulative hazard** | "How much **total failure-risk** has built up by time *t*?" | **H(t)**, cumulative hazard |

They are **three ways of describing the same underlying thing**. If you know one, you can work out the others. Each view is useful for a different purpose.

**Car analogy 🚗**

- **S(t)** = the fuel gauge's "what's left" (how many are still going).
- **h(t)** = the **speedometer**: how fast risk is being added *at this instant*.
- **H(t)** = the **odometer**: total distance (risk) accumulated so far. It never goes down.

---

## 2. Notation (just two symbols to learn)

- **T** = the (random) time until the event happens for a person.
- **t** = a particular time we are asking about (e.g., 5 years).

---

## 3. The Survival Function S(t)

### Definition

> **S(t) = P(T > t)**: the probability that the event has **not yet happened** by time *t*.

In words: *"the chance of surviving **beyond** time t."*

### Properties (these are always true)

- **S(0) = 1**: at the start, everyone is still event-free (100%).
- **S(t) never increases**: as time passes it can only stay flat or drop. (Once the event has happened, it can't un-happen.)
- **S(t) → 0** for large *t* if everyone eventually has the event (not always, e.g., some people are never "cured of" or never "hit by" the event within the study window).
- Values are always between **0 and 1**.

### "Survival rate" in everyday language

When someone says *"the 5-year survival rate is 80%"*, they mean:

> **S(5 years) = 0.80**: 80% are still event-free at 5 years.

Despite the word "rate," this is really a **proportion / probability**, not a "per unit time" rate. The true "rate" idea belongs to the **hazard** (next section). Keeping these two apart avoids lots of confusion.

### The survival curve

Plotting S(t) against t gives a **step-down curve** that starts at 1.0 and drops each time someone has the event:

```
S(t)
1.0 |‾‾‾‾‾|
    |     |___
0.8 |         |____
    |              |
0.5 |              |____          <- median survival time is where
    |                   |____        the curve crosses 0.5
0.2 |                        |__
    +---------------------------- t (time)
```

### Useful summaries read off the curve

- **Survival at a time:** e.g., S(12) = "12-month survival."
- **Median survival time:** the time where S(t) first drops to **0.5 or below**. Half the group has had the event by then. (Preferred over the *mean*, because censoring makes the mean hard to compute and the data is skewed.)

---

## 3b. Related: the CDF F(t) 

**F(t) = 1 − S(t) = P(T ≤ t)**: the probability the event **has** happened by time *t*. It is the "mirror image" of survival. In some fields (like customer churn) people plot **F(t)** ("fraction who have churned") instead of S(t).

---

## 4. The Hazard Function h(t)

### Plain-English definition

> **Hazard h(t)** = the **instantaneous rate** at which the event happens at time *t*, **given that the person has survived up to time t**.

Key phrase : *"given you're still around."* The hazard only looks at people who are **still at risk** at that moment.

### Intuition by example

- **Human mortality:** The hazard of dying is fairly high for newborns, low for kids and young adults, and then **rises steadily** with age. Even though *fewer* old people are still alive, the ones who are alive have a **higher** per-moment risk.
- **Machine:** A new machine might fail early from defects (high hazard at first), then run reliably (low hazard), then wear out (high hazard again).

### Slightly more formal

For a tiny time step Δ:

```
h(t) ≈ P(event in [t, t+Δ) | still event-free at t) / Δ
```

### Hazard is a **rate**, not a probability

- It can be **bigger than 1** (e.g., "2 events per year" is perfectly fine). Probabilities cannot exceed 1, but rates can.
- Its unit is "**events per unit time**" (per day, per month, per year).
- It is **not** the same as S(t). A high hazard makes S(t) drop quickly, but the two are different things.

### Common hazard shapes

| Shape | Looks like | Real-life example |
|---|---|---|
| **Constant** | flat line | Random failures, such as radioactive decay (the "memoryless" case) |
| **Increasing** | rises over time | Wear-out: aging, mechanical fatigue |
| **Decreasing** | falls over time | "Infant mortality": risk is highest early, e.g., risk after surgery, early customer churn |
| **Bathtub** | high → low → high | Human life; electronics (early defects, stable period, wear-out) |

```
 h(t)
  |\                              /
  | \                            /     <- bathtub
  |  \__________________________/
  +------------------------------ t
```

---

## 5. The Cumulative Hazard Function H(t)

### Definition

> **H(t)** = the **running total of the hazard** from time 0 up to time *t*.

In calculus terms it is the area under the hazard curve:

```
H(t) = ∫₀ᵗ h(u) du
```

### Intuition

- Hazard = how fast risk is added **now**. Cumulative hazard = **total risk added so far**.
- **H(0) = 0**, and **H(t) never decreases** (it only goes up, like an odometer).
- H(t) is **not a probability**; it can be bigger than 1.

### Reading it

If H(t) = 0.5, roughly speaking you've accumulated "half an expected event's worth" of risk. If H(t) = 2, you've accumulated "two expected events' worth" of risk.

### Why bother with it?

- It is **much easier to estimate** from data than the hazard itself (the raw hazard is jumpy and noisy).
- Its **slope** tells you about the hazard:
  - Straight line → **constant** hazard
  - Curving upward → **increasing** hazard
  - Flattening → **decreasing** hazard

---

## 6. How the three are connected (the key formulas)

| Relationship | Formula | In words |
|---|---|---|
| Hazard → Cumulative hazard | **H(t) = ∫₀ᵗ h(u) du** | Add up the hazard |
| Cumulative hazard → Survival | **S(t) = exp( −H(t) )** | More accumulated risk → lower survival |
| Survival → Cumulative hazard | **H(t) = −ln S(t)** | Same thing, rearranged |
| Survival → Hazard | **h(t) = − d/dt ln S(t)** | How steeply (log) survival is dropping |

The most important one to remember:

> ### **S(t) = e^(−H(t))**

Check that it makes sense:

- t = 0 → H = 0 → S = e⁰ = **1** (everyone alive)
- H grows large → S = e^(−large) → near **0** (almost nobody left)

### Constant-hazard special case (a handy sanity check)

If hazard is constant, **h(t) = λ**, then:

- H(t) = λ·t
- S(t) = e^(−λt) (the exponential distribution)
- Median survival = ln(2) / λ ≈ 0.693 / λ

Example: λ = 0.1 per month → median ≈ 6.9 months, and S(12) = e^(−1.2) ≈ 0.30.

---

## 7. Putting it all together in one table

| | Survival S(t) | Hazard h(t) | Cumulative hazard H(t) |
|---|---|---|---|
| **Asks** | Still going at *t*? | Risk right now? | Risk piled up so far? |
| **Starts at** | 1 | (depends) | 0 |
| **Trend** | Only goes down | Any shape | Only goes up |
| **Range** | 0 to 1 | 0 to ∞ | 0 to ∞ |
| **Is a probability?** | Yes | **No** (a rate) | **No** |
| **Unit** | none (proportion) | events per time | none |
| **Best used for** | Communicating results ("80% at 5 yrs") | Understanding *when* risk is high | Estimating and checking models |

---

## 8. Estimating these from real, censored data (worked example)

In practice we don't know S(t); we **estimate** it from data. With censoring, the standard tool is the **Kaplan–Meier (KM) estimator** for S(t) and the **Nelson–Aalen estimator** for H(t).

### The idea in one sentence

> At each time an event occurs, work out the fraction of people **who were still at risk** that had the event; survival is the **running product** of "fraction who survived this step."

Censored people are handled naturally: they count as "at risk" **until** the moment they leave, then quietly drop out of the pool without being counted as events.

### Data (10 people, time in months)

| Person | Time | Event (1 = event, 0 = censored) |
|---|---|---|
| A | 2 | 1 |
| B | 3 | 1 |
| C | 4 | 0 |
| D | 5 | 1 |
| E | 5 | 1 |
| F | 6 | 0 |
| G | 7 | 1 |
| H | 8 | 0 |
| I | 9 | 1 |
| J | 10 | 0 |

### Vocabulary for the table

- **n = number at risk** just before time *t* (still being observed, event-free).
- **d = number of events** at time *t*.
- **Step hazard** = d / n (the chance of the event *this step*, among those at risk).
- **KM survival:** S(t) = previous S × (1 − d/n)
- **Nelson–Aalen cumulative hazard:** H(t) = previous H + d/n

### Step-by-step

| Time t | n at risk | d events | d/n | S(t) (KM) | H(t) (Nelson–Aalen) |
|---|---|---|---|---|---|
| 0 | 10 | 0 | — | 1.000 | 0.000 |
| 2 | 10 | 1 | 0.100 | 1 × 0.900 = **0.900** | 0.100 |
| 3 | 9 | 1 | 0.111 | 0.900 × 0.889 = **0.800** | 0.211 |
| 4 | 8 | 0 (C censored) | 0 | 0.800 | 0.211 |
| 5 | 7 | 2 | 0.286 | 0.800 × 0.714 = **0.571** | 0.497 |
| 6 | 5 | 0 (F censored) | 0 | 0.571 | 0.497 |
| 7 | 4 | 1 | 0.250 | 0.571 × 0.750 = **0.429** | 0.747 |
| 8 | 3 | 0 (H censored) | 0 | 0.429 | 0.747 |
| 9 | 2 | 1 | 0.500 | 0.429 × 0.500 = **0.214** | 1.247 |
| 10 | 1 | 0 (J censored) | 0 | 0.214 | 1.247 |

### What to notice

1. **Curve only moves at event times.** Censoring times (4, 6, 8, 10) don't make the curve drop, but they **shrink the at-risk group (n)**, so later drops are bigger. Look at t = 9: only 2 people remain at risk, so a single event drops survival by half.
2. **Median survival = 7 months.** S(t) first falls to ≤ 0.5 at t = 7 (0.429).
3. **Survival at 5 months ≈ 57%**; S(5) = 0.571.
4. **H(t) only goes up** and S(t) only goes down. 
5. **S ≈ e^(−H) is only approximate here.** For example at t = 9, e^(−1.247) ≈ 0.29, while KM gives 0.214. They match exactly for smooth, continuous-time theory, but with a *tiny* sample and big discrete steps the two estimators differ a bit. With more data, they agree closely.

---

## 9. Common beginner mix-ups

| Mistake | Correct idea |
|---|---|
| "Hazard is the probability of the event." | Hazard is a **rate**, and it can exceed 1. |
| "Survival rate = hazard." | Survival is a proportion still event-free. Hazard is the instantaneous risk among those still at risk. |
| "If hazard is high, survival is high." | The opposite: higher hazard → survival **falls faster**. |
| "Hazard must increase because fewer people remain." | Hazard is measured **per person still at risk**, so it can go up, down, or stay flat. |
| "Cumulative hazard is a probability." | It's a running total of risk. It can go above 1. |
| "Censored people count as events." | They count as at risk until they leave, and are **never** counted as an event. |
| "The curve drops at censoring times." | Only at event times. Censoring just shrinks the at-risk group. |

---

## 10. Summary / cheat sheet

- **S(t) = P(T > t):** probability of being event-free beyond *t*. Starts at 1, only goes down. "Survival rate" in everyday speech = S at a chosen time.
- **h(t):** instantaneous event **rate** at *t* among those still at risk. A rate (can exceed 1), shape can be flat, rising, falling, or bathtub.
- **H(t) = ∫ h(u) du:** accumulated hazard. Starts at 0, only goes up. Not a probability.
- **Master link: S(t) = e^(−H(t))**, equivalently H(t) = −ln S(t).
- **Median survival** = first time S(t) ≤ 0.5.
- **Kaplan–Meier** estimates S(t): multiply (1 − d/n) over event times. **Nelson–Aalen** estimates H(t): add d/n over event times.
- **Censoring** shrinks the at-risk count *n* but never counts as an event.

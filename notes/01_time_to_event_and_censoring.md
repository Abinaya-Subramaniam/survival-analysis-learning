# 01 — Time-to-Event Data and Censoring

## 1. The big idea in one paragraph

Survival analysis answers the question: **"How long until something happens?"**

That "something" is called the **event**. It does not have to be death, despite the name. The catch is that, in real data, we often **don't get to see the event happen for everyone** (people leave the study, or the study ends first). That missing information is called **censoring**, and handling it correctly is the whole reason survival analysis exists.

---

## 2. Time-to-event data

Every observation has two ingredients:

1. **A start point** (time zero): when we begin the clock.
2. **An event**: the thing we are waiting for.

The **time-to-event** is the time between them.

| Field | Start (time zero) | Event | Time-to-event |
|---|---|---|---|
| Medicine | Diagnosis or start of treatment | Death, relapse, recovery | Days/months until that happens |
| Customers | Date they sign up | Cancels subscription (churn) | Months until cancellation |
| Engineering | Machine is switched on | Machine breaks | Hours until failure |
| HR | Employee joins | Employee quits | Years until resignation |
| Banking | Loan is issued | Borrower defaults | Months until default |

### Three things you must define clearly

Before any analysis, answer these:

- **What is time zero?** (Everyone's clock must start at a comparable moment.)
- **What counts as the event?** (Be precise: "death from any cause" vs "death from cancer" are different events.)
- **What is the time unit?** (Days, weeks, months, years.)

### Why not just use regular tools?

Time-to-event data has two properties that break ordinary methods:

- **Time is always positive and usually skewed.** Most people have short times, a few have very long ones.
- **Some times are incomplete (censored).** This is the big one.

---

## 3. Censoring: the key concept

### Plain-English definition

> **Censoring** means we only know that a person's event time is *at least* some value, but we don't know the exact time because we stopped watching before the event happened.

A censored observation is **not** a missing value and **not** useless. It tells us something real: *"this person survived at least this long."*

### A story to make it concrete

A clinic follows 5 patients for a 12-month study to see how long until a relapse.

```
Patient A  |=======X                 relapse at month 7
Patient B  |===========O             moved away at month 10   (censored)
Patient C  |============|            study ended at month 12, no relapse  (censored)
Patient D  |===X                     relapse at month 4
Patient E  |=====================... joined late, still fine at end         (censored)
           0    2    4    6    8   10   12  (months)

X = event happened     O or | = observation stopped, no event seen (censored)
```

- **A and D**: we saw the event. We know their exact times (7 and 4 months).
- **B, C, E**: we did **not** see the event. For B we only know "no relapse for *at least* 10 months."

If we threw B, C, E away, we'd lose real information and our answer would be **biased** (it would look like relapses happen sooner than they really do).
If we treated their stop time as the event time, we'd be **wrong in the other direction**.

> **Rule of thumb:** a censored time means *"the event time is **greater than** this number."*

---

## 4. Types of censoring

### 4.1 Right censoring (by far the most common)

We know when observation **started**, but it **stopped before** the event happened. The true event time lies to the **right** (later) on the timeline.

Common causes:

- **Study ends** before the event (administrative censoring).
- **Loss to follow-up**: person moves, stops responding, drops out.
- **Withdrawal for another reason**: e.g., a customer account is closed because of a company merger, not because they chose to leave.

```
start ----------------- last seen ~~~~~~~~~~~~~~> event (unknown, somewhere later)
```


### 4.2 Left censoring

The event **already happened before** we started watching. We know it happened, but not exactly when.

*Example:* You test people for an infection at the start of a study and find some already infected. You don't know *when* they got infected, only that it was **before** the first test.

```
event (unknown, earlier) <~~~~~~ first observation ---------->
```

### 4.3 Interval censoring

We only know the event happened **between two check-ups**, not exactly when.

*Example:* A patient's scan is clear at month 6 and shows a tumor at month 12. The tumor appeared sometime **between** months 6 and 12.

```
clear at 6 |~~~~~~~~ event somewhere in here ~~~~~~~~| found at 12
```

### Quick comparison

| Type | What we know | Example |
|---|---|---|
| **Right** | Event is *after* time *t* | Study ended, no event yet |
| **Left** | Event is *before* time *t* | Already infected at first test |
| **Interval** | Event is *between* *t₁* and *t₂* | Clear at 6 months, positive at 12 |

> In this learning series we focus on **right censoring**. Keep the other two in mind so you recognize them later.

---

## 5. How right-censored data is stored

Each row needs **two** columns, not one:

| patient | time (months) | event |
|---|---|---|
| A | 7 | 1 |
| B | 10 | 0 |
| C | 12 | 0 |
| D | 4 | 1 |
| E | 9 | 0 |

- **`time`**: how long we observed the person (until the event *or* until we lost them).
- **`event`** (sometimes called `status`): a flag.
  - `1` = the event **was observed**
  - `0` = **censored** (event not observed)

This pair `(time, event)` is the basic input to nearly every survival method and library (including `lifelines` in Python).

> **Common beginner mistake:** reading `event = 0` as "nothing happened to this person ever." It only means "we did not **see** it happen while watching."

---

## 6. Why we can't just ignore or delete censored data

Imagine 10 customers, and we want the average time until cancellation. The study lasts 12 months.

| Customer | Months observed | Cancelled? |
|---|---|---|
| 1 | 3 | Yes |
| 2 | 5 | Yes |
| 3 | 6 | Yes |
| 4 | 12 | No (still active) |
| 5 | 12 | No |
| 6 | 12 | No |
| 7 | 12 | No |
| 8 | 12 | No |
| 9 | 12 | No |
| 10 | 12 | No |

**Wrong approach 1: delete the censored rows.**
Left with only customers 1–3, the average is (3+5+6)/3 ≈ **4.7 months**. This makes it look like customers leave very fast, when 7 out of 10 are still loyal. Badly **biased downward**.

**Wrong approach 2: treat "12" as the cancellation time.**
This pretends those 7 customers cancelled at month 12, but they haven't. It **underestimates** how long they really stay.

**Right approach: use survival methods** (Kaplan–Meier, Cox regression, etc.), which use the partial information from censored people ("stayed at least 12 months") correctly.

---

## 7. The key assumption: non-informative censoring

Survival methods assume that **censoring is unrelated to the person's chance of having the event.** In other words, people who drop out are not systematically more (or less) likely to have the event than those who stay.

| Situation | Okay? |
|---|---|
| Study ends on a fixed date for everyone | Fine |
| Patient moves city for a new job |  Probably fine |
| Patients drop out because they feel **too sick** to attend |  **Problem**: dropout is linked to the outcome |
| Customers stop logging in because they are about to churn, and we mark them "lost" |  **Problem** |

When censoring is linked to the outcome, it is called **informative censoring**, and it can make results misleading. Always ask: *"Why did this person leave the study?"*

---

## 8. A few related terms we will meet soon

| Term | Meaning |
|---|---|
| **Event** | The outcome we wait for (death, churn, failure…). |
| **Survival time** / **duration** | Time from start until event or censoring. |
| **Censored** | Event not observed during follow-up. |
| **Follow-up time** | How long someone was observed. |
| **Survival function S(t)** | Probability of **not** having the event by time *t*. *(Covered in the next note.)* |
| **Hazard** | The instantaneous risk of the event at time *t*, given you've survived until *t*. *(Later note.)* |

---

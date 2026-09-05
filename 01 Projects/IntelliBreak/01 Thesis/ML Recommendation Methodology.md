---
type: thesis
project: IntelliBreak
status: proposed
tags:
  - thesis
  - machine-learning
  - methodology
  - ml-target
---

# ML Recommendation Methodology

## Current Status

**PROPOSED — METHODOLOGY DEFINED, IMPLEMENTATION NOT YET STARTED**

The Work Pattern Decline Prediction methodology is the current proposed direction for IntelliBreak's machine learning component.

However, implementation must wait until:

1. Training data availability is verified.
2. Sufficient historical examples are confirmed.
3. Class balance is checked.
4. The target-generation pipeline is verified.
5. Temporal validation is implemented.
6. The final ML model is selected.

---

## Current Proposed Methodology: Version 4 — Work Pattern Decline Prediction

### Definition

Predict whether the user's task completion rate will show a significant decline in the subsequent 20–40 minute window compared with the user's personal historical baseline.

### Ground Truth

Future task completion rate < 50% of the user's personal baseline.

### Task Completion Rate

Number of tasks with status = DONE and ended_at inside the target window / target-window duration in hours.

### Personal Baseline

Median task completion rate across the user's historical completed work sessions.

### Important Clarification

This target predicts an **OBSERVABLE CHANGE IN WORK BEHAVIOR**.

It does **NOT** claim to measure:

- fatigue
- stress
- mental health
- physiological recovery
- medical conditions
- subjective tiredness

**IntelliBreak is not a medical or diagnostic system.**

---

## Temporal Structure

### Feature Window
[t-20 minutes, t]

### Gap
[t, t+20 minutes]

### Target Window
[t+20 minutes, t+40 minutes]

### Temporal Separation — Critical for Reducing Target Leakage

The model must never use future information from the target window when calculating features.

This temporal separation is specifically intended to reduce target leakage and ensure the model learns predictive patterns from historical observations rather than circular logic.

---

## Currently Proposed Features

Subject to verification against the actual IntelliBreak database:

- recent task completion rate
- recent task switching/activity
- time since last break
- current/continuous work duration
- recent break patterns
- time-of-day indicators
- historical personal baseline
- other objectively available work-pattern indicators

### Important Caveats

- Do **NOT** claim that cognitive load is available unless the actual IntelliBreak database contains a defensible field for it.
- Do **NOT** use "natural completion point" unless the database has an objective way to determine it.

---

## ML Model Output

### What the Model Predicts

Probability that task completion rate will significantly decline in the next 20–40 minutes.

**Example:** P(decline) = 0.82

### What This Means

"The model estimates an 82% probability of the defined work-pattern decline event occurring in the future target window."

### What This Does NOT Mean

- "Employee productivity is 82%."
- The employee is fatigued.
- The employee needs to take a break.
- The employee should work faster or take fewer breaks.

---

## Recommendation Engine

### Separation of Concerns

Keep the ML model and recommendation engine separate:

**ML:**
Recent work behavior → probability of future work-pattern decline

**Recommendation Engine:**
ML probability + current observable behavior + break/work context → wellness-oriented recommendation

### Example Recommendations

**High probability:**
"Your recent work pattern suggests you may benefit from a short break."

**Moderate probability:**
"Your work pattern is beginning to change. Consider scheduling a break soon."

**Low probability:**
"Your current work pattern appears stable. Continue as needed."

### Language Guidelines

**IMPORTANT:** Do not use language claiming the system knows the employee is fatigued.

**Avoid:**
- "You are fatigued."
- "You need a break."
- "Your performance is declining."

**Prefer:**
- "Your recent work pattern suggests you may benefit from a break."
- "Your work pattern is changing. A break might help."

---

## Wellness Purpose

### Primary Objective

The objective of IntelliBreak is **NOT** to maximize employee task throughput.

Task completion rate is used as an **observable indicator of work-pattern change**, not as a productivity metric.

### Ultimate Purpose

The ultimate purpose is to support:

- sustainable work habits
- appropriate break timing
- focus management
- healthy work/break intervals
- prevention of excessive continuous work
- employee self-care

### Recommendation Principle

Recommendations must encourage sustainable behavior rather than tell employees to work faster or take fewer breaks.

---

## Connection to Adviser KPI Framework

### Strongest Conceptual Connection

**Call Center KPI:** TPH (Tickets Processed / Productive Hours)

**IntelliBreak Adaptation:** Tasks Completed / Productive Work Hours

### Important Note

This is a conceptual adaptation and is **NOT** being used as the primary ML optimization target. It provides context for understanding the relationship between work patterns and the observable metrics IntelliBreak can measure.

### Other KPI Mappings (Contextual)

- AHT (Average Handle Time) → Average Task Duration
- FTR (First Time Resolution) → Task completion without rework/reopening
- FRT (First Response Time) → Task creation/assignment → first work activity
- Customer Satisfaction → Currently has no direct IntelliBreak equivalent

---

## Why Work Pattern Decline Over Recovery Need?

### Previous Consideration: Recovery Need Prediction

Recovery Need was considered but not selected as the final target because:

**Recovery need is an internal state that cannot be directly measured from current IntelliBreak data.**

### The Problem with Recovery Need as a Target

Taking a break does **NOT** necessarily prove that an employee needed recovery because a break may occur because:

- it was scheduled
- it was lunch
- the employee chose to rest
- a task was completed
- an external interruption occurred

### Solution: Observable Work-Pattern Change

The current methodology predicts an **observable work-pattern change** instead of an assumed internal recovery state. This approach:

- Uses only objectively measurable data from IntelliBreak
- Avoids making claims about internal employee states
- Reduces the risk of false positives and model misalignment with reality
- Aligns with the wellness purpose of preventing excessive continuous work

---

## Implementation Next Steps

1. Inspect actual IntelliBreak historical data.
2. Determine the number of usable training examples.
3. Generate candidate temporal training samples without leakage.
4. Analyze class balance.
5. Establish a simple baseline model.
6. Evaluate using temporal validation.
7. Approve the final model before production integration.
8. Implement the recommendation engine only after validating the model.

---

## Historical Versions (Preserved for Reference)

### Version 1 — Previous ML Attempt

Status: **Superseded**

The first approach attempted to implement machine learning for work-break recommendations but was not completed. Details of this version are maintained in project history for reference.

### Version 2 — Task Throughput

Status: **Superseded**

Focused on maximizing task throughput as the primary optimization target. This approach was reconsidered because:

- It conflicted with the wellness-first purpose of IntelliBreak
- Optimizing for task throughput could encourage unsustainable work patterns
- It did not align with the goal of supporting healthy work habits

### Version 3 — Recovery Need Prediction

Status: **Superseded**

Attempted to predict recovery need directly from work patterns. This approach was reconsidered because:

- Recovery need is an internal state not directly observable in IntelliBreak data
- Break behavior is influenced by many factors beyond recovery (scheduling, lunch, task completion)
- The methodology conflated observable behavior with internal states
- A more grounded, observable target would reduce model misalignment

**See "Why Work Pattern Decline Over Recovery Need?" section above for detailed reasoning.**

### Version 4 — Work Pattern Decline Prediction

Status: **CURRENT PROPOSED APPROACH**

The methodology defined in this document. This is the current direction, pending verification of training data and implementation readiness.

---

## Related Notes

- [[Current Thesis Requirements]]
- [[Focus Rate - Research Notes]]
- [[Flutter Folder Structure]]
- [[Authentication]]
- [[Break System]]

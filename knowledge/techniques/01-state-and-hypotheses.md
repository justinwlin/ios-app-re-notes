# Locate the state owner and test competing explanations

## When to use this

Use this when a visible symptom has several plausible causes: a countdown appears
to expire, a feed filter behaves inconsistently, an image does not render, or a
screen stops accepting touches. The goal is to distinguish the data that causes
the behavior from the UI that merely displays it.

## Mechanism

A UI usually reflects state owned elsewhere. Several consumers can read that state:
the renderer, navigation, persistence, completion rules, and analytics. A getter
hook changes only callers of that getter. Direct field reads or a second cached
collection can continue to see the old state. Conversely, changing shared state
without understanding its consumers can break invariants those consumers expect.

The CatTok refinement demonstrates this distinction: modifying one returned list
did not keep related feed state consistent. The later change operated on the
underlying related collections at a common access point. Our general deduction is
to list consumers and invariants before choosing where to transform data.

## Procedure

1. Write down the exact sequence that reproduces the symptom. Include starting
   state, inputs, and the observation that counts as failure.
2. Choose two or more explanations that predict different observations. For a
   frozen screen: busy main thread, lock wait, disabled interaction, or an overlay.
3. Inspect state before modifying it. Use a thread sample/backtrace for execution
   hypotheses, a view hierarchy for interaction hypotheses, and field/collection
   observations for model hypotheses.
4. Find the state writer and the decision consumer. Record the field, selector,
   method token, address, or request identity connecting them.
5. Make a narrow experimental change that separates the explanations. Predict its
   result first; a change that makes the symptom disappear does not by itself prove
   why it disappeared.
6. Replace the experimental workaround with a change at the established cause.
   Check the original reproduction plus nearby interactions.

## Concrete diagnostic example: unresponsive navigation

Suppose a swipe transition leaves a screen that ignores taps. A paused stack at
the ordinary run loop does not establish an infinite loop. Check whether the view
tree contains a transition/dimming container with interaction disabled. Then
inspect the navigation gesture's delegate and the places that enable/disable it.
A lifecycle callback that disables a gesture during the transition can leave a
different failure than a delegate that rejects the gesture before it begins.

Changing a delegate to always allow or always reject can distinguish those cases,
but either is initially an experiment. Record which result rules out which theory.
Do not keep a global gesture override when only one transition needs correction.

## Concrete diagnostic example: a countdown

```text
UI-only change:
  remaining -= 1
  render("180")
  if remaining <= 0: lose()

State change:
  remaining -= 0
  render(remaining)
  if remaining <= 0: lose()
```

This pseudocode illustrates the distinction; it is not recovered app source.
For the second form to achieve unlimited play, the relevant loss rule must read
that same positive value. Prove that connection instead of assuming a shared name
means a shared variable. Separately identify bomb, inactivity, event, or server
limits; changing one counter does not change every time-related rule.

## Boundary-case thinking

The asset-filename example shows why a tiny reproduction is valuable: a particular
input can collide with a sentinel value in an internal transformation. Test the
transform's boundaries, not just more examples of the original symptom. Check zero,
empty, absent, duplicate, and already-completed states when they have different
meaning in the code. Treat any AI-generated explanation as a hypothesis until it
matches the observed bytes and a discriminating test.

## Evidence to retain

Keep the reproduction, competing explanations, observation that distinguishes
them, state-owner/consumer connection, and final behavior check. Record remaining
unknowns separately from a successful experiment.

Sources: [CatTok state refinement, 25:54](https://www.youtube.com/watch?v=YW3jL2gI9IE&t=1554),
[Disneyland diagnosis, 1:03](https://www.youtube.com/watch?v=SpHl5_0n3Ps&t=63),
[asset-name example](https://www.youtube.com/watch?v=Dt4o7hz0B60).
The procedure and pseudocode above are our synthesis for future investigations.

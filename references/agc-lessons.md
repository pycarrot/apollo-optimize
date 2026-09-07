# What the Apollo Guidance Computer teaches

Use this file when deciding how far to push, or when a trade-off between speed,
correctness, and size needs a tie-breaker. These are engineering lessons, not history
trivia; each maps to a rule in the ladder.

## The budget was the design

Fixed memory: 36,864 words of core rope for programs, 2,048 words erasable. The team did
not write the software and then shrink it; the size limit was the first requirement, and
every routine was allocated words before it was coded. Lesson: **set the budget before
the code** (`budgets.md`). A change without a budget is a change whose cost nobody chose.

## Density beat speed when memory was the constraint

The AGC ran most guidance math in an *interpreted* language that was ~10× slower than
native instructions but far more compact. Compactness won because memory was the
scarcer resource. Lesson: **optimize the constraining resource, not the one that is
easiest to measure**. On a free-tier database the constraint is row writes, not latency;
on an edge worker it is CPU milliseconds and bundle bytes, not memory.

## Priority scheduling and load shedding

The executive ran jobs by priority and, when overloaded, dropped the lowest-priority
work rather than failing. During Apollo 11's descent, spurious radar interrupts consumed
~15% of cycles; the 1201/1202 alarms fired, the executive shed display updates, and the
guidance loop kept running. Lesson: **know which work is essential and make the rest
droppable**: defer analytics, batch bookkeeping, and never let a non-critical path block a
critical one (`backend.md` "respond, then work").

## Restart protection

Every job stored enough state that a full restart could resume it. Restarts were a
designed, tested path, not an emergency. Lesson: **tests and invalidation stories are the
restart tables**. Caching, batching, and deferral all create states that can go stale or
be lost; each must have a written way to recover, and a test that exercises it.

## Every bit inspected

Programs were reviewed word by word; the rope memory was literally woven by hand and
could not be patched after manufacture. Lesson: **there is no "fast enough"**, only
"nothing left within budget that I can find". Small savings on hot paths are worth
taking, and the review is where they are found.

## Simplicity as a safety property

Complex features were removed when they could not be verified within the memory and
schedule. The lunar landing did not have a full autopilot; it had a simple, verifiable
loop and a human on the throttle. Lesson: **rung 1 of the ladder, "don't do it", is
also the most reliable code you will ever ship**.

## What to do when you are tempted to stop

Ask the three AGC questions:

1. What is this costing, in the resource that is actually scarce here?
2. If that resource were 10% smaller tomorrow, what would I remove first?
3. Have I written down what I left in and why, so the next pass starts from here?

If you can answer the second question, do it now. If you cannot, you have not looked
hard enough.

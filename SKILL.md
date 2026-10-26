---
name: ios-touch
description: Make websites and apps feel like iOS-style touch animation, using original spring and rubber-band code. Use by default for any website, app, sheet, list, card, or control that needs animated touch, dragging, flicking, or overscroll. Covers spring physics, interruptibility, velocity handoff, momentum projection, rubber-band overscroll, tuning presets, reduced motion, and verification.
---

# iOS Touch

Contents: Status and ownership note, The core idea, When to use,
Rule 1: Springs, Rule 2: Interruptibility, Rule 3: Velocity handoff,
Rule 4: Rubber-band overscroll, Tuning presets, Performance and
accessibility, Verification checklist, Copyright, trademark, and
patent boundary, Default rule.

## Status and ownership note

This skill is an original summary, written in our own words, with our own
code examples. It is not an Apple product, it is not affiliated with or
endorsed by Apple, and it contains no Apple source code, no decompiled
code, no Apple brand, logo, copy, assets, sounds, or system artwork.

Learned from: a public Instagram reel by Justin Sun
(@justinbuilds.mov, "Why does iOS feel so nice to use", 2026-10-24),
public SwiftUI spring documentation, and general physics. The reel
describes, at a high level, two ideas: rubber-band scrolling and
spring-based motion. This skill turns those ideas into original code.

## The core idea, in one paragraph

Cheap motion eases from A to B on a fixed curve over a fixed time.
iOS-style motion has mass. A value accelerates, overshoots a little,
and settles, because it is driven by a spring, not by a timer. A user
can grab it at any moment, reverse it, and flick it, and it never
jumps, because the next animation starts from the live on-screen
value and carries the live velocity forward.

## When to use, and when not to

Use by default for: websites, web apps, mobile apps, sheets, drawers,
tabs, lists, cards, toggles, sliders, drag-and-drop, pull-to-refresh,
and any control a finger or cursor touches.

Do not use for: video rendering timelines (use a frame function of
time instead), a one-off fade that carries no gesture, or any place
where motion is the only way a change is communicated. Motion is
feedback, never the message alone.

## Rule 1: Springs, not easing curves, for gesture-driven motion

The physics is damped harmonic motion:

- acceleration = (-stiffness * (position - target) - damping * velocity) / mass
- velocity += acceleration * dt
- position += velocity * dt

Run it per frame. On the web use requestAnimationFrame. On native
use the platform spring. Never use a CSS transition or a keyframe
animation for anything the user can grab mid-flight, because those
restart from a fixed path and cannot smoothly reverse.

Clamp dt to at most about 1/30 of a second, so a slow frame does not
explode the spring. Stop when both the distance to target and the
velocity are below a small threshold, then snap exactly to target.

Original web integrator, safe to copy into a project:

```js
let x = 0, v = 0, target = 0;

function springStep(dt, stiffness, damping, mass) {
  dt = Math.min(dt, 1 / 30);
  const a = (-stiffness * (x - target) - damping * v) / mass;
  v += a * dt;
  x += v * dt;
  if (Math.abs(x - target) < 0.1 && Math.abs(v) < 0.1) {
    x = target;
    v = 0;
    return true; // settled
  }
  return false;
}
```

On SwiftUI, use the public API, not a hand-rolled curve:

```swift
withAnimation(.spring(response: 0.4, dampingFraction: 0.8)) {
    isExpanded.toggle()
}

// Full physics form, when velocity matters:
withAnimation(.interpolatingSpring(mass: 1, stiffness: 170, damping: 26, initialVelocity: 0)) {
    isExpanded.toggle()
}
```

## Rule 2: Interruptibility is the whole feel

- Never lock input while an animation runs.
- On interrupt, read the live presentation value, not the logical
  target value. Starting from the target causes a visible jump.
- Carry velocity through the retarget. Replacing one animation with
  another at a reversal, with velocity reset to zero, creates a
  hard stop, like hitting a wall.
- For 2D motion, run one spring per axis. X and Y can have different
  velocities, and a single spring on distance will desync them.
- Test the grab-and-reverse case first, before testing the clean
  start-to-finish case. If the reverse feels wrong, the skill has
  not been applied, whatever the clean case looks like.

## Rule 3: Velocity handoff and momentum projection

While dragging, track position and time, and estimate release
velocity from the last few pointer samples, not from the whole drag.
On release:

1. Hand that velocity to the spring as its initial velocity.
2. For a flick, project forward first: a fast flick should travel
   further than a slow drag to the same point, then settle at the
   nearest valid stop (a detent, a page, a list edge).
3. Use velocity sign, not just position, to decide commit versus
   cancel on a sheet or swipe. A fast flick down dismisses even
   from a short drag; a slow drag back up cancels even from far down.

Keep the projection simple and clamped. Never let a projected
target run past a real bound. Bounds are handled by Rule 4.

## Rule 4: Rubber-band overscroll, with a ceiling

When content is pulled past an edge, do not stop hard and do not
follow the finger 1:1. Apply progressive resistance, with a ceiling,
so the content stretches but cannot be dragged away.

Hyperbolic form, as described in the reel, with x = finger pull
distance, d = screen or container height, c = resistance constant:

```
offset = (1 - 1 / (c * x / d + 1)) * d
```

Properties to check: offset is 0 when x is 0, offset grows more
slowly as x grows, and offset can never pass d, no matter how far
the finger pulls. That ceiling is what makes it feel controlled
instead of loose.

A common equivalent form used in UIKit-style write-ups is:

```
offset = (x * d * c) / (d + c * x)
```

Pick one form, use it consistently, and tune c by feel. A larger c
means less resistance at first and a faster approach to the ceiling.
On release past an edge, spring back to the edge with the same
spring as Rule 1, starting from the stretched position and velocity.

## Tuning presets

Start here, then tune by feel on a real phone. Faster is usually
better than slower. Over 0.5 seconds feels sluggish unless it is a
signature moment.

SwiftUI presets, iOS 17 and later style:
- Smooth, default transitions: duration 0.5, bounce 0.0
- Snappy, micro-interactions and toggles: duration 0.3, bounce 0.15
- Bouncy, playful attention moments: duration 0.5, bounce 0.3
- Interactive, gesture tracking: response 0.15, damping 0.86

Response and damping form:
- Button press: response 0.2, dampingFraction 0.7
- Sheet presentation: response 0.5, dampingFraction 0.9
- Playful pop: response 0.3, dampingFraction 0.6
- DampingFraction 1.0 means no overshoot. Below 0.6 means visibly
  bouncy. Above 1.0 means overdamped, no bounce, slower settle.

Web starting points, mass 1:
- Snappy: stiffness 300, damping 30
- Smooth: stiffness 170, damping 26
- Bouncy: stiffness 200, damping 15

Never set a duration that fights the spring. A spring computes its
own settle time from stiffness, damping, and mass. If a spec gives
both a fixed duration and spring physics, pick the spring and note
the duration as approximate.

## Performance and accessibility

- Animate transform and opacity only. Never animate width, height,
  top, left, or any property that forces layout per frame.
- Keep the spring loop cheap: one integrator per animated value,
  no allocations per frame, no work in the loop that belongs in
  setup.
- Respect Reduce Motion. Replace spring motion with a short
  crossfade, a much shorter animation (about 0.1 to 0.15 seconds,
  no bounce), or no motion. On the web check
  prefers-reduced-motion. On SwiftUI check accessibilityReduceMotion.
- Supplement motion with another signal where it matters, such as
  a label change or haptic feedback. Never use motion alone to
  communicate a state change a user must not miss.

## Verification checklist, before calling it done

1. Drag it 1:1. The element follows the finger with no lag that
   reads as rubbery, and no lead that reads as detached.
2. Grab it mid-flight and reverse it. No jump, no hard stop.
3. Flick it fast and slow. Fast travels further and lands on the
   correct stop. Slow stays near the drag point.
4. Pull past every edge. Resistance grows, the stretch caps, and
   release springs cleanly back to the edge.
5. Play it at phone width and at 60 fps, with no layout jank.
6. Turn on Reduce Motion. The change still reads, without the
   spring.
7. If any check fails, fix the spring, the velocity handoff, or
   the bound. Do not fix it by adding a fade, a delay, or a
   different easing curve on top.

## Copyright, trademark, and patent boundary

This section is part of the skill, not a footnote. Follow it on
every build that uses this skill.

- Copyright protects expression, not a function or a general idea.
  Writing new code that achieves the same motion is not the same
  as copying someone else's code. This skill therefore uses only
  original wording and original code examples. Never paste Apple
  code, decompiled code, a video transcript, or another creator's
  script into a project, a skill, or a repo.
- Trademark: do not use Apple branding, logos, product names as
  your product name, or copy that implies Apple made, endorsed, or
  approved your work. "iOS-style" in this skill is descriptive only,
  for compatibility with a motion feel, and every public copy must
  carry a not-affiliated disclaimer like the one at the top of this
  file. Do not name a repo, app, or page as if it were an Apple
  product.
  Note, owner decision 2026-10-26: Darren chose "iOS Touch" as the
  product name despite the guidance above. That choice increases
  trademark risk, and it does not change the disclaimer requirement
  or the ban on implying Apple endorsement.
- Patent: this skill makes no claim that any Apple patent is
  expired, invalid, or cleared. Some overscroll techniques were
  covered by Apple patents in the past, and patent history for
  rubber-banding includes challenges and changing rulings. That
  history is exactly why a public post must stay at the level of
  general spring physics and original code, must not clone an
  exact iOS user interface, and must not copy Apple implementation
  details. This skill is not legal advice. For a commercial
  product, get real counsel before shipping a feature that copies
  a patented interaction step by step.
- If a rights holder objects to any element, remove that element
  first and talk after. Do not repost the same element under a
  new name to get around an objection.
- Lowest-risk order for publishing: (1) internal skill and
  owner-only demo, (2) private repo, (3) public repo only after
  the file has been read for copied code, copied copy, Apple
  branding, and exact iOS UI cloning, and only with a
  not-affiliated disclaimer in the README.

## Default rule

When a brief asks for a website, an app, or anything that needs
animated touch, load this skill first, alongside the design and
coding skills for that build. Do not substitute a CSS transition,
a fixed easing curve, or a generic fade-and-slide for the spring
integrator in this skill, unless the brief explicitly names that
substitution and the reason for it.

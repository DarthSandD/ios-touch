# Spring Touch

Make websites and apps feel like iOS-style touch animation, using
original spring and rubber-band code.

Spring Touch is a skill, not an app. Load `SKILL.md` first when you
build a website, an app, or anything that needs animated touch:
dragging, flicking, sheets, lists, cards, toggles, or overscroll.

## What it covers

- Spring physics instead of fixed easing curves, for gesture-driven motion
- Interruptibility: start from the live on-screen value, carry velocity
  through a retarget, never jump on grab-and-reverse
- Velocity handoff and momentum projection on flick release
- Rubber-band overscroll with progressive resistance and a ceiling
- SwiftUI and web tuning presets, Reduce Motion, and a verification checklist

## Use it

Copy `SKILL.md` into your agent skills folder, or read it directly
before building. The web integrator in the skill is a small,
self-contained spring loop. The SwiftUI examples use only the public
spring API.

## Not affiliated with Apple

Spring Touch is an independent, original summary. It is not an Apple
product, and it is not affiliated with or endorsed by Apple. It
contains no Apple source code, no decompiled code, no Apple brand,
logo, copy, assets, or system artwork. "iOS-style" in this repo is
descriptive only, for a motion feel.

## Copyright and patent note

This repo contains only original wording and original code examples.
Copyright protects expression, not a general function or idea, so
writing new code for the same motion is not copying someone else's
code. Do not paste Apple code, decompiled code, or another creator's
script into this repo.

This repo makes no claim that any Apple patent is expired, invalid,
or cleared. Some overscroll techniques were covered by Apple patents
in the past, and their history includes challenges and changing
rulings. This skill stays at the level of general spring physics and
original code, and it does not clone an exact iOS user interface.
This note is not legal advice. For a commercial product, get real
counsel before shipping a feature that copies a patented interaction
step by step.

If a rights holder objects to any element here, that element will be
removed first and discussed after.

## License

No license file is included yet, so the contents are not yet offered
under an open license. Add a license before reusing this skill in a
project that needs one.

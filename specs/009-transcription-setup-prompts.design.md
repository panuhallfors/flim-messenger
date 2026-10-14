---
decision: 009
slug: transcription-setup-prompts
created: 2026-10-14
---

# 009 — Transcription setup prompts (design)

Companion to `009-transcription-setup-prompts.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Decision 005 puts the method choice in Settings and finishes a method’s setup before it is used. Decision 004 shows a plain-language **not set up** status on a voice message when no method is in use. Try again on that status cannot succeed until a method exists.

This decision steers the user into setup: a soft step at the end of post-login onboarding, a stronger one-shot call-to-action on the first unset voice message, and — amending 004 — a setup link on every not-set-up bubble instead of Try again.

## Product design

### When onboarding asks

After the existing post-login sequence (identity confirmation when required, app lock when mandatory, analytics when required, notification permissions when required), and only when **no** transcription method is in use, show one more full-screen step.

If a method is already in use (for example from an earlier session on this device), skip this step entirely.

The step is soft:

1. Short explanation that voice messages can get a title, summary, and transcript once a method is chosen.
2. **Primary** — continue into decision 005’s selector (and from there that method’s setup). Finishing a method’s setup dismisses onboarding as usual and leaves that method in use.
3. **Secondary** — not now. Marks the onboarding ask as done without choosing a method. The rest of the app opens.

Leaving mid-setup from the primary path without finishing a method leaves no method in use. The onboarding ask counts as done once the user has either finished a method from this step or chosen not now — so the step does not loop on every launch. Returning later is through Settings (005) or the voice-message setup link below.

### Not set up on the bubble (amends 004)

Whenever the brief place shows decision 004’s **not set up** status, the action control is a **setup link** that opens decision 005’s selector (and setup). It is not Try again. Layout weight stays the quiet failure block from 004 (status + link). No room notice; the player stays.

### First voice message — stronger CTA once

The first time this account on this device **shows** a voice-message bubble while no method is in use — whether the user sent it or someone else did — that same brief place uses a **stronger** setup call-to-action (still opening 005’s selector). After that first prompt has been shown once, later unset voice messages keep only the quiet not-set-up status and setup link above.

“First time” is per Matrix account on this device. It is independent of whether the user skipped the onboarding ask. If they already finished a method before any voice message appears, this stronger prompt never shows.

### What the prompts open

The onboarding primary action, the stronger first-voice-message CTA, and the quiet not-set-up setup link all open the same product surface as Settings → Voice message transcription: decision 005’s selector first, then that method’s setup. They do not invent a parallel chooser.

### Relationship to 004 and 005

- 005 remains the owner of methods, selector, and setup. This decision only adds when and how the user is steered there.
- 004’s not-set-up status remains; its action becomes setup (this decision). Other failure statuses keep Try again as 004 specifies.
- A successful method in use restores 004’s normal brief behaviour.

## Platform notes

- Both: the onboarding step is the last conditional step in the existing post-login onboarding flow. Presentation matches the other soft prompts there (full-screen dialog style). Strings are Flim catalog keys (decision 002).
- Both: the “already asked / already shown first CTA” flags live with the same account/app settings pattern the other onboarding flags use — durable across relaunches for that account on that device.
- iOS: plug into the existing onboarding state machine after notification permissions.
- Android: same step order and outcomes when that onboarding sequence is replayed.

## Open issues

_(none)_

## History

- 2026-10-14: first write
- 2026-10-14: not-set-up always setup link (amends 004); stronger CTA only once on first voice message
- 2026-10-14: platforms = ios, android

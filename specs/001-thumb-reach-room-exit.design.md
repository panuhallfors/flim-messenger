---
decision: 001
slug: thumb-reach-room-exit
created: 2026-09-10
---

# 001 — Thumb-reach exit from a room (design)

Companion to `001-thumb-reach-room-exit.md`. **Replay-canonical** with that decision: later platforms (e.g. Android after iOS) implement this product design, not a parallel one.

## Context

Upstream opens a room with a conventional navigation push and returns via the top-leading back control — far from the resting thumb. This decision makes open/close a **container transform** (Material name; iOS: shared element / matched geometry): the list row grows into the room surface, and dismiss returns it to its place. A **bottom strip** is the thumb-reachable, discoverable control that drives that dismiss. Horizontal swipe-to-back is out of scope.

## Product design

### Open and close — container transform

- Tapping a conversation list row expands that row into the full room surface. Closing reverses the same transition onto the row’s place in the list.
- Animate **bounds** (width, height, corner radius), not a uniform scale of the whole element. Uniform scale stretches text and squashes the avatar; a growing clipped rectangle with content re-laid out inside is the intended look.
- **Crossfade** list-row content ↔ room content. Do not morph preview text into bubbles. Exception: **avatar and display name** travel as a true shared element from the row into the room header so one continuous object ties the transition together.
- Timing: old content fades out in roughly the first ~30% of progress; room content fades in from ~50% → 100%.
- **One progress scalar** `0 → 1` owns the expansion. Strip activation (and any later dismiss affordances) drive the same value. Use a spring-based, interruptible animation — not a fixed-duration timeline — so velocity can carry from gesture into settle, and a mid-gesture release can snap open or closed.
- Durations (when not gesture-driven): open ~220 ms, close ~160 ms. Close may be faster; the user already knows the destination. Re-opening the same room within a few seconds may skip the morph entirely.
- **Prefetch:** begin preparing the room surface on touch-down (not touch-up) so content or a skeleton is ready before the morph runs. Swapping a skeleton for real content mid-transition is worse than showing a skeleton through the end of the animation.
- **Source row missing or moved** while the room is open:
  - Row visible at a new index → close to the new on-screen rect (order change is informative).
  - Row scrolled off-screen → do **not** animate to an off-screen point; shrink toward the screen edge in that row’s direction and fade out.
  - Opened from notification / deep link with no source row → use a fallback (rise from the strip region, or centre fade). Separate transition; do not fake a row morph.
- **Reduced motion:** drop geometric morph; crossfade only.
- **Performance:** if the morph cannot hold frame budget on the weakest supported device (long room + large list), fall back to a simple crossfade for that device class. A stuttering morph is worse than an instant cut.

### Bottom strip — visible return

- The room sits above the conversation list in z-order. Along the bottom of the room surface, a full-width strip is a window back toward that list — an inverted bottom sheet: the background peeks from below, and dismiss collapses the room back onto its row via the container transform (not a lateral slide).
- The strip is the **default discoverable** exit. It must look like chrome for “return to the list,” not like another chat to open.
- **Peek content:** blur / dim heavily, or show a neutral surface. Never a sharp top list row with avatar + name that reads as “open this conversation.”
- **Keyboard:** when the composer is focused and the keyboard is up, hide the strip and let the room extend to the bottom. Return almost always happens while reading, not while typing; a mis-hit that dismisses mid-compose is worse than a wasted back tap.
- **System gesture inset:** leave a few millimetres above the home indicator / Android gesture nav region, and keep the strip’s drag-sensitive area above that inset so a downward system gesture is never captured by the strip. Tap anywhere in the strip still works inside the inset.
- Closing animation should feel like lifting the room off the stack and returning it to its row. Prefer shrinking and fading along the transform over sliding the full screen height away (full-height slides feel slow on a high-frequency action).

### Strip affordance — making “this closes the room” legible

A bare peek or handle is not self-explanatory; the strip must be read correctly on first sight, without coaching.

- **Required content:** a **downward caret** (collapse glyph) plus a **destination label** naming the list exactly as its own screen title does (e.g. “Conversations”), centred as one pill-shaped unit. The glyph alone is ambiguous — the word is what removes guessing, and it is also the accessibility label.
- **Why a downward caret does not fight the morph:** the glyph describes the outcome — put this layer away, the list is underneath — not the geometric path. Keep the *dominant read* of the close animation as receding in depth (shrink + dim), with translation toward the row secondary, so “down / underneath” stays consistent even when the target row sits high on screen.
- **Degradation, in order:** icon + label → icon only (largest accessibility text sizes, or very narrow widths) → never label only. Grabber-pill-plus-label is the accepted fallback if prototyping shows the caret reads as “scroll down” instead of “close”.
- **Whole strip is the hit target,** not just the pill: full width, at least the platform minimum touch height above the safe-area inset. Icon and label alignment is cosmetic; reach never depends on it.
- **Value when idle:** when useful, the strip may also surface why returning matters (e.g. “3 new in other chats”) so the vertical space earns its keep and doubles as an inbox cue without visiting the top bar. Do not reflow or animate that text while the user is reading — resolve it when the strip appears, then hold it steady.
- **First run:** one-time introduction only (brief label emphasis on the first room open, then settle). No repeating coach marks, no blocking tooltip.
- **Separation from the composer:** the strip must be visually distinct from the composer row above it (elevation edge or surface change), so the two most-tapped bottom elements never read as one control.
- **Post-keyboard guard:** dismissing the keyboard reveals the strip underneath a thumb that is often already descending. The strip must ignore activation for a short settle window after it appears, so “close the keyboard” cannot become “close the room”. This is the single most likely mis-tap in the design.
- **Haptics:** a light confirmation on close commit; nothing on reveal.

### Activation — tap and drag

Both activations drive the one progress scalar; there is no second close animation.

- **Tap** anywhere in the strip runs the close as a spring settle to `0`. This is the discoverable, guaranteed path and the accessibility path.
- **Drag upward** from the strip scrubs progress directly: the room shrinks toward its row as the finger travels, so the user sees the destination before committing. Release decides by position and velocity combined — a short flick with enough speed completes, a slow drag past roughly the halfway point completes, anything else springs back open. Velocity transfers from finger to spring; the gesture is interruptible at any moment, including re-grabbing mid-settle.
- **Direction:** only upward drag scrubs. A downward drag on the strip does nothing (it belongs to the system), and the strip never becomes a handle for pulling the list up over the room.
- **Rubber-banding:** dragging beyond fully-closed resists rather than overshooting into the list, so an over-long drag cannot skip past the list into some other state.
- Drag is a shortcut for people who discover it, never a requirement. Everything reachable by drag is reachable by tap.

### Closing while the keyboard is open

The strip is hidden during composition, so the room needs an exit that does not reintroduce a control next to `send`. The answer is **staged dismissal**, driven by the same downward gesture the platform already uses to put the keyboard away.

- **Stage one** — a downward drag on the timeline (or system back) dismisses the keyboard and reveals the strip. **Stage two** — tap or drag the strip to close the room. Two deliberate actions, in the order the platform itself uses: Android’s back button has always dismissed the keyboard before leaving the screen, so this is a model people already hold.
- **The detent is sticky:** momentum from stage one does not carry into stage two. A fast flick to dismiss the keyboard stops at the keyboard-dismissed state and never continues into closing the room. Continuing requires a fresh touch. Together with the strip’s settle window on appearance, this makes “I dismissed the keyboard and lost my conversation” impossible by momentum alone.
- **A draft is never lost by leaving.** Closing with text in the composer preserves the draft for that room, and the row the room collapses back onto shows a draft indicator — so the user watches the unsent text land somewhere visible instead of wondering whether it survived. This is what makes staged dismissal acceptable rather than merely defensible: the cost of leaving mid-sentence is visibly zero.
- **No close control adjacent to the composer while composing.** During composition the highest-value pixels belong to the draft, the attachment control, and `send`; adding a dismiss target there trades a rare need for a frequent, expensive mis-tap.
- Screen-reader and external-keyboard users get the same two stages as discrete, labelled actions (dismiss keyboard, then return to conversations) — never a gesture-only path.

### Layout consequences inside the room

- Inset the timeline’s bottom by the strip height so no message is permanently hidden behind it; scroll-to-latest and typing indicators sit **above** the strip, never overlapping it.
- Bottom-anchored transient UI (context menus, reaction pickers, snackbars) must not cover the strip while it is the only visible exit.
- Contrast must come from the surface itself, not from blur alone — verify in dark mode and over media-heavy backgrounds.
- Where the list is already on screen (tablet / regular-width split layout), the room is not a collapsed row and the strip has no destination: hide it and keep conventional navigation.

### Consistency with system back

- OS-level back (Android predictive back, iOS interactive edge-swipe if enabled by the platform, VoiceOver escape gesture, hardware or external keyboard back) drives the **same progress scalar** and the same container transform. No separate dismiss animation, and no state where the system back and the strip disagree about where “back” goes. This is not the fork adding a horizontal gesture — it is refusing to break the platform’s own.

### Accessibility and non-thumb use

- VoiceOver / TalkBack and desk / two-hand use still need an explicit “back to conversations” action. The strip owns that role: it is a real button, labelled with its destination, exposed at a predictable place in the reading order (adjacent to the composer, not buried after the timeline), and it responds to the platform’s standard escape gesture.
- Dynamic Type / font scaling grows the strip rather than truncating or clipping it; the touch target never shrinks below the platform minimum.
- Do not leave return discoverable only by gesture, and do not rely on colour or blur alone to distinguish the strip.
- Because the strip is a labelled, always-visible control while reading, the top-leading back chevron is no longer load-bearing for this exit and may be removed on compact layouts. It stays where the list is already visible (see below).

### Handedness

- This exit does not need a handedness setting: the strip spans the full width. Do not mirror the entire UI, bubble sides, or menu logic for handedness. Any future handedness preference (floating controls elsewhere) is a separate decision.

## Alternatives rejected

- **Full-screen horizontal swipe-to-back (one or both directions)** — earlier candidate; deferred/rejected here. Conflicts with reply-swipe and horizontal child scrollers; animation direction fights “back”; occupies gestures other apps use for message info or next chat. Not part of this design.
- **Downward-drag dismiss of a top-anchored modal room** — conflicts with keyboard dismiss; handedness-neutral but weaker than a thumb-zone strip plus row morph.
- **Floating thumb-corner back button** — reliable and findable, but a small hit target and visually bolted-on compared with a structural strip + transform.
- **Moving the whole room chrome to a bottom bar** — largest IA change; solves reach but is unnecessary once the strip + container transform exist.
- **Bottom drag-handle only (no visible strip chrome)** — fast once learned, poor discovery without coaching.
- **Handedness-mirrored swipe direction as the primary fix** — unnecessary for a full-width strip; full-UI mirroring confuses users.
- **Keeping top-leading chevron as the only exit** — fails the thumb-reach intent.
- **Uniform scale of the list row into the room** — looks cheap (stretched type, squashed avatar); bounds + clip is required.
- **Morphing all row content into room content** — unstable; only avatar + name are shared elements.
- **Handle / grabber pill with no glyph or label** — the affordance that prompted this revision; reads as “drag something” without saying what, and is invisible to screen readers. Kept only as the fallback shape *with* a label.
- **Upward caret on the strip** — matches a literal “lift the room away” path but contradicts the list peeking from below and the depth-recede close; two direction cues that disagree read as a bug.
- **Close glyph (✕)** — implies discarding the conversation or leaving the room, not returning to a list.
- **Label-only strip** — scannable but low-contrast as an affordance; a glyph is what makes it read as a control at a glance.
- **Live list preview in the strip as the affordance** — already rejected above for looking tappable; it also cannot carry a stable label.
- **Repeating coach marks or a tutorial overlay** — a return action used hundreds of times a day must be legible from its own shape, not from instructions.
- **One continuous gesture that dismisses the keyboard and closes the room in a single motion** — elegant, but it puts an irreversible action at the end of the most common flick in the app; the sticky detent exists precisely to prevent it.
- **A dismiss control beside the composer (or in a keyboard accessory row) while composing** — puts a leave-the-screen target next to `send`, in the densest part of the layout, for a rare need.
- **Keeping the strip visible over the keyboard** — steals draft space and creates the same neighbouring-target problem.
- **Downward drag on the strip to pull the list up over the room** — collides with the OS home gesture and contradicts the collapse-into-row model.

## Platform notes

- iOS: matched-geometry / shared-element style transition; respect safe area and home indicator.
- Android (replay): Material container transform is the reference vocabulary; same product rules.
- Measure morph cost early on the lowest supported device before locking the animation as non-optional.

## Follow-on

- Exact strip copy and unread aggregation rules for “N new in other chats”.
- Prototype comparison of caret + label against grabber + label, to confirm the caret is not read as “scroll”.
- Whether the draft indicator on the collapsed row needs its own copy rules, or can reuse upstream’s draft presentation.
- Whether a later decision reintroduces a horizontal dismiss shortcut (explicitly out of scope here).
- Auto / inferred handedness for *other* thumb controls, if any are added later.
- Prototype check: does return-to-row feel correct after opening from a vertically scrolling list (direction is “back to place,” not “up”).

## History

- 2026-09-10: first write
- 2026-09-10: same intent, tighter design — strip affordance (caret + destination label), post-keyboard mis-tap guard, layout insets, system-back consistency, accessibility and split-layout rules
- 2026-09-10: added drag-to-close on the strip alongside tap, and staged dismissal (sticky detent + draft preservation) as the exit while the keyboard is open. Absorbed the detail that previously sat in the decision’s acceptance list

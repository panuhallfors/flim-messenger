---
decision: 001
slug: thumb-reach-room-exit
created: 2026-09-10
---

# 001 — Thumb-reach exit from a room or space (design)

Companion to `001-thumb-reach-room-exit.md`. **Replay-canonical** with that decision: later platforms (e.g. Android after iOS) implement this product design, not a parallel one.

## Context

Upstream opens a room (or a space) with a conventional navigation push and returns via the top-leading back control — far from the resting thumb. **This decision applies only to narrow (compact) layouts.** There it makes open/close a **container transform** (Material name; iOS: shared element / matched geometry): the list row grows into a full-screen **popup**, and dismiss shrinks and fades it back to its place. A **bottom strip** is the thumb-reachable, discoverable control that drives that dismiss. Upstream’s existing horizontal edge swipe-to-back (and other system / assistive back paths) **remain** on narrow layouts and drive the same close; what is out of scope is inventing an additional full-screen horizontal swipe-to-back as a fork gesture.

**Wide (regular-width / split) layouts:** do not apply this design. Keep upstream Element X room and space navigation and chrome exactly as today — including whatever back control (or lack of one) upstream already shows when the list sits beside the detail.

Everything below is written for narrow layouts, in terms of "a list" and "a row" opening into "a surface." The primary case is the conversation list opening a room. The same primitive recurses for spaces, which can nest before reaching a room — see "Recursing into spaces" below for what stays identical and what differs.

## Product design

### Open and close — container transform

- Tapping a conversation list row expands that row into the full room surface. Closing reverses the same transition onto the row’s place in the list.
- Animate **bounds** (width, height, corner radius), not a uniform scale of the whole element. Uniform scale stretches text and squashes the avatar; a growing clipped rectangle with content re-laid out inside is the intended look.
- **Crossfade** list-row content ↔ room content. Do not morph preview text into bubbles. Exception: **avatar and display name** travel as a true shared element from the row into the room header so one continuous object ties the transition together.
- Timing: old content fades out in roughly the first ~30% of progress; room content fades in from ~50% → 100%.
- **One progress scalar** `0 → 1` owns the expansion. Strip activation (and any later dismiss affordances) drive the same value. Use a spring-based, interruptible animation — not a fixed-duration timeline — so velocity can carry from gesture into settle, and a mid-gesture release can snap open or closed.
- Durations (when not gesture-driven): open ~220 ms, close ~160 ms. Close may be faster; the user already knows the destination.
- **Prefetch:** begin preparing the room surface on touch-down (not touch-up) so content or a skeleton is ready before the morph runs. Swapping a skeleton for real content mid-transition is worse than showing a skeleton through the end of the animation.
- **Source row missing or moved** while the room is open:
  - Row visible at a new index → close to the new on-screen rect (order change is informative).
  - Row scrolled off-screen → do **not** animate to an off-screen point; shrink toward the screen edge in that row’s direction and fade out.
  - Opened from notification / deep link with no source row → use a fallback (rise from the strip region, or centre fade). Separate transition; do not fake a row morph.
- **Reduced motion:** drop geometric morph; crossfade only.
- **Performance:** if the morph cannot hold frame budget on the weakest supported device (long room + large list), fall back to a simple crossfade for that device class. A stuttering morph is worse than an instant cut.

### Bottom strip — visible return

- The room (or space) is a **popup** over the list: it sits above that list in z-order. Directly **under the panel**, a full-width strip is the permanent, thumb-reachable control back toward the list. Dismiss is a **shrink and fade** along the container transform — the popup receding onto its row — not a lateral slide and not a sheet that travels the full screen height.
- The strip is the **default discoverable** exit. It must look like chrome for “return to the list,” not like another chat to open.
- **Always present while the room is open** on compact layouts — including while the composer keyboard is up. The strip does not hide, slide away, or get covered by the keyboard. That single layout is what makes return while composing the same action as return while reading.
- **Stacking when the keyboard is up**, top to bottom: room panel (composer at its bottom) → strip → keyboard. **Nothing sits between the strip and the keyboard.** The strip stays glued under the panel and rides up with it; the keyboard docks directly under the strip and may go edge-to-edge there. It must never draw over the strip.
- **Composer cluster stays above the strip.** Attachment picker, emoji picker, voice-message hold, location share, and suggestion / dictation / emoji bars all belong with the panel, immediately above the strip — the strip is simply the next band under that cluster, then the keyboard keys.
- **Peek content:** blur / dim heavily, or show a neutral surface. Never a sharp top list row with avatar + name that reads as “open this conversation.”
- **System gesture inset (keyboard down only):** when the keyboard is hidden, the phone’s home indicator / Android gesture-nav region is at the bottom of the screen. Leave a few millimetres so a downward system gesture is never captured by the strip’s drag; tap anywhere in the strip still works. That inset is system chrome at the bottom of the display — not a band of app UI between the strip and a keyboard. **Keyboard up:** the keyboard owns the bottom edge; the strip sits flush above it and does not compete with the home gesture.
- Closing should feel like lifting the popup off and returning it to its row. Prefer shrinking and fading along the transform over sliding the full screen height away (full-height slides feel slow on a high-frequency action).

### Strip affordance — making “this closes the room” legible

A bare peek or handle is not self-explanatory; the strip must be read correctly on first sight, without coaching.

- **Required content:** an **upward caret** plus a **destination label** naming the list exactly as its own screen title does (e.g. “Conversations”), centred as one pill-shaped unit. The glyph alone is ambiguous — the word is what removes guessing, and it is also the accessibility label.
- **Why an upward caret:** it matches the drag-to-close direction and reads as “lift this popup away.” Keep the _dominant read_ of the close animation as receding in depth (shrink + dim), with translation toward the row secondary, so the caret is not promising a vertical slide even when the target row sits high on screen.
- **Degradation, in order:** icon + label → icon only (largest accessibility text sizes, or very narrow widths) → never label only. Grabber-pill-plus-label is the accepted fallback if prototyping shows the caret reads as “scroll up” instead of “close”.
- **Whole strip is the hit target,** not just the pill: full width, at least the platform minimum touch height. Icon and label alignment is cosmetic; reach never depends on it.
- **Unread elsewhere:** if there are new messages outside the open room/space, show a small, quiet indicator on the strip (e.g. a dot) — presence only, no count and no copy like “3 new in other chats.” Do not animate or pulse it while the user is reading; set it when the strip is shown (or when unread state changes between glances) and keep it still.
- **First run:** one-time introduction only (brief label emphasis on the first room open, then settle). No repeating coach marks, no blocking tooltip.
- **Separation from neighbours:** the strip must be visually distinct from the panel bottom above it (composer) and, when typing, from the keyboard top below it (elevation edge or surface change) — so those neighbouring targets never read as one control.
- **Mis-tap guard while typing:** the strip sits in a sandwich between composer/`send` and the top of the keyboard. Leave a clear visual and touch separation on both edges (more than a hairline). Highest-risk hits are a fat finger dropping off `send`/composer, or climbing off the top keyboard row, onto the strip. Spacing and the strip’s own surface are what prevent that — not a settle window (the strip was already there; it only changes _who_ neighbours it).
- **Haptics:** a light confirmation on close commit; nothing on keyboard show/hide.

### Activation — tap and drag

Both activations drive the one progress scalar; there is no second close animation.

- **Tap** anywhere in the strip runs the close as a spring settle to `0`. This is the discoverable, guaranteed path and the accessibility path.
- **Drag upward** from the strip scrubs progress directly: the room shrinks toward its row as the finger travels, so the user sees the destination before committing. Release decides by position and velocity combined — a short flick with enough speed completes, a slow drag past roughly the halfway point completes, anything else springs back open. Velocity transfers from finger to spring; the gesture is interruptible at any moment, including re-grabbing mid-settle.
- **Direction:** only upward drag on the strip scrubs the close. A downward drag on the strip does not close the room and does not dismiss the keyboard (keyboard dismiss stays on timeline/composer / system back). When the keyboard is down, downward drag near the bottom edge belongs to the system home gesture — keep the strip’s drag-sensitive area clear of that inset as above. The strip never becomes a handle for pulling the list up over the room.
- **Rubber-banding:** dragging beyond fully-closed resists rather than overshooting into the list, so an over-long drag cannot skip past the list into some other state.
- Drag is a shortcut for people who discover it, never a requirement. Everything reachable by drag is reachable by tap.

### Closing while the keyboard is open

There is no separate compose-mode exit. The strip sits in the gap **below the panel and above the keyboard**, so tap or drag on the strip closes the popup the same way it does while reading — one control, one place relative to the panel, keyboard up or down.

- **The strip always closes the popup**, including while the keyboard is visible. Tapping (or drag-closing) the strip is not a keyboard-dismiss control; the keyboard goes away because the popup closed.
- **Keyboard dismiss vs popup close stay distinct for other gestures.** A downward drag on the timeline / composer still only dismisses the keyboard (platform-normal). Momentum from a keyboard-dismiss flick never continues into a popup close.
- **System back buttons** (Android back, hardware back) while the keyboard is up follow the platform two-step: first press dismisses the keyboard; a subsequent press closes the popup via the same progress scalar. That two-step applies only to those system back buttons — not to the strip, and not to iOS interactive edge-swipe (there is no back button; edge-swipe still closes the popup, keyboard up or down).
- **A draft is never lost by leaving.** Closing with text in the composer preserves the draft for that room, and the row the room collapses back onto shows a draft indicator — so the user watches the unsent text land somewhere visible instead of wondering whether it survived.
- **No extra close control next to `send`.** The strip is below the panel, not inside the composer row. Composer density stays intact.
- Screen-reader and external-keyboard users get the same labelled strip action (close the popup) whether or not the software keyboard is showing. Dismiss-keyboard remains a separate, prior action only for system back buttons and the usual composer/timeline dismiss, not for the strip.

### Layout consequences inside the room

- Inset the timeline’s bottom by the strip height so no message is permanently hidden behind it; scroll-to-latest and typing indicators sit **above** the strip, never overlapping it.
- **Pushes inside a room or space** (room settings, members, media viewer, message info, user profile, polls, and the rest of upstream’s inner stack) keep **upstream presentation and back chrome**. The strip, container transform, and chevron-removal apply only to the room/space join itself — the surface that opened from a list row — not to screens pushed on top of that join.
- Bottom-anchored transient UI (context menus, reaction pickers, snackbars) must not cover the strip while it is the only visible exit. Composer-owned sheets and bars stay above the strip as above.
- Contrast must come from the surface itself, not from blur alone — verify in dark mode and over media-heavy backgrounds.
- **Wide / split layouts:** out of scope for this decision — leave upstream room/space presentation alone (no fork strip, no fork transform, no fork chevron policy).

### Consistency with system back

- OS-level back (Android predictive back, iOS interactive edge-swipe from upstream navigation, VoiceOver escape gesture, hardware or external keyboard back) stays available and, when it actually closes the join, drives the **same progress scalar** and the same shrink-and-fade. Do not remove or disable upstream’s edge swipe-to-back. No separate dismiss animation, and no state where a completed system-back close and the strip disagree about where “back” goes. Android / hardware **back buttons** may dismiss the keyboard first (see “Closing while the keyboard is open”); that extra step is button behaviour, not a second close animation. The fork is not adding a new full-screen horizontal gesture — it keeps the platform’s own and wires the actual close to this popup dismiss.

### Accessibility and non-thumb use

- VoiceOver / TalkBack and desk / two-hand use still need an explicit “back to conversations” action. The strip owns that role: it is a real button, labelled with its destination, exposed at a predictable place in the reading order (adjacent to the composer, not buried after the timeline), and it responds to the platform’s standard escape gesture.
- Dynamic Type / font scaling grows the strip rather than truncating or clipping it; the touch target never shrinks below the platform minimum.
- Do not leave return discoverable only by gesture, and do not rely on colour or blur alone to distinguish the strip.
- **Remove the top-leading back chevron** on narrow layouts wherever this strip is present — for room and space **joins** alike. The strip is the on-screen exit; keeping both would leave two competing ways back and keep the old top-corner stretch in the chrome. Do not remove chevrons from inner pushes, which have no strip. On wide / split layouts, do not invent a fork rule: keep upstream’s chrome unchanged.

### Handedness

- This exit does not need a handedness setting: the strip spans the full width. Do not mirror the entire UI, bubble sides, or menu logic for handedness. Any future handedness preference (floating controls elsewhere) is a separate decision.

### Recursing into spaces

A space's row opens into that space's own list of children (rooms and/or nested sub-spaces) exactly the way a room's row opens into the room: container transform in, bottom strip back out. No new primitive — the same one, applied at whichever join is in front of the user.

- **One join at a time.** Space A → Space B → Room C closes C onto its row in B, then B onto its row in A, then A onto its row in the top-level spaces list — three independent transitions, each with its own strip pointing only at its immediate parent. A strip never skips a level. This keeps "collapses onto where it came from" true everywhere, not just at the room join, and it composes with system back for free (each join is its own back-stack entry).
- **Deep entry synthesises the missing ancestors.** Opening a room straight from a notification, search, or a deep link (skipping the spaces/rooms browse entirely) still needs a valid ancestor chain for the strip and system back to close onto — apply the existing "opened with no source row" fallback (rise from the strip region / centre fade) at each synthesised level, not just the room.
- **The strip's label is always the immediate parent's name** (a space's name, or the top-level spaces list's own title), never the eventual root. Closing is one level at a time; a jump-to-root shortcut is out of scope for this decision.
- **Telling a space screen from a room screen apart, at a glance:**
  - **Canvas tone.** A quiet, single-token background shift between the two: a space's screen (list of children, hero header) sits on a slightly different neutral surface than a room's screen (timeline + composer) — e.g. one step apart on the same neutral scale, not a change of hue or brand colour. Subtle enough that no one would describe it as "the app looks different here," but present enough to register peripherally over repeated use. Applies to the whole screen behind the header/timeline, not to the strip.
  - Both cues below are secondary to structural ones that already exist for free: a room screen has a composer above its strip, a space screen does not; a space screen carries a hero header (avatar, name, visibility, member count, topic), a room's top bar does not. Canvas tone and row style are reinforcement, not the primary signal — don't lean on them alone.
- **Telling a joined sub-space row from a joined room row apart, in the same children list:**
  - Upstream's `SpaceRoomCell` currently renders both kinds with the same furniture when joined — only unjoined rows get a "Join" accessory — so the tap destination is invisible until after the tap. That gap is in scope here.
  - **Row style (required).** Sub-space rows and room rows must not share an identical trailing / secondary layout when both are joined:
    - **Sub-space row:** keep (or lean on) the existing richer secondary line — visibility + member count — and add a quiet trailing disclosure cue (e.g. a chevron or space glyph) that reads as “opens another list.”
    - **Room row:** use a chat-list secondary — last-message preview and/or timestamp — and **no** disclosure chevron, so it reads as “opens a conversation.”
  - Keep the difference in the row's own furniture (leading/trailing accessory, secondary text), not colour-only, so it still reads in reduced-transparency and monochrome accessibility modes. Do not invent a second cell type for the sake of novelty — one cell with branched furniture is enough.
  - Accessibility: VoiceOver / TalkBack must announce the kind (“Space” / “Room” or equivalent) so the visual cue is not the only channel.
- **Strip content stays the same shape at every level:** caret + destination label. A space's strip just names a different destination (its parent space, or "Spaces") than a room's strip does. Do not invent a different affordance shape for the space case — consistency of the control is what lets it stay unlearned.
- **Wide / split layouts:** same as for rooms — this recursive paradigm does not apply; upstream space navigation stays as-is.

## Platform notes

- iOS: matched-geometry / shared-element style transition; respect safe area and home indicator.
- iOS: applies to both `RoomFlowCoordinator` (room join) and `SpaceFlowCoordinator` (space join, including its `presentingChild` nested-space state) — one shared transition/strip component, not a per-flow reimplementation.
- Android (replay): Material container transform is the reference vocabulary; same product rules.
- Measure morph cost early on the lowest supported device before locking the animation as non-optional.

## Open issues

_(none)_

## History

- 2026-09-10: first write
- 2026-09-10: same intent, tighter design — strip affordance (caret + destination label), post-keyboard mis-tap guard, layout insets, system-back consistency, accessibility and split-layout rules
- 2026-09-10: added drag-to-close on the strip alongside tap, and staged dismissal (sticky detent + draft preservation) as the exit while the keyboard is open. Absorbed the detail that previously sat in the decision’s acceptance list
- 2026-09-10: keyboard-open exit revised — strip stays visible in the gap below the panel and above the keyboard; staged dismissal and “strip below keyboard” rejected
- 2026-09-10: extended to the space/room hierarchy — same container-transform-plus-strip primitive recurses one level at a time through nested spaces to a room; added a quiet canvas-tone shift between space and room screens and a subtle row-style difference between room and (sub-)space rows in the same list, both as reinforcement of structural cues that already exist (composer presence, hero header)
- 2026-09-10: top-leading back chevron removed on compact layouts wherever the strip is present (rooms and spaces); keeping both rejected
- 2026-09-10: joined sub-space vs joined room rows distinguished by row furniture (disclosure + space secondary vs chat-list secondary); a11y announces kind
- 2026-09-10: clarified — keep upstream edge swipe-to-back (wired to the same close); reject only inventing a full-screen fork swipe
- 2026-09-10: layout scope — entire fork paradigm (transform, strip, chevron removal, space/room differentiation) is narrow/compact only; wide/split left exactly as upstream
- 2026-09-10: dropped Alternatives rejected and Follow-on; Open issues replaces Follow-on (empty)
- 2026-09-10: strip unread cue is a quiet presence indicator (dot), not count/copy text
- 2026-09-11: popup (shrink + fade), not an inverted bottom sheet; upward caret matching drag-up; strip always closes the popup even with the keyboard up (two-step keyboard-then-panel is system back buttons only); stacking is panel → composer cluster → strip → keyboard with nothing between strip and keyboard; inner pushes keep upstream chrome; composer-owned pickers and suggestion/dictation/emoji bars sit above the strip

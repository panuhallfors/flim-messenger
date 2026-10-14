---
id: 003
slug: flim-visual-refresh
title: Flim visual refresh (neutral surfaces, glass controls, signature gradient)
status: proposed
created: 2026-10-13
amends: [001]
platforms: [ios, android]
---

# 003 — Flim visual refresh (neutral surfaces, glass controls, signature gradient)

## Intent

Flim gets its own look on top of Element X: calmer neutral surfaces with more air, translucent glass for secondary controls, colour reserved for a few meaningful cues, and one signature gradient. The look applies **across the whole app**, not just the screens in the mocks: the mocks show the key screens, and every other screen gets the same colours and component styling.

This is a **restyle only**. Every piece of information and every control upstream shows stays. The navigation model does not change: no new popups, sheets, overlays or transitions beyond what decision 001 already defines. Narrow layouts keep 001's navigation, wide/split layouts keep upstream navigation, and both get the same visual language.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] The whole app has a recognisably Flim look, distinct from Element X, in both light and dark mode. Not just the conversation list, rooms, spaces and tab bar, but also settings, details and member screens, sheets, pickers, onboarding and the rest. No screen is left in the old Element styling.
- [ ] Colour is used sparingly and on purpose: surfaces, text and placeholder avatars are neutral, and colour only draws the eye to what is selected, a draft, or the signature line.
- [ ] Selected things in content (an active filter, your own messages, the open conversation in a split view) are recognisable at a glance without a solid coloured fill.
- [ ] Across the app, primary actions (such as Send and confirming a form) share one consistent look, and on/off and choice controls (toggles, checkmarks, sliders) share another. Neither uses Element's colours, and the two are never confused with each other.
- [ ] Unread conversations are recognisable at the row's trailing edge without relying on colour, and conversations that will notify still read as different from quiet unread ones.
- [ ] Nothing upstream shows is lost to the restyle: filters, room header subtitle and badges, unread / mention / mute / call indicators, sender-name colours, tabs and badges are all still there and still distinguishable.
- [ ] The look holds on narrow and wide layouts alike.
- [ ] Moving around the app works exactly as before this decision: the same screens open the same way (pushed, as a sheet, full-screen, or as 001's popup) with the same transitions.
- [ ] Everything stays legible and operable with large text, Increase Contrast, Reduce Transparency and in dark mode.

## Non-goals

- Any change to the navigation model: no new popups, sheets, overlays, panels or transitions, and no change to how any screen is presented, beyond what decision 001 defines. The popup panel and strip in the mocks are 001's, not new here. Narrow layouts follow 001, wide/split follow upstream.
- Rearranging screens: layouts, content order and information architecture stay as upstream (or as 001). Only the styling changes.
- Removing, hiding or merging any upstream information or control for the look's sake: filter pills, room header subtitle (presence, encryption, etc.), verification / history-sharing badges, status emoji, list-row badges, tabs (including Search), tab badges.
- A custom tab bar below iOS 26; older iOS keeps the system tab bar.
- Recolouring semantic states such as errors / failed sends, verification and destructive actions. These are not the "accent".
- Changing per-user sender-name colours in the timeline.
- Restyling content or UI we do not draw: system UI (alerts, share sheet, photo / document pickers, keyboard, context-menu chrome), web content (e.g. Element Call), and user media.
- A custom typeface; the system font stays.
- New app icon or launch / brand artwork (separate decision if wanted).

## Notes

Builds on decision 001 (popup room/space, bottom strip, row-style differentiation of sub-spaces vs rooms). Where 001 described visual detail, this decision is the source of truth. 001 was edited in place (not yet implemented) to remove the parts this overrides.

Upstream already matches several parts of the patch, so here the work is restyling rather than new behaviour: the room-list filters already have a leading clear (×) button and no "All" pill, the unread dot already sits on the row's trailing side, space avatars are already rounded squares, iOS 26 already renders the tab bar as a floating glass capsule, the composer already swaps the voice-message button for Send once there is text, and sheets already use confirm / cancel toolbar actions.

The "Draft:" row label is fork copy (the draft-on-row cue comes from 001) and goes in the Flim catalog from decision 002.

Design (replay-canonical with this decision): [`003-flim-visual-refresh.design.md`](003-flim-visual-refresh.design.md). Reference mocks: [`003-flim-visual-refresh.assets/`](003-flim-visual-refresh.assets/). Where a mock shows an accent/violet unread cue (including on the strip), treat that as outdated — implement `unread.strong` / `unread.soft` per the design, not the mock’s accent colour.

## Verification

### ios

- Tests: none yet (expect re-recorded preview snapshots for the affected screens, plus fork previews named with `003` for selected / unread / draft states)
- Agent UI: none

### android

- deferred

### web

- out of scope (Flim is iOS and Android only)

## History

- 2026-10-13: proposed
- 2026-10-13: explicit app-wide scope (all screens get the visual language, not only tokens); explicit "no navigation-model change beyond 001" in intent, acceptance and non-goals
- 2026-10-13: patch 2 (primary actions, Send, state controls, form sheets, settings) folded into the design; acceptance gains a bullet for consistent primary-action vs state-control styling
- 2026-10-14: platforms = ios, android
- 2026-10-14: design stack with 001 — app-owned bars below the strip, just above the keyboard; system IME chrome with the keyboard
- 2026-10-14: note — mock accent/violet unread (incl. strip) superseded by unread.strong / unread.soft

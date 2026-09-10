---
id: 001
slug: thumb-reach-room-exit
title: Thumb-reach exit from a room or space
status: proposed
created: 2026-09-10
amends: []
platforms: [ios]
---

# 001 — Thumb-reach exit from a room or space

## Intent

On **narrow (compact) layouts**, leaving a room, or a space, for the list it opened from must be reachable with the thumb without stretching to the top-leading back control. A room or space opens from its list row via a container transform and closes the same way; a bottom strip is the visible, full-width control that returns to that list. The same pattern applies at every level of the space/room hierarchy (spaces can nest inside spaces before reaching a room), one level at a time.

**Wide (regular-width / split) layouts are unchanged:** keep upstream Element X navigation and chrome exactly as today. This decision does not redesign them.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] On a narrow layout, a person holding the phone in one hand can open a conversation and return to the list using the thumb alone, without shifting grip or reaching the top of the screen.
- [ ] On a narrow layout, the way back is visible whenever a conversation is open — reading or composing — found without knowing a gesture, and it says where it leads.
- [ ] Returning while composing does not lose the draft.
- [ ] On a narrow layout, opening and closing read as the same conversation moving in and out of its place in the list, so the direction of travel feels logical and reversible rather than like an unrelated screen.
- [ ] The exit works the same in either hand, and does not depend on mirroring the rest of the interface.
- [ ] On a narrow layout, the top-leading back control is gone; return lives on the strip (and system / assistive back), not in the corner.
- [ ] The exit remains available and comprehensible with assistive technology, with large text, and with reduced motion.
- [ ] On a narrow layout, moving into and out of a space works the same way as a room, one level at a time, however many spaces are nested.
- [ ] On a narrow layout, a person can tell at a glance whether they are browsing a space's contents or reading/composing inside a room, even without reading any text.
- [ ] On a narrow layout, in a space's children list, a joined sub-space row is distinguishable from a joined room row before the tap — so “opens another list” vs “opens a chat” is readable without guessing.
- [ ] On a wide / split layout, room and space navigation match upstream Element X — no fork chrome from this decision.

## Non-goals

- Inventing a bidirectional (or any) full-screen horizontal swipe-to-back as a fork exit. Upstream’s existing edge swipe-to-back remains and must not be removed.
- Handedness / left–right mirroring of the whole UI, message bubble sides, or menu logic.
- Swipe-to-next-room, swipe-for-message-info, or other horizontal room-level gestures on this surface.
- Replacing the composer, attachment, or send controls with the strip.
- Mandating a handedness setting for this exit (the strip is full-width and hand-agnostic).
- A control that jumps straight to the root of the space hierarchy from a deeply nested room in one action (closing is one level at a time).
- Changing wide / split (regular-width) room or space navigation — leave upstream as-is; this decision applies only to narrow (compact) layouts.
- Redesigning screens pushed on top of a room or space (settings, members, media, message info, and the rest of the inner stack) — those keep upstream presentation and back chrome.

## Notes

Upstream Element X keeps room → list exit, and space → list exit, on the top-leading back control (and whatever chrome it uses in split layouts). On **narrow layouts**, this decision replaces that exit model for listed platforms, for both rooms and spaces. On **wide / split layouts**, upstream stays.

Codebase terms (iOS): a **space** is a `SpaceServiceRoom` with `isSpace == true`, browsed via `SpacesScreen` (top-level) / `SpaceScreen` (inside a space); a **room** is the conversation itself (`JoinedRoomProxyProtocol`, `RoomFlowCoordinator`). Spaces can nest (`SpaceFlowCoordinator`'s `presentingChild` state) before reaching a room.

Design (replay-canonical with this decision): [`001-thumb-reach-room-exit.design.md`](001-thumb-reach-room-exit.design.md)

## Verification

### ios

- Tests: none yet
- Agent UI: none

### android

- deferred

### web

- deferred

## History

- 2026-09-10: proposed
- 2026-09-10: acceptance tightened (strip affordance, hit target, post-keyboard guard, content insets, system-back parity, split layout) — same intent
- 2026-09-10: acceptance rewritten at outcome level per revised `AGENTS.md` guidance; the detail it carried moved into the design. Design gained drag-to-close on the strip and a staged exit while the keyboard is open
- 2026-09-10: acceptance wording — way back visible while composing too; design keeps the strip in the gap below the panel (above the keyboard when shown)
- 2026-09-10: scope widened to the space/room hierarchy — title, intent, acceptance and non-goals now cover spaces (which can nest) recursing the same open/close pattern, and telling a space screen apart from a room screen at a glance. Design gained a matching section
- 2026-09-10: acceptance + design — top-leading back control is removed wherever the strip is present (was soft “may be removed”)
- 2026-09-10: joined sub-space vs joined room rows in the same list distinguished by row style (was a non-goal / follow-on)
- 2026-09-10: clarified — upstream edge swipe-to-back stays; only a fork-invented full-screen horizontal swipe is rejected
- 2026-09-10: scope — fork paradigm (transform, strip, chevron removal, space/room differentiation) applies only to narrow/compact layouts; wide/split stays upstream unchanged
- 2026-09-10: design guidelines — no Alternatives rejected / Follow-on; Open issues (empty) instead
- 2026-09-11: non-goal — inner pushes stay upstream; design clarifications (upward caret, popup shrink+fade, strip vs system-back, composer cluster above the strip) — same intent

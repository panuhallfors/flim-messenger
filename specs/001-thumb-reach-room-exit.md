---
id: 001
slug: thumb-reach-room-exit
title: Thumb-reach exit from a room
status: proposed
created: 2026-09-10
amends: []
platforms: [ios]
---

# 001 — Thumb-reach exit from a room

## Intent

Leaving a room for the conversation list must be reachable with the thumb without stretching to the top-leading back control. A room opens from its list row via a container transform and closes the same way; a bottom strip is the visible, full-width control that returns to the list.

## Acceptance

User-observable outcomes. The design document specifies how these are achieved.

- [ ] A person holding the phone in one hand can open a conversation and return to the list using the thumb alone, without shifting grip or reaching the top of the screen.
- [ ] The way back is visible while reading a conversation — it can be found without knowing a gesture, and it says where it leads.
- [ ] Returning is also possible while composing a message, without losing the draft.
- [ ] Opening and closing read as the same conversation moving in and out of its place in the list, so the direction of travel feels logical and reversible rather than like an unrelated screen.
- [ ] The exit works the same in either hand, and does not depend on mirroring the rest of the interface.
- [ ] The exit remains available and comprehensible with assistive technology, with large text, and with reduced motion.

## Non-goals

- Bidirectional (or any) full-screen horizontal swipe-to-back as a primary or secondary exit.
- Handedness / left–right mirroring of the whole UI, message bubble sides, or menu logic.
- Swipe-to-next-room, swipe-for-message-info, or other horizontal room-level gestures on this surface.
- Replacing the composer, attachment, or send controls with the strip.
- Mandating a handedness setting for this exit (the strip is full-width and hand-agnostic).

## Notes

Upstream Element X keeps room → list exit on the top-leading back control. This decision replaces that exit model for listed platforms.

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

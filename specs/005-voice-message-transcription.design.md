---
decision: 005
slug: voice-message-transcription
created: 2026-10-13
---

# 005 — Voice message transcription (design)

Companion to `005-voice-message-transcription.md`. **Replay-canonical** with that decision: later platforms implement this product design, not a parallel one.

## Context

Decision 004 shows a title, summary, and transcript on a voice message and sends the title and summary to the room. The words come from one in-app service. Callers do not know which method produced them.

This decision is how the user chooses that method and finishes its setup. It amends 004 where a brief was attempted with no choice of method. The race, the wait, the notice, and the failure statuses stay as 004 specifies.

Copy on these screens describes the working service: where the voice message goes, and what the user gets back.

## Product design

### Where it lives

The root settings screen has its own section, **Voice message transcription**. It is not nested under notifications, labs, or account management.

The section is one row. The row names the method in use, or says that none is set up. It opens the transcription screen.

### Selector first

The transcription screen opens on a selector. One method can be in use. Nothing is in use until that method's setup has been finished. The choices:

1. **Flim** — Flim's own service, hosted in the EU. The voice message and the room context from decision 004 are sent there. The service returns a title, a summary, and a transcript.
2. **Custom server** — a server the user names. The voice message is sent there to be transcribed, and the transcript and room context are sent there to be summarised.
3. **On this device** — experimental. The voice message stays on the device. Quality is not as good as an external service.

Each row says that, in ordinary language, before the user commits. Tapping a row starts that method's setup. It does not switch the method in use until setup finishes.

The method already in use is marked. Opening it shows its finished setup (the account and plan, the saved server, or the on-device confirmation), not the first step again. Switching away keeps the other method's saved setup, so coming back does not ask the user to register twice. Switching away does change which method is in use, once the newly chosen setup is finished.

With no method in use, a voice message does not produce a brief. The brief's place shows that transcription is not set up yet, and Try again. After a method is in use, Try again runs that method.

### Flim

Setup is a short sequence. One step at a time.

1. **Email.** One address, then continue.
2. **Code.** Flim sends a one-time code to that address. The user enters it. They can ask for a new code. A new address creates the account. An existing address signs in.
3. **Plan.** Transcription with Flim needs a plan on this account. The step offers both:
   - a promo code, which attaches a free plan
   - a subscription, bought in the app

   The user completes one. Cancelling the purchase, or a promo the service refuses, stays on this step and says what happened, in plain language. The other option remains.

Finishing the plan step makes Flim the method in use and opens the account screen: the email, the plan, and sign out.

An account that already has a plan skips the plan step and lands on that account screen. A second device uses the same email and a new code, and does not redeem another promo or buy another subscription for a plan that is already there.

Leaving before the plan step does not make Flim the method in use. The account is kept: returning continues at the plan step, not at the email.

The account is the email, not the Matrix user. One Flim transcription account serves the Matrix sessions on this device. Signing out of Matrix does not sign it out. Signing out is on the account screen. After sign-out, Flim is not the method in use.

The subscription bought here is attached to this account. Restoring purchases is offered on the plan step, and on the account screen when a signed-in account has no active plan. A restored subscription attaches to the account they are signed in to.

Flim's service holds the prompts. The app sends the voice message and the room context.

With an active plan whose allowance is used up, the brief shows decision 004's allowance failure. The bubble does not open a store. The plan step and the account screen are where the user subscribes or applies a promo.

### Custom server

One screen, after the selector:

- transcription address and model name
- summary address and model name — the same address may fill both, and the screen makes that the easy path
- a key

Whisper and Parakeet are both chosen by the model name. Saving, once the required fields are filled, makes this the method in use.

The key is write-only after it is saved: the screen does not show it back in full. It is not placed in the Matrix account.

The app asks that server for a transcript, then for a title and summary from the transcript and the room context.

### On this device

The quality note is on the choice before it is turned on: experimental, and not as good as an external service.

When the device can both transcribe speech and run an on-device model, confirming the choice makes it the method in use. What "can" means, including models that are still preparing, and how that method produces the brief, is decision 007. This screen confirms the choice or shows the reason 007 supplies.

When the device cannot, the row stays visible, cannot be confirmed, and says why (what the device lacks, or that the models are still preparing). The user remains on the selector. Another method can still be set up.

## Platform notes

- Both: the section is on the root settings screen. The selector is the next screen; each setup step is the next screen after that. Strings are Flim catalog keys (decision 002), not Element's `Localizable` / `Untranslated` files. All three choices are shown on iOS and on Android.
- iOS: the custom-server key and the Flim session live in the Keychain. They are not written to preferences in the clear, and not written into Matrix account data.
- Android: the custom-server key and the Flim session live in the platform credential store. They are not written to preferences in the clear, and not written into Matrix account data.
- Both: the subscription uses the platform purchase sheet. The account already exists before the sheet is shown, and the purchase is tied to that account.
- Both: on-device availability follows decision 007. The row is not hidden.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: end-state setup only. Selector, then one step at a time for Flim (email, code, promo or subscription). No interim behaviour in this design
- 2026-10-13: failure statuses removed here; they are specified on decision 004
- 2026-10-13: on-device capability test and listener are decision 007; this screen only confirms or shows that reason
- 2026-10-13: Android shows the same three choices as iOS

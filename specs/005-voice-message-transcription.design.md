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

With no method in use, a voice message does not produce a brief. The brief's place shows that transcription is not set up yet; the action opens setup (decision 009 amends 004 — not Try again). After a method is in use, Try again on other failures runs that method.

### Flim

Setup is a short sequence. One step at a time.

1. **Email.** One address, then continue.
2. **Code.** Flim sends a one-time code to that address. The user enters it. They can ask for a new code. A new address creates the account. An existing address signs in.
3. **Plan.** Transcription with Flim needs a plan on this account. The step shows both:
   - a promo code, which attaches a free plan
   - a subscription bought in the app — **visible but disabled**, with plain-language copy that paid subscription is **coming later** (English source: “Coming later.”). Restore purchases is likewise shown disabled with the same coming-later note when it would otherwise appear.

   The user finishes the plan step with a promo the service accepts. A promo the service refuses stays on this step and says what happened, in plain language. The disabled subscription control cannot be completed.

Finishing the plan step makes Flim the method in use and opens the account screen: the email, the plan, and sign out.

An account that already has a plan skips the plan step and lands on that account screen. A second device uses the same email and a new code, and does not redeem another promo for a plan that is already there.

Leaving before the plan step does not make Flim the method in use. The account is kept: returning continues at the plan step, not at the email.

The account is the email, not the Matrix user. One Flim transcription account serves the Matrix sessions on this device. Signing out of Matrix does not sign it out. Signing out is on the account screen. After sign-out, Flim is not the method in use.

When a later decision enables store billing, the subscription bought here will attach to this account, and restore will be offered on the plan step and on the account screen when a signed-in account has no active plan. Until then those controls stay disabled with the coming-later copy.

Flim's service holds the prompts. The app sends the voice message and the room context.

With an active plan whose allowance is used up, the brief shows decision 004's allowance failure. The bubble does not open a store. The plan step and the account screen are where the user applies a promo (or, later, subscribes).

### Custom server

One screen, after the selector. It assumes the user has not set up an API before. The explanation comes first, in ordinary language, then the fields.

Flim needs an account at a service that can listen to a voice message and write a short summary. Any service that speaks the OpenAI API works. The user can create one at **OpenAI**, **Cortecs** (cortecs.ai, a European provider), or **OpenRouter**, or use an account they already have. The voice message is sent to that service.

On that service's site:

1. Open the page for **API keys**. Create a key and copy it. Paste it into the key field here. The service usually shows the full key only once.
2. Copy the **address** the service shows for its API (sometimes called a base URL). Paste that into the address field. The start of the address is enough — not a long path.
3. Copy two **model names** from the service's model list: one that listens, and one that writes the summary. Whisper and Parakeet are both fine as the listening model when that service lists them.

The screen names the three examples and gives the address to paste when it is a fixed one:

| Service | Where the key is | Address to paste |
|---|---|---|
| OpenAI | [platform.openai.com/api-keys](https://platform.openai.com/api-keys) | `https://api.openai.com/v1` |
| Cortecs (European) | [cortecs.ai](https://cortecs.ai), then Credentials → Create API key | `https://api.cortecs.ai/v1` |
| OpenRouter | [openrouter.ai/keys](https://openrouter.ai/keys) | `https://openrouter.ai/api/v1` |

One address and one key cover both listening and the summary. The screen starts that way. A second address is there only if the summary lives somewhere else.

When the user enters a **second (summary) address** that is different from the transcription address, show a **second key** field for that host. It defaults to **“same key”** (reuse the transcription key) so a single-provider split URL stays easy; the user can paste a different key when the summary host is another provider. Never imply that one provider’s key is sent to another host unless the user chose “same key” or typed the same secret into both fields.

The fields, in everyday labels as well as the names above:

- transcription address and model name
- summary address and model name
- transcription key
- summary key (only when a distinct summary address is in use; otherwise one key covers both)

Saving, once the required fields are filled, makes this the method in use.

Each key is write-only after it is saved: the screen does not show it back in full. Keys are not placed in the Matrix account.

The app asks that server for a transcript, then for a title and summary from the transcript and the room context. Decision 006 sends each key only to its own address.

### On this device

The quality note is on the choice before it is turned on: experimental, and not as good as an external service.

When the device can both transcribe speech and run an on-device model, confirming the choice makes it the method in use. What "can" means, including models that are still preparing, and how that method produces the brief, is decision 007. This screen confirms the choice or shows the reason 007 supplies.

When the device cannot, the row stays visible, cannot be confirmed, and says why (what the device lacks, or that the models are still preparing). The user remains on the selector. Another method can still be set up.

## Platform notes

- Both: the section is on the root settings screen. The selector is the next screen; each setup step is the next screen after that. Strings are Flim catalog keys (decision 002), not Element's `Localizable` / `Untranslated` files. All three choices are shown on iOS and on Android.
- iOS: the custom-server key(s) and the Flim session live in the Keychain. They are not written to preferences in the clear, and not written into Matrix account data. When two keys are saved, store them as separate Keychain entries keyed by role (transcription vs summary).
- Android: the custom-server key(s) and the Flim session live in the platform credential store. They are not written to preferences in the clear, and not written into Matrix account data. When two keys are saved, store them as separate entries keyed by role (transcription vs summary).
- Both: when store billing is enabled later, the subscription uses the platform purchase sheet. The account already exists before the sheet is shown, and the purchase is tied to that account. Until then the control stays disabled.
- Both: on-device availability follows decision 007. The row is not hidden.

## Open issues

_(none)_

## History

- 2026-10-13: first write
- 2026-10-13: end-state setup only. Selector, then one step at a time for Flim (email, code, promo or subscription). No interim behaviour in this design
- 2026-10-13: failure statuses removed here; they are specified on decision 004
- 2026-10-13: on-device capability test and listener are decision 007; this screen only confirms or shows that reason
- 2026-10-13: Android shows the same three choices as iOS
- 2026-10-13: custom-server setup explains, for a non-technical user, how to get a key and an address from OpenAI, Cortex, or OpenRouter (or an equivalent service)
- 2026-10-13: the example is Cortecs (cortecs.ai), named as a European provider; any OpenAI-compatible service works
- 2026-10-14: plan step — subscription and restore visible but disabled; coming-later copy; promo is the working path
- 2026-10-14: platforms = ios, android
- 2026-10-14: distinct summary address gets a second key field (default “same key”); never cross-send keys by accident
- 2026-10-14: amended by 009 — not-set-up action is setup, not Try again

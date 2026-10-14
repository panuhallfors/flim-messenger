# Decision log review — 2026-10-14

Scope: every entry in [`specs/INDEX.md`](specs/INDEX.md) (000 – 009, decisions and designs), read against the product goal:

1. easy to use with one hand (room / space navigation),
2. voice messages are transcribed and summarised,
3. its own branding, separate from upstream Element X.

Each finding is labelled **Blocker** (the app is broken, cannot ship, or the spec cannot be implemented as written), **Major** (a real contradiction or a product gap a user will hit), or **Minor** (wording, consistency, polish). Recommendations are suggestions for new or amending decisions. Nothing here changes the specs.

---

## TL;DR

- **The log is not yet enough for a working fork.** 000a only renames the app and changes its identifiers. A build made from 000a alone **will not receive push notifications**: upstream's push gateway is `matrix.org`, which can only deliver to Element's own bundle id. The OIDC sign-in consent screen, the privacy, terms and help links, analytics, and bug reports still point at Element. No decision owns the **app icon**: 000a and 003 both list it as a non-goal.
- **Transcription has three product-level problems:**
  - Nobody can turn the feature off. Every voice message shows a "not set up" failure until the user picks a method.
  - Recipients transcribe automatically and with no trigger defined. Scrolling back through history could use up the quota and send other people's audio to a cloud service without their consent.
  - The sender's AI-written, first-person summary is published under their name with no preview, no undo and no opt-out.
- **Three spec contradictions break implementation as written:**
  - The companion notice cannot carry both a custom `rel_type` and a thread relation.
  - The system keyboard's own suggestion bar cannot sit above 001's strip.
  - The Flim "allowance used up" message tells users to buy a subscription that 005 and 008 make impossible to buy.
- **Platform scope doesn't line up.** 000a, 001, 002 and 003 are iOS-only. 005–009 require Android. An Android build would have transcription, but no Flim identity, look or navigation.
- **One-handed use covers leaving a room or space and nothing else.** Search, new chat, filters, room details and settings all stay at the top of the screen.

---

## 1. Contradictions between decisions (or with workspace policy)

### 1.1 Blocker — Companion notice cannot be both a custom relation and in a thread (004)

004's design sets `m.relates_to.rel_type = me.flim.voice_summary` and also says _"If the voice message is in a thread, the notice is in that same thread."_ In Matrix, an event has exactly one `rel_type`, and a thread reply needs `rel_type: m.thread` pointing at the thread root. Both can't be true.

**Recommendation:** amend 004's protocol. The notice can use `m.thread` (with `m.in_reply_to` the voice event) when the voice message is in a thread, and the custom relation everywhere else. Alternatively, move the link to the voice event into the `me.flim.voice_summary` content block (`voice_event_id`) and make the trust check use that.

### 1.2 Blocker — Stack order in 001 needs the system suggestion bar above the strip, which can't be done

001 (and the L3/D3 mock in 003) stacks: panel → composer → **suggestion / dictation / emoji bars** → strip → keyboard keys. On iOS the QuickType predictive bar, the dictation UI and the emoji search bar are part of the **system keyboard**. On Android the suggestion strip belongs to the input method (the keyboard app). The app cannot place its own view between them and the keys. Only app-drawn bars (Element's @-mention and command suggestions) can sit above the strip.

**Recommendation:** amend 001 and 003 by narrowing the rule to _app-owned_ bars. Say that the system candidate bar stays attached to the keyboard, so the real stack is composer → app bars → strip → [system suggestion bar + keys]. Redo the 001 mis-tap analysis for that stack: the strip now sits next to the suggestion bar, not the top row of keys. Update the mock.

### 1.3 Major — "Buy a larger subscription" when no subscription can be bought (004 vs 005 / 008)

004 says the Flim allowance failure must say _a larger transcription-service subscription is needed_. 005 and 008 make subscriptions visible but disabled ("Coming later"). The only plan is the 100-per-day promo plan, and it resets at UTC midnight. A user who runs out is sent to a control that does nothing.

**Recommendation:** amend 004 so the wording depends on the plan. On the free promo plan, say the daily limit is used up and give the reset time. Use the subscription wording only once store billing ships.

### 1.4 Major — Flim's own string catalog vs workspace policy (002 vs `AGENTS.md`, `ios/AGENTS.md`, `jev-pref.json`)

- Root `AGENTS.md` (Jev rules), `ios/AGENTS.md` and `jev-pref.json` all say _"new English strings go in `Untranslated.strings`"_.
- 002, 004, 005 and 009 say the opposite for fork copy: Flim strings go in `Flim.strings`, **never** in `Untranslated`.
- 002's design also says _"(add this to AGENTS.md)"_, and that hasn't been done.

An implementing agent or the Jev reviewer following the workspace rules will put fork strings in the wrong place, or flag the right place as a violation.

**Recommendation:** update both `AGENTS.md` files with a two-catalog rule: fork-owned copy goes to `Flim.strings` (en-GB + fi); upstream-bound copy goes to `Untranslated`. Then run `npx jev-pref sync`.

### 1.5 Major — On-device "not ready" shows up as "not set up" (007 vs 005 / 009)

007 says that if the on-device check later fails, _the method stays selected_ and settings says "not ready". Its failure table maps _"the phone can no longer transcribe and summarise on device"_ to **Not set up**, and 009 turns that status into a setup call-to-action. A user who did set up on-device, and whose models are re-downloading after an OS update, is told _"Transcription isn't set up"_ and pushed into setup. 007 also maps "models not ready when the attempt runs" to _Generic failure_, which overlaps with the same case.

**Recommendation:** map every "selected but not currently able" case to one status (Generic failure, with "on this device isn't ready yet" detail text, plus Try again). Use **Not set up** only when no method is in use.

### 1.6 Major — Custom server: two addresses, one key (005 / 006)

005 allows a second address for the summary when it _"lives somewhere else"_, but there is only one key field. 006 sends that key as a bearer token to both addresses. With two providers, the second call fails as _Sign-in rejected_, and the first provider's key is sent to the second provider.

**Recommendation:** when a second address is entered, show a second key field (defaulting to "same key"). Never send one provider's key to a different host.

### 1.7 Minor — 004's acceptance stops being true after 005–008

004's acceptance and design say the shipping app uses the mock in real rooms. Successful results publish _sample text_ with "Made with Flim", and _every third call fails_. After 005, nothing runs without a method. After 006, 007 and 008, every method has a real engine, so the mock is unreachable in production. 006's acceptance even says other paths _"still use the mock"_ until their decisions land. No decision says **"the mock must never publish to a real room once 005 lands."** Under partial replay (for example, 008's server not deployed yet), real rooms would get fake summaries attributed to Flim.

**Recommendation:** an amending decision that makes the mock preview/test-only from 005 onward, and turns 004's sample-text and every-third-call acceptance bullets into "superseded".

### 1.8 Minor — 005 still says "Try again" for "not set up"

005's design (Selector first, last paragraph) still says the not-set-up bubble offers _Try again_. 009 amends 005 and later wins, but 005's History has no "amended by 009" line, unlike 004's.

### 1.9 Minor — Title and summary limits don't match

- **Title:** 004 says at most 50 characters. The 006 and 007 prompts say at most 4 words / 35 characters. 008 says "short" and lets the service's own prompts differ.
- **Summary:** 004 says "about two sentences". The 006 and 007 prompts say at most four sentences / 20 words / 160 characters.

The product limit should live in one place (004), with prompts aiming under it. Right now each method produces visibly different brief shapes.

### 1.10 Minor — 001 vs 003 mocks

The 003 mocks show a violet (accent) unread dot on the strip, while the 003 text requires `unread.strong`. This is covered by the "mocks superseded" note, but implementers copy mocks. Re-export the mock, or annotate it.

---

## 2. Omissions — what a working fork still needs

### 2.1 Blocker — Push notifications

`AppSettings.pushGatewayBaseURL` is `https://matrix.org`. That push gateway (Sygnal) holds Element's APNs/FCM credentials for Element's app ids only. With the new bundle id from 000a, **no push notifications arrive.** Neither decision 000a nor its human checklist mentions this.

**Recommendation:** add to 000a's human setup, or as its own decision, since it has a product side (privacy, where push metadata goes): host a Sygnal (EU, next to the 008 service), set up an APNs key and an FCM project, and set the pusher app ids. Make "receives a push on a locked phone" a 000a acceptance item.

### 2.2 Major — Element links and services still configured upstream (000a)

Today these still point at Element:

- **OIDC / MAS consent:** `clientName` already comes from the display name, but `logoURL`, `clientURI` (`websiteURL`), `tosURI` and `policyURI` are `element.io`. `oAuthRedirectURL` is `element.io/oauth/ios/<bundle id>`. The consent screen shows "Flim Messenger" with Element's logo and Element's terms. The redirect can't be completed for our bundle id without our own domain.
- **Privacy, acceptable use, copyright and help links** (encryption, verification, backup): `element.io`.
- **Analytics** (PostHog), **Sentry** and the **rageshake bug report** (`bugReportApplicationID = "element-x-ios"`): users would agree to _Element's_ analytics, or file bug reports to Element.
- **`accountProvisioningHost`** and `elementWebHosts` (permalinks): `element.io`.

**Recommendation:** extend 000a's design, or write a 000b "Flim service endpoints", so it lists each one as replace, disable, or keep on purpose. For analytics and bug reports, the sensible first ship is to disable them (the upstream consent screen then disappears).

### 2.3 Major — App icon and launch artwork have no owner

000a lists _"icons… beyond display name + identifiers"_ as a non-goal. 003 lists _"New app icon or launch / brand artwork (separate decision if wanted)"_ as a non-goal. For a separately branded app, the icon is the most visible brand element, and Element's logo and name are trademarks. Shipping Flim with Element's icon is confusing and a trademark risk.

**Recommendation:** a branding decision for the app icon (light, dark and tinted variants on iOS 26; adaptive on Android), the launch screen, the onboarding / splash artwork, notification icons, and App Store / Play listing assets.

### 2.4 Major — Home-screen name will be cut off (000a)

`APP_DISPLAY_NAME = Flim Messenger` is 14 characters. The iOS home screen shows about 11–12 characters before it adds an ellipsis ("Flim Messe…"), and many Android launchers truncate too.

**Recommendation:** use **Flim** as the home-screen display name and keep **Flim Messenger** as the store listing name. Amend 000a's acceptance item, which currently requires "Flim Messenger" on the home screen.

### 2.5 Major — Android scope doesn't match (000a – 003 vs 005 – 009)

005–009 list `platforms: [ios, android]`. 000a, 001, 002 and 003 are `ios` only, and 000a explicitly defers the Android application id. Following the log as written gives an Android app with Flim transcription screens and the `me.flim` event namespace, but no Flim app id, catalog, look or one-handed navigation. `AGENTS.md` asks every decision to consider both platforms.

**Recommendation:** add `android` to 000a–003 (with `deferred` verification), or make Android `deferred` in 005–009. Don't leave the Android build in a mixed state.

### 2.6 Major — Store, privacy and legal requirements for the Flim account (005 / 008)

These items are based on current App Store / Play policy as I understand it. Check them against the latest review guidelines.

- **Account deletion.** An app that lets users create an account must let them delete it inside the app (App Store 5.1.1(v), and Play has a similar rule). 005 and 008 only offer sign-out. GDPR erasure applies to an EU service in any case.
- **Sharing personal data with AI services.** Apple now requires clear disclosure and explicit permission before personal data is shared with third-party AI. Flim mode sends audio plus five earlier messages from _other people_ in end-to-end-encrypted rooms to Cortecs / Mistral. Custom mode sends them to OpenAI or OpenRouter. 005's selector copy says this, but there is no explicit consent step and no privacy policy / terms for the Flim service.
- **Promo codes as the only unlock.** Unlocking a feature with a code redeemed against an external account, while the in-app purchase is disabled, is a known App Review risk (3.1.1). A visibly disabled "Coming later" button can also be rejected as placeholder content.
- **Session lifetime.** 008 doesn't say when a Flim session expires or how it refreshes. _Sign-in rejected_ tells the user to "fix it in Settings", but the account screen only offers sign-out.
- **Limits.** No maximum voice-message length or upload size, and no promo-plan expiry, is defined. The cost of the free plan is open-ended.

### 2.7 Major — The companion notice can be too big (004)

The full transcript goes in the notice's content. Matrix events are limited to 64 KiB, and encryption overhead adds to that. A long voice message will fail to send, and that failure isn't one of 004's statuses.

**Recommendation:** cap the transcript stored in the notice, and specify what happens past the cap: truncate with a marker, or leave the transcript out.

### 2.8 Minor — Finding old briefs (004)

When a Flim-marked voice message scrolls into view from history, its companion notice may not be loaded yet, so 004's rule waits 15 seconds and then transcribes again. The design should say to look up existing companions through the relations API before waiting. Otherwise each old voice message costs a call.

### 2.9 Minor — Hidden notice can still mark the room unread

The companion notice is hidden from the timeline but is still an event. It may set the room's "has unread" state with nothing visible to read. Specify that a trusted companion never counts toward the client-side unread state.

### 2.10 Minor — Hardcoded upstream "Element" strings

Upstream's English catalog has only 11 strings that say "Element" literally. Most name other products (Element Call, Element Pro, Element Classic). "Element Default" / "Element Fade" (notification sounds) and _"use the new Element X app"_ do read as our branding. 002 forbids patching Element keys, so these need Flim keys at their call sites. A one-line audit item in 000a covers it.

---

## 3. Usability and product design

### 3.1 Major — No way to turn transcription off (005 / 009 / 004)

The methods are Flim, custom server and on-device. There is no **Off**. A user who chooses "Not now" in 009, or who simply doesn't want the feature, sees a _"Transcription isn't set up"_ failure block with a setup link under **every voice message, forever**. A deliberate choice is treated as an error.

Also, 004 marks **every** Flim-sent voice message _"a brief may follow"_, even when the sender has no method. Every recipient running Flim then waits 15 seconds for a brief that will never come.

**Recommendation:** add an explicit **Off** choice to 005's selector. With Off, show no brief block at all (only the player), and don't send the marker. Send the marker only when a method is in use. Keep the 009 prompts for users who haven't chosen yet, and stop prompting once they choose Off.

### 3.2 Major — When does a recipient transcribe? (004)

004 says _"when a voice message arrives… start the service immediately."_ It doesn't say whether "arrives" means synced, shown in the timeline, or opened. Read literally:

- Voice messages in rooms the user never opens get transcribed in the background.
- Scrolling back through an old room starts one call per historical voice message. A busy group can use up the 100-per-day Flim plan, or the user's own paid custom-server key, before lunch.
- Other people's audio (and the context messages) leave the device without the user ever looking at them.

**Recommendation:** start a recipient's brief **when the voice message is shown in the timeline**. Never do it on background sync, and never for history older than a threshold, where it is tap-to-transcribe instead. Optionally add a setting for "Automatic / on tap".

### 3.3 Major — Publishing an AI summary in the sender's name (004)

The sending device automatically publishes a title and summary written by a model, **in the first person** ("I like roses"), to everyone in the room. There is no preview, no edit, no per-message undo and no "don't share" setting. With on-device mode (lower quality, by 005's own copy) or a 35-character limit, a wrong summary will sometimes say something the sender didn't say, in their voice. "Made with Flim" helps, but recipients often read only the summary.

**Recommendation:**

- A setting "Share title and summary with the room" (worth discussing whether it should default on).
- A "Remove summary" action on your own voice message that redacts the notice.
- Consider third-person or neutral wording for the _shared_ summary, while keeping first person for local display.

### 3.4 Major — The full transcript always shows in the timeline (004)

The block shows title, summary **and the full transcript** under every voice message, with no collapsed state. A 3-minute voice message adds a wall of text to the timeline and defeats the point of having a summary.

**Recommendation:** show title + summary by default, with the transcript behind a "Show transcript" disclosure. The transcript should also be selectable and copyable.

### 3.5 Major — Recipients' consent (004 / 005 / 008)

A recipient in Flim or custom mode sends _someone else's_ voice and the five earlier messages to a cloud service, without that person knowing, including from end-to-end-encrypted rooms. Combined with 3.2 this is the biggest trust risk in the log for a Matrix (privacy-focused) client.

**Recommendation:**

- Default room context to "off" for cloud methods, or limit it to the reply target.
- Let users turn transcription off per room.
- Write the consent copy in 005 explicitly, rather than leaving it as a line in the selector row.

### 3.6 Minor — Custom server is saved without a check (005 / 006)

Saving makes the server the method in use without testing it. The first bad key or wrong model only shows up on someone's voice message.

**Recommendation:** a "Test" step on save: a short bundled clip plus a summary call, with the result shown in 004's status wording.

Also check that the provider examples work with **one address for both steps**. 006 needs `/audio/transcriptions` on the transcription host. OpenRouter is listed as an example, but as far as I know it doesn't offer that endpoint, so the "one address and one key cover both" default would fail there. Confirm this, and Cortecs's endpoint, before they ship as recommendations.

### 3.7 Minor — Error statuses that give the user nothing to do (008)

For Flim, _Setup is wrong_ ("The transcription settings look wrong") has nothing the user can fix, because Flim has no settings fields. Flim-side 400 / 404 / 422 errors should map to _Service fault_.

---

## 4. One-handed use

001 is thorough for its scope: the strip, matching animation into and out of the list row, keeping the draft, accessibility, nested spaces. Points to review:

- **Major — Scope covers leaving a room or space, not the rest of one-handed use.** The things a one-handed user needs most often all stay at the top of the screen: search, new chat (+), filter pills, the account / settings avatar, room details (tapping the header), call buttons, and reaching the first rows of a long list. For the goal "easy to use with one hand", a second decision should cover entry points. For example: search moves to the bottom (the iOS 26 tab bar already has a bottom search role), room details open from the strip or a bottom control, and new chat becomes a bottom action.
- **Major — Tab bar vs strip is unspecified.** On iOS 26 the floating tab bar sits where the strip goes. 001 and 003 don't say whether the room popup covers or hides the tab bar. State it: the tab bar is hidden while a room or space is open.
- **Minor — Large phones in landscape are "wide".** iPhone Plus / Pro Max models in landscape have a regular width size class, so rotating switches to upstream split navigation and the strip disappears. That's probably acceptable. Write it down so it isn't reported as a bug.
- **Minor — Small phones with the keyboard up.** The composer, app bars, a 54 pt strip and the keyboard together leave little timeline on an iPhone SE / mini-sized screen. 001 already allows the label to drop to icon-only. Check the smallest supported device early, as 001 already asks for the animation.
- **Minor — Accidental room closes while typing.** 001 handles this with spacing, and the draft is kept. Once 1.2 is fixed, the neighbour below the strip is the suggestion bar, where people tap often. Prototype it before locking.

---

## 5. Branding (summary of the above)

| Brand surface                                  | Covered by      | Status                                                                                             |
| ---------------------------------------------- | --------------- | -------------------------------------------------------------------------------------------------- |
| Display name, bundle / app-group ids           | 000a            | Covered; home-screen name too long (2.4)                                                           |
| Look: colours, components, light/dark          | 003             | Covered thoroughly                                                                                 |
| App icon, launch, store assets                 | none            | Missing (2.3)                                                                                      |
| OIDC consent logo / terms, website, help links | none            | Still Element (2.2)                                                                                |
| Push, analytics, bug reports                   | none            | Broken or still Element (2.1, 2.2)                                                                 |
| Fork-owned copy (en-GB + fi)                   | 002             | Covered; conflicts with workspace policy (1.4)                                                     |
| Event namespace                                | 004 (from 000a) | `me.flim`, a personal namespace. Fine if that domain stays yours; it's permanent once events exist |
| Android identity                               | none            | Deferred while 005–009 require Android (2.5)                                                       |

---

## 6. Suggested next steps (in order)

1. **Workspace guidance:** fix the strings rule in `AGENTS.md`, `ios/AGENTS.md` and `jev-pref.json` (1.4). No new decision needed.
2. **000a amendment, or a new 000b, for service endpoints:** push gateway, OIDC / MAS domain, privacy / terms / help links, analytics / Sentry / bug reports off, home-screen name "Flim" (2.1, 2.2, 2.4, 2.10).
3. **New branding decision:** app icon, launch screen, store assets (2.3).
4. **004 protocol amendment:** thread relation, transcript size cap, relations lookup before the wait, unread handling (1.1, 2.7, 2.8, 2.9).
5. **New transcription-behaviour decision, amending 004 / 005 / 009:**
   - an Off method, with the marker sent only when a method is in use (3.1);
   - recipients start a brief only when the voice message is shown (3.2);
   - a sharing setting and "Remove summary" (3.3);
   - the transcript collapsed by default (3.4);
   - a consent step and limits on room context (3.5);
   - the mock limited to tests and previews (1.7).
6. **Small amendments:**
   - Allowance wording per plan (1.3).
   - On-device "not ready" status (1.5).
   - A second key when there's a second address (1.6).
   - Flim error mapping (3.7).
   - Custom-server test on save (3.6).
   - Title and summary limits (1.9).
   - 005 History line for 009 (1.8).
7. **001 / 003 amendment:** keyboard suggestion bar stack (1.2), tab bar hidden in rooms (section 4), mock re-export (1.10).
8. **Before shipping Flim mode:** account deletion, privacy policy / terms, session lifetime, length and plan limits, and an App Review pass on promo codes and the disabled subscription button (2.6).
9. **Platform scope:** pick either Android on 000a–003 or Android deferred on 005–009 (2.5).
10. **Optional follow-on:** a one-handed entry-points decision (search, new chat, room details at the bottom) (section 4).

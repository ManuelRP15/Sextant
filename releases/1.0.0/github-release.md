The first public release. Sextant answers one question from the Salesforce page itself — what is this text, and what does it say in every other language? — and keeps track of what changed.

## Highlights

- **Hold-to-inspect hover.** Point at any label, field, button or picklist value in Lightning and see what metadata it is, its API name, and every language the org has configured. One answer or none: where a string cannot be identified with confidence, Sextant says *Unknown origin* instead of offering a list of guesses.
- **Edit in place** for Custom Labels, custom (`__c`) fields and custom picklist values, with every save re-reading the org's current value first so a colleague's more recent edit is never silently overwritten. When Salesforce refuses a write, you see the org's own message.
- **Translate All** annotates every translatable element on the page at once, filters to *Missing*, *Identical* or *Complete*, and steps through the issues with the page scrolling to each in turn.
- **Search** goes from a translated string — the one quoted in the ticket — back to the component it belongs to, across the org's metadata rather than only its translatable slice.
- **Workspace** captures every edit automatically, with before/after history, a `package.xml` export ready to deploy, and a portable export of the whole session. A component you kept is marked when it moves in the org after you captured it, naming the language that changed.
- **Activity** shows what has happened in the org, built from evidence rather than inference, and notices a deployment landing — including one this browser did not start.
- **Compare environments**: one component across several orgs as a table, with a consensus value and an exception set; two reads of the same org over time, always stating which two moments are compared.
- **Create Custom Labels** with their translations in one grid, checked against the org as you type.
- **Settings → Privacy & security.** See what Sextant keeps in your browser and how much space it uses, export it (never with your Salesforce session), delete it by scope, and read every permission Sextant asks for and why.
- Light, Dark and System themes; English and Spanish.

## Improvements

- **The index says how old it is** and how much of the org it covered, rather than implying freshness it cannot vouch for. Large orgs are read incrementally and the read resumes across browser restarts.
- **Saving straight to a production org always asks first.** *Don't ask me again* does not apply to a production org, or to an org Sextant has not identified yet.
- **Automatic interface language.** A new install follows the browser's language when Sextant has it; choosing a language in Settings keeps that choice.
- **Absence is distinguished from failure** throughout: "there is nothing here" and "we could not find out" are different sentences, never the same empty state.

## Fixes

- Activity no longer attributes a colleague's change to you because you had pinned the component.
- The Workspace's search box matches the label you see on the row, not only its API name.
- API names and object names no longer clip mid-word in rows that have room for them.
- The Workspace's row summaries follow the interface language.

## Privacy

- **No backend, no account, no telemetry, no analytics, no third-party requests.** The only host Sextant contacts is your own Salesforce org, using your own session, making the requests Setup already makes. Everything Sextant keeps stays in this browser profile.
- The privacy policy (version 3) covers the extension, the website and the interactive demo, in English and Spanish, and a generated `DATA_HANDLING.md` lists every item Sextant stores.

## Security

- **The browser enforces where Sextant can send requests.** A content security policy limits Sextant's pages and background process to Salesforce, and forbids scripts from anywhere else and string evaluation. It is verified in a real browser on every release build.
- Every request is checked against the org it belongs to before your session is attached; the session is never written to storage, logs, messages or exports.
- Messages to Sextant's background process are checked against their sender; imported files are size-limited and validated field by field; links built from org data are validated before they open.
- Developer diagnostics are off until you turn them on and strip anything shaped like a credential.

## Permissions

- `storage`, `unlimitedStorage`, `activeTab`, `cookies`, `alarms`, and access to `*.lightning.force.com`, `*.my.salesforce.com` and `*.salesforce.com`. What each one is for, and what it does not do, is in `PERMISSIONS.md`.
- The `tabs` permission — the one that makes Chrome say *read your browsing history* — is not requested. The toolbar menu names the org you are looking at through Sextant's Salesforce site access alone, which a browser test verifies on every release build.

## Compatibility

- Chrome or Edge (Chromium), Manifest V3. Lightning Experience only; Sextant does not run in Classic.
- **API Enabled** on your user, and *Setup → Session Settings → "Lock sessions to the domain in which they were first used"* disabled — with it on, the background process's session is rejected, which is the most common reason a fresh install appears not to connect.
- Translation Workbench enabled, with at least one language, for translation data to exist at all.

## Known issues

- Editing covers 9 of 13 metadata types, two of them for custom (`__c`) members only. Object labels and related lists are deferred; standard buttons and tabs are Salesforce's own translations and stay read-only.
- Hover surfaces objects, fields, picklist values and Custom Labels; the other types are read and indexed but kept out of the hover surface by design.
- A save through the Metadata API takes about a minute, almost all of it Salesforce's own deploy queue. Custom Labels use a faster path.
- The hover inspector and Translate All are shown in English; the rest of Sextant follows the interface language.
- Not validated against Professional Edition or orgs using API Access Control restrictions.

## Install

Not on the Chrome Web Store yet. The official install path is the store listing; nothing here is meant to be installed by hand.

## Read more

- [Changelog](https://usesextant.dev/changelog/) · [Privacy policy](https://usesextant.dev/privacy/) · [Security policy](https://usesextant.dev/security/) · [How a release is built](https://usesextant.dev/release-process/)

Sextant is an independent product. It is not made, endorsed or supported by Salesforce, Inc.

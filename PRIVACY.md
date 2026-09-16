# Privacy Policy — Sextant

**Version 3 · Last updated: 2026-09-16** · [Versión en español](PRIVACY.es.md) (the English
text governs)

> **⚠️ NOT YET PUBLISHABLE — three open items, each checked by `npm run publication:check`:**
>
> 1. **LEGAL REVIEW REQUIRED.** Written by the developer, verified against the code, and not reviewed by
>    a lawyer. Sections marked *legal review* in particular must be reviewed before publication.
> 2. **DECISION REQUIRED — who publishes Sextant.** §0 names the publisher. Who that is depends on the
>    unresolved ownership question and must not be guessed.
> 3. **DECISION REQUIRED — contact.** §14 needs a channel that is actually read (the same decision as
>    `SECURITY.md` §1).
>
> The Chrome Web Store also requires this text at a public URL; the Trust Center publishes it once
> these items are closed.

---

## 0. Who this policy is about

Sextant is a browser extension for Chrome and Edge that shows what Salesforce metadata a piece of text
is, with its translations, and lets you edit those translations. It is published by
`[ PUBLISHER — DECISION REQUIRED ]` ("the developer", "we").

Sextant is an independent product. It is not made, endorsed or supported by Salesforce, Inc.
Salesforce and Lightning are trademarks of Salesforce, Inc.

## Summary

Sextant has no server and no account. It works inside your browser, with the Salesforce session you
already have, and talks to your Salesforce org and nothing else.

These six statements are the same ones Sextant shows in Settings → Privacy & security. Each is backed
by the browser's own enforcement, by automated tests or by a check on every release build;
[`DATA_HANDLING.md`](DATA_HANDLING.md) says which, what each one does not mean, and where to verify it.

- Sextant sends requests only to Salesforce, and the browser blocks its pages from contacting any other site.
- No analytics, telemetry, crash reporting or advertising.
- There is no Sextant account or server. Nothing you do in Sextant is sent to its developer.
- Everything Sextant runs ships inside the installed extension. It downloads no code.
- Your Salesforce session is read when a request needs it and is never stored, logged or exported by Sextant.
- What Sextant keeps stays in this browser profile. You can see it, export it and delete it in Settings.

## 1. What Sextant reads from Salesforce

To do its job, Sextant reads from the Salesforce org you are signed in to, through Salesforce's own
APIs, **as you** — so it can read only what your Salesforce user is allowed to read:

- **Translatable metadata and its translations**: custom labels, object and field labels, picklist
  values, record types, buttons and links, quick actions, tabs, apps, flows and page layout sections;
  their API names and ids; and the languages the org has enabled.
- **The org's component lists**, including when each component was last modified and the **name of
  the Salesforce user** who last modified it, as Salesforce reports it.
- **Org and user identity**: the org's id, name, instance and whether it is a sandbox or production;
  your user id, username and name. Sextant uses these to keep each org's data apart and to show you
  which org you are working in.
- **Recent deployment records** (their ids, status and dates, and the components they contained), so it
  can tell you when a deployment has landed.

Sextant does not read your business records — accounts, contacts, opportunities, cases and so on —
through Salesforce's APIs. Because it runs inside Lightning pages, it can see what those pages display;
it reads label text there to identify it, inside your browser, and sends nothing from the page anywhere.

## 2. Your Salesforce session

Sextant's background process reads your org's `sid` session cookie from your browser when it needs to
call your org's API as you.

- It is read only for your org's Salesforce API address.
- It is never stored, logged or exported by Sextant, and is sent only to the org it belongs to.
- You are never asked for a Salesforce password, and never should be.

Signing out of Salesforce ends the session; Sextant then shows the org as signed out.

## 3. What Sextant stores, and where

Sextant keeps its data in the browser's storage for this extension (`chrome.storage.local`), in your
browser profile, on your computer. It is not synced to your Google account and not sent anywhere.

It keeps: your settings; the orgs you have used; a cache of the metadata it has read; the history of
changes it has observed in your orgs and the translation edits you made; translation snapshots for
comparisons; the translation changes you queued or deployed; your Workspace; and developer diagnostics,
only if you turn them on. Some of this includes names of Salesforce users in your orgs (for example, who
last modified a component).

Every stored item, why it is kept and for how long is listed in [`DATA_HANDLING.md`](DATA_HANDLING.md)
§3, which is generated from the code. Sextant does not encrypt this data beyond what your operating
system and browser provide; anyone who can use your browser profile can read it.

## 4. What Sextant sends, and to whom

**To your Salesforce org:** the requests described in §1, and — only when you ask — the translation
changes you save or deploy (§5). They go to your org's own API address.

**To the developer:** nothing. There is no Sextant server, account, analytics, telemetry or crash
reporting. We do not receive, see or hold the data Sextant handles in your browser.

**To anyone else:** nothing. Sextant loads no fonts, scripts, styles or images from other sites, has no
advertising, and does not sell, rent or share data.

**The Chrome Web Store and your browser.** Google distributes Sextant and its updates through the
Chrome Web Store, under Google's own terms, and — like every publisher — the developer can see the
aggregate statistics the store provides, such as the number of users. Your browser contacts its maker
for its own purposes, such as checking for updates. Sextant sends nothing to either.

## 5. What Sextant writes to your org

When you save or deploy a translation, Sextant writes it to your Salesforce org through Salesforce's
APIs — the same change you could make in Setup.

- It writes only when you ask.
- It re-reads the current value first and refuses to overwrite a change someone made after you
  started editing.
- Saving a change straight to a production org, or to an org Sextant cannot identify, asks you first.
- The write is made **as your Salesforce user**, with your permissions. Salesforce records metadata
  deployments against your user. Salesforce's *Setup Audit Trail* does **not** record translation
  changes — whether made with Sextant or in Setup's own Translation Workbench — which is why Sextant
  keeps its own local record of the changes you make with it.

## 6. Artificial intelligence

**Sextant contains no AI feature and sends nothing to any AI provider.**

Sextant's source code contains unfinished groundwork for a possible AI capability. It is excluded from
the extension you install, and the extension's security policy would block such a request in any case.
If that ever changes, this policy will say — before the capability ships — exactly what would be sent,
to whose service, and under whose key.

## 7. Files you create

Sextant can export files at your request: an export of its local data, Activity history, the Workspace,
reports and `package.xml` files. They are saved where you choose and are yours to handle. They may
contain org metadata, translation text and names of people in your orgs. An export of Sextant's local
data never contains your Salesforce session or any other credential.

Sextant can also import a Workspace or Activity file you choose. It is read locally and checked before
anything from it is kept.

## 8. If you contact us

If you write to us — for support or to report a security problem — we receive what you send, such as
your email address and your message, and use it only to answer you and to fix the problem you reported.
Please never send a Salesforce session id, password or real org data (see `SECURITY.md` §2).

> **⚠️ LEGAL REVIEW REQUIRED** — how long correspondence is kept, and on what legal basis, depends on
> the publisher and the channel chosen (§0, §14).

## 9. Your choices and rights

> **⚠️ LEGAL REVIEW REQUIRED** — this section describes rights under data-protection law (such as the
> EU and UK GDPR) and must be reviewed before publication.

- **See, export and delete** what Sextant keeps: Settings → Privacy & security. Deleting there removes
  Sextant's data from your browser and never changes anything in Salesforce.
- **Remove everything**: uninstall Sextant; the browser deletes its storage.
- **Stop Sextant reading an org**: sign out of that org, or remove the extension's access to Salesforce
  sites in your browser's extension settings.

Because the data Sextant handles stays in your browser and is not sent to us, we hold no copy of it and
cannot access, correct or delete it for you — you can do each of those directly, as above. For anything
you have sent us yourself (§8), you can ask us to access, correct or delete it using the contact in §14.

If you use Sextant in an organisation's Salesforce org, that organisation may have its own rules about
the data in it, including the names of users that Sextant shows and stores locally.

## 10. Children

Sextant is a professional tool for Salesforce administrators and developers. It is not directed at
children.

## 11. Security

How Sextant protects what it handles, what it cannot protect against, and how to report a problem are
in [`SECURITY.md`](SECURITY.md). No software is free of defects; Sextant's approach is to keep what it
can reach small, check it automatically, and say plainly where the limits are.

## 12. The website and the interactive demo

Sextant's website (`https://usesextant.dev`) and the interactive demo it hosts (`/demo/`) are static pages.

- **The website** sets no cookies, runs no script, has no analytics, and loads no font, style, script or
  image from any other site. As with any website, the hosting provider receives each request — the page
  requested, your IP address and your browser's identification — under its own terms.
- **The demo** runs Sextant's own code in your browser against a fictional Salesforce org. It sends nothing
  anywhere: its content security policy allows no request except for its own files, and it keeps nothing in
  your browser's storage, so closing the tab discards everything you did in it.
- Neither the website nor the demo connects to any Salesforce org, and nothing you do in the demo reaches
  the developer.

> **⚠️ LEGAL REVIEW REQUIRED** — the hosting provider's data-processing terms and access-log retention
> (`docs/HOSTING.md` §8) are the provider's; whether this policy must name the provider is a question for
> counsel.

## 13. Changes to this policy

Each version is dated and numbered. A change to what Sextant collects, stores or sends will be made here
**before** the release that introduces it, and described in that release's notes and store listing.

| Version | Date | Change |
|---|---|---|
| 3 | 2026-09-16 | Added §12: the website and the interactive demo — no cookies, no script, no analytics on the site; the demo sends nothing and keeps nothing — and this policy's public address. Section numbers 12 and 13 became 13 and 14. |
| 2 | 2026-09-15 | Corrected two statements that were not accurate: the browser's enforcement of Sextant's network boundary comes from its content security policy, not its host permissions; and translation changes do not appear in Salesforce's Setup Audit Trail. Added names of Salesforce users, deployment records, imports, the Chrome Web Store's statistics and your rights. |
| 1 | 2026-09-01 | First version. |

## 14. Contact

<!-- DECISION REQUIRED: the same channel as SECURITY.md §1. -->
`[ PRIVACY CONTACT — NOT YET CONFIGURED ]`

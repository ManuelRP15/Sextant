# Sextant permissions

> **Generated — do not edit by hand.** Rendered from `src/shared/permission-explanations.ts` by
> `npm run docs:data-handling`. A test fails when this file and the code disagree, and another fails when the
> manifest asks for a permission this table does not explain.

Every permission and host permission in the extension's manifest, what it is for, where the code uses it,
what it does not do, what stops working without it, the install-time warning it contributes to, and whether
it could be narrower. The same entries are shown in Settings → Privacy & security and on the Trust Center,
and are the text of the Chrome Web Store's permission justifications.

## Summary

| Permission | Kind | Why | Install warning |
|---|---|---|---|
| `storage` | Chrome API | Keeps your settings, the orgs you use, what Sextant has read from them, your Workspace and your history in this browser profile. | None |
| `unlimitedStorage` | Chrome API | A large org's metadata can exceed the browser's default storage limit for an extension. | None |
| `activeTab` | Chrome API | Lets Sextant's toolbar menu read the address of the tab you opened it on, so it can name the Salesforce org you are looking at. | None |
| `cookies` | Chrome API | Reads your Salesforce session cookie for your org's API address when Sextant calls Salesforce as you. | None |
| `alarms` | Chrome API | Resumes a long read of a large org after the browser pauses Sextant's background process. | None |
| `https://*.lightning.force.com/*` | Host access | Salesforce Lightning pages — the only pages where Sextant shows what a label is and edits translations in place. | Read and change your data on the listed Salesforce sites |
| `https://*.my.salesforce.com/*` | Host access | Your org's API address. Sextant calls Salesforce's own APIs there, as you, with the session you already have. | Read and change your data on the listed Salesforce sites |
| `https://*.salesforce.com/*` | Host access | Salesforce addresses outside my.salesforce.com, such as older instance addresses. | Read and change your data on the listed Salesforce sites |

## Each permission

### `storage`

**Why:** Keeps your settings, the orgs you use, what Sextant has read from them, your Workspace and your history in this browser profile.

**What it does not do:** Does not sync anything to your Google account and does not send what it keeps anywhere.

**Without it:** Sextant would remember nothing and re-read every org on every page.

**Install warning it contributes to (paraphrased):** None.

**Used by:**

- `src/shared/data-inventory.ts`
- `src/background/index.ts`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `unlimitedStorage`

**Why:** A large org's metadata can exceed the browser's default storage limit for an extension.

**What it does not do:** Does not give Sextant access to any storage outside its own, or to other sites' data.

**Without it:** On a large org the saved index fails to write, and Sextant cannot finish reading the org.

**Install warning it contributes to (paraphrased):** None.

**Used by:**

- `src/shared/storage-budget.ts`
- `src/shared/budgets.ts`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `activeTab`

**Why:** Lets Sextant's toolbar menu read the address of the tab you opened it on, so it can name the Salesforce org you are looking at.

**What it does not do:** Grants nothing for other tabs or later visits, runs no code in the page, and adds no install-time warning. Sextant does not ask for the broader `tabs` permission, which Chrome describes as reading your browsing history.

**Without it:** The toolbar menu could not tell which org a Salesforce Setup page belongs to, and would treat it as a page outside Salesforce.

**Install warning it contributes to (paraphrased):** None.

**Used by:**

- `src/popup/Popup.tsx`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `cookies`

**Why:** Reads your Salesforce session cookie for your org's API address when Sextant calls Salesforce as you.

**What it does not do:** Reads one cookie, only for Salesforce addresses, and never stores, logs or exports it. It cannot read cookies of any other site.

**Without it:** Sextant cannot read from or write to Salesforce at all.

**Install warning it contributes to (paraphrased):** None.

**Used by:**

- `src/background/index.ts`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `alarms`

**Why:** Resumes a long read of a large org after the browser pauses Sextant's background process.

**What it does not do:** Does not run while the browser is closed and schedules nothing outside Sextant.

**Without it:** A full read of a large org would stop every time the browser paused the worker.

**Install warning it contributes to (paraphrased):** None.

**Used by:**

- `src/background/index.ts`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `https://*.lightning.force.com/*`

**Why:** Salesforce Lightning pages — the only pages where Sextant shows what a label is and edits translations in place.

**What it does not do:** Sextant reads label text on these pages, inside your browser, to identify it. It does not send page content anywhere.

**Without it:** The hover inspector and Translation Mode could not run.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites

**Used by:**

- `manifest.config.ts`
- `src/content/index.tsx`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `https://*.my.salesforce.com/*`

**Why:** Your org's API address. Sextant calls Salesforce's own APIs there, as you, with the session you already have.

**What it does not do:** Each request goes only to the org whose session it uses.

**Without it:** Sextant could not read metadata or save translations.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites

**Used by:**

- `src/shared/salesforce-transport.ts`
- `src/background/index.ts`

**Could it be narrower?** No — it is as narrow as the feature allows.

### `https://*.salesforce.com/*`

**Why:** Salesforce addresses outside my.salesforce.com, such as older instance addresses.

**What it does not do:** It does not widen what Sextant sends or reads: every request still goes only to the API address of the org whose session it uses.

**Without it:** An org still served from an older instance address could stop working. No such org has been confirmed.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites

**Used by:**

- `src/shared/salesforce-origin.ts`
- `manifest.config.ts`

**Could it be narrower?** No confirmed use: Sextant builds my.salesforce.com API addresses. Narrow it (with the matching CSP entry) only after checking an org without enhanced domains (ROADMAP P46-PM4).

## What is deliberately absent

- No `optional_permissions` and no `optional_host_permissions`: everything Sextant needs is declared up front.
- No `externally_connectable`: no web page or other extension can message Sextant.
- No `web_accessible_resources` of Sextant's own; the bundler generates one entry for the chunks the content
  script loads, restricted to Lightning pages (`manifest.config.ts` explains why the hand-written one was removed).
- No `scripting`, `webRequest`, `webNavigation`, `history`, `bookmarks`, `identity` or `management`.

Changing any of this is a Trust Architecture Change (`TRUST_ARCHITECTURE_CHANGES.md`).

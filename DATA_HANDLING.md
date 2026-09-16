# How Sextant handles data

> **Generated — do not edit by hand.** Rendered from `src/shared/trust-facts.ts`, `permission-explanations.ts`,
> `data-inventory.ts` and `local-data.ts` by `npm run docs:data-handling`. A test fails when this file and
> the code disagree, so what follows describes the code as it is, not as someone remembered it.

This is the precise, checkable companion to the [privacy policy](PRIVACY.md). It is written for
anyone who wants to verify what Sextant does rather than take it on trust: a security reviewer, an
administrator deciding whether to allow the extension, or a user who wants the details.

## 1. What Sextant says about itself, and how each statement is known

### Sextant sends requests only to Salesforce, and the browser blocks its pages from contacting any other site.

**How it is known:** Enforced by the browser; Checked by automated tests; Checked in every release build.

**What it does not mean:** The browser enforces this for Sextant's own pages and its service worker. The part of Sextant that runs inside the Salesforce page sends nothing to any site itself — it asks the service worker — and that is checked by tests, not enforced by the browser. The browser policy allows any host under salesforce.com; the code sends each request only to the API host of the org whose session it uses. It does not describe what Salesforce, the browser or other extensions do.

**Where to check:**

- manifest.config.ts — content_security_policy.extension_pages, connect-src 'self' https://*.salesforce.com
- security-qa/csp-enforcement.pw.ts — loads the built extension; requests to other hosts never reach the network
- src/shared/salesforce-transport.ts — the only function that performs a request; destination checked per request
- src/security/network-boundary.test.ts — fails if any other module makes a request

### No analytics, telemetry, crash reporting or advertising.

**How it is known:** Enforced by the browser; Checked by automated tests; Checked in every release build.

**What it does not mean:** The Chrome Web Store shows every publisher aggregate statistics such as install counts, and Chrome itself contacts Google (for example to check for updates). Sextant sends nothing to either.

**Where to check:**

- scripts/release/scan-dist.mjs — fails a release whose bundle contains an analytics or telemetry endpoint or SDK
- src/security/network-boundary.test.ts — no request primitive outside the Salesforce transport
- manifest.config.ts — the CSP leaves no destination for such data

### There is no Sextant account or server. Nothing you do in Sextant is sent to its developer.

**How it is known:** Enforced by the browser; Part of the release process.

**What it does not mean:** If you contact the developer (for support or a security report), what you send is received like any message.

**Where to check:**

- manifest.config.ts — no host a Sextant server could live on is reachable
- docs/security/RELEASE_SECURITY.md — adding a backend is a Trust Architecture Change requiring a policy update first

### Everything Sextant runs ships inside the installed extension. It downloads no code.

**How it is known:** Enforced by the browser; Checked by automated tests; Checked in every release build.

**What it does not mean:** Updates arrive through the Chrome Web Store as a new version of the whole package, and the release process records the SHA-256 of every package it builds. A trusted update could still change what Sextant does; that is why every change to these statements must ship with a matching change to this list.

**Where to check:**

- Manifest V3 and manifest.config.ts — script-src 'self', no 'unsafe-eval'
- security-qa/csp-enforcement.pw.ts — string evaluation is refused in an extension page
- src/security/code-boundary.test.ts — no eval, Function constructor or HTML injection in the source

### Your Salesforce session is read when a request needs it and is never stored, logged or exported by Sextant.

**How it is known:** Checked by automated tests.

**What it does not mean:** Sextant uses the session you already have in your browser; signing out of Salesforce ends it.

**Where to check:**

- src/background/index.ts — getSessionId: read from the cookie store per request, never written
- src/security/session-boundary.test.ts — the session value never reaches storage, logs, messages or exports

### What Sextant keeps stays in this browser profile. You can see it, export it and delete it in Settings.

**How it is known:** Checked by automated tests.

**What it does not mean:** It is not encrypted by Sextant beyond what your operating system and browser profile provide. Anyone with access to this browser profile can read it.

**Where to check:**

- src/shared/data-inventory.ts — every stored key, classified; chrome.storage.local only (no sync)
- src/security/data-inventory.test.ts — fails when code stores a key the inventory does not describe

## 2. Permissions

Every permission in the extension's manifest, what it is for, and what it does not do. A test fails if the
manifest gains or loses a permission without this list changing with it.

### `storage`

Keeps your settings, the orgs you use, what Sextant has read from them, your Workspace and your history in this browser profile.

**Does not:** Does not sync anything to your Google account and does not send what it keeps anywhere.

**Without it:** Sextant would remember nothing and re-read every org on every page.

### `unlimitedStorage`

A large org's metadata can exceed the browser's default storage limit for an extension.

**Does not:** Does not give Sextant access to any storage outside its own, or to other sites' data.

**Without it:** On a large org the saved index fails to write, and Sextant cannot finish reading the org.

### `activeTab`

Lets Sextant's toolbar menu read the address of the tab you opened it on, so it can name the Salesforce org you are looking at.

**Does not:** Grants nothing for other tabs or later visits, runs no code in the page, and adds no install-time warning. Sextant does not ask for the broader `tabs` permission, which Chrome describes as reading your browsing history.

**Without it:** The toolbar menu could not tell which org a Salesforce Setup page belongs to, and would treat it as a page outside Salesforce.

### `cookies`

Reads your Salesforce session cookie for your org's API address when Sextant calls Salesforce as you.

**Does not:** Reads one cookie, only for Salesforce addresses, and never stores, logs or exports it. It cannot read cookies of any other site.

**Without it:** Sextant cannot read from or write to Salesforce at all.

### `alarms`

Resumes a long read of a large org after the browser pauses Sextant's background process.

**Does not:** Does not run while the browser is closed and schedules nothing outside Sextant.

**Without it:** A full read of a large org would stop every time the browser paused the worker.

### `https://*.lightning.force.com/*`

Salesforce Lightning pages — the only pages where Sextant shows what a label is and edits translations in place.

**Does not:** Sextant reads label text on these pages, inside your browser, to identify it. It does not send page content anywhere.

**Without it:** The hover inspector and Translation Mode could not run.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites.

### `https://*.my.salesforce.com/*`

Your org's API address. Sextant calls Salesforce's own APIs there, as you, with the session you already have.

**Does not:** Each request goes only to the org whose session it uses.

**Without it:** Sextant could not read metadata or save translations.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites.

### `https://*.salesforce.com/*`

Salesforce addresses outside my.salesforce.com, such as older instance addresses.

**Does not:** It does not widen what Sextant sends or reads: every request still goes only to the API address of the org whose session it uses.

**Without it:** An org still served from an older instance address could stop working. No such org has been confirmed.

**Install warning it contributes to (paraphrased):** Read and change your data on the listed Salesforce sites.

**Under review:** No confirmed use: Sextant builds my.salesforce.com API addresses. Narrow it (with the matching CSP entry) only after checking an org without enhanced domains (ROADMAP P46-PM4).

## 3. What Sextant stores

Everything is in `chrome.storage.local` — the browser's storage for this extension, in this browser profile,
on this computer. Sextant uses no other storage: no sync storage (nothing reaches a Google account), no
cookies of its own, no web storage, no IndexedDB, no files of its own. Nothing listed here is sent anywhere by Sextant.
It stays when the browser restarts and when you sign out of Salesforce. Uninstalling the extension deletes all
of it; Settings → Privacy & security shows its size and deletes it selectively.

Sextant does not encrypt this data beyond what the operating system and the browser profile provide.

### Settings and preferences

Your Sextant preferences: languages you work in, theme, display size, shortcuts, Activity view options, and the actors you marked as automated users.

- **Why:** To keep Sextant configured the way you left it.
- **How long:** Until you change or reset them.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `settings` | Preferences object (`Settings` in `types.ts`). | Set by you | No | No | Yes |
| `auditViewPrefs` | Activity view options: sort, columns, density. | Set by you | No | No | Yes |
| `auditSystemActorOverrides` | Actors (Salesforce user names) you marked as automated, per org. | Set by you | Yes | No | Yes |
| `privacyDeletionNotice` | A one-time note that local data was just deleted, removed on the next start so Settings can say what happened. | Sextant's own bookkeeping | No | No | **Never** |

### Orgs you have used

For each Salesforce org Sextant has read: its org id, name, environment and My Domain address; your user id, username and name in it; when you last used it; and which org is active.

- **Why:** To keep every org's data apart, to show which org you are working in, and to switch between orgs.
- **How long:** Until you delete it. Updated each time you open that org.
- **If deleted:** Sextant can read it again from Salesforce.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `orgDirectory` | Org id → identity (org name, environment, host, your user id, username and name), first/last seen, last indexed. | Read from Salesforce | Yes | Yes | Yes |
| `activeOrg` | The org Sextant is pointed at: org id, Lightning origin, since when. | Set by you | No | No | Yes |
| `orgIdentities` | API host → identity, for the content script's provenance footer. | Read from Salesforce | Yes | Yes | Yes |
| `lastOrgOrigin` | The Lightning origin of the last Salesforce page read. | Sextant's own bookkeeping | No | No | Yes |

### Org metadata cache

What Sextant has read from each org so it can answer without asking again: component API names and ids, labels and translations in your languages, the org's component list with last-modified dates and names, which APIs the org exposes, read progress and timing.

- **Why:** To answer hover, search and Activity instantly, and to avoid re-reading a large org on every page.
- **How long:** Replaced on each read and kept until deleted. Sextant reads it again when needed.
- **If deleted:** Sextant can read it again from Salesforce.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `orgIndex::…` | The org's translation index: every translatable component's API name, type, id and values per language. | Read from Salesforce | No | Yes | Yes |
| `metadataAudit::…` | The org's component list (`listMetadata` rows) with last-modified date and last-modified-by name. | Read from Salesforce | Yes | Yes | Yes |
| `orgCapabilities::…` | Which Tooling API objects the org and your permissions expose. | Read from Salesforce | No | No | Yes |
| `indexSweepSlice::…` | One wave of an in-progress full-org read. | Read from Salesforce | No | Yes | Yes |
| `indexSweep::…` | Checkpoint of an in-progress full-org read. | Sextant's own bookkeeping | No | No | Yes |
| `indexStatus` | Per API host: read state, counts and times. | Sextant's own bookkeeping | No | No | Yes |
| `pageRoundLedger` | Per host: the last page read's org, languages and objects, to avoid re-reading within minutes. | Sextant's own bookkeeping | No | Yes | Yes |
| `autoIndexAttempts` | Per org: when Sextant last started a full read by itself. | Sextant's own bookkeeping | No | No | Yes |
| `clockSkew` | Per org: measured difference between this computer's clock and Salesforce's. | Read from Salesforce | No | No | Yes |

### Activity history and change baselines

What Sextant observed changing in each org over time — which component, when, and who Salesforce reports as its last modifier — your own translation edits, and the baselines used to notice the next change.

- **Why:** To show what changed and what was yours. Salesforce does not keep this history, so Sextant does.
- **How long:** Up to one year and 6,000 events per org; older events are removed automatically. Baselines are replaced as the org is read.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `activityEvents` | Per org: observed changes (component, time, reported modifier) and your own translation edits. | Salesforce and you | Yes | Yes | Yes |
| `activityEventsMigratedAt` | When the pre-event-log history was migrated. | Sextant's own bookkeeping | No | No | Yes |
| `renameTokenEventPurgeAt` | When a one-time correction of earlier events ran. | Sextant's own bookkeeping | No | No | Yes |
| `auditObservations` | Per org: the component baseline change detection compares against. | Read from Salesforce | Yes | Yes | Yes |
| `translationValueBaseline` | Per org: the translation values value-change detection compares against. | Read from Salesforce | No | Yes | Yes |

### Translation snapshots

Complete reads of an org's translations, kept so two moments or two orgs can be compared.

- **Why:** To answer whether translations arrived, and what changed between two reads.
- **How long:** The most recent snapshots per org; older ones are replaced.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `translationSnapshots::…` | The org's complete translation reads, for comparisons. | Read from Salesforce | No | Yes | Yes |

### Deployments

Translation changes you queued or sent — including the values being written — their outcome, deploy notices, and the component lists of deployments Sextant noticed in an org.

- **Why:** To track a deployment until Salesforce finishes it, even if the browser restarts, and to report what a deployment contained.
- **How long:** Kept until deleted. A deployment still in progress cannot be deleted until it finishes.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `translationDeploys` | Per org: deploy batches — edits with their values, status, Salesforce's answer. | Salesforce and you | No | Yes | Yes |
| `deployWatch` | Per org: deployments noticed and offers dismissed. | Read from Salesforce | No | No | Yes |
| `deployManifests::…` | The component lists of deployments noticed in the org. | Read from Salesforce | No | Yes | Yes |

### Workspace

Components you kept, your edits with their before and after values, review marks and the groups you named.

- **Why:** Your working set, and a package.xml of what you touched.
- **How long:** Until you remove items or delete the Workspace.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `workspaceItems` | Pinned components and captured edits with before/after values, per org. | Set by you | No | Yes | Yes |
| `workspaceReviewed` | Review marks, per org. | Set by you | No | No | Yes |
| `workspaceComponents` | Components kept in the Workspace, per org. | Set by you | No | Yes | Yes |
| `workspaceGroups` | Groups you named, per org. | Set by you | Yes | No | Yes |

### Diagnostics

Developer diagnostics, only when turned on: detection traces (element text, page address and how Sextant identified it) and the timing record started from the console.

- **Why:** To investigate a detection or performance problem you chose to capture.
- **How long:** Bounded: the newest traces and timing marks replace the oldest. Until cleared.
- **If deleted:** It is gone for good; Salesforce does not keep a copy Sextant could read back.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `detectionDiagnostics` | Detection traces you captured: element text, page URL, identification steps. | Salesforce and you | No | Yes | Yes |
| `__stiDiag` | Timing marks recorded after `stiDiagReset()`; details redacted of credentials. | Sextant's own bookkeeping | No | Yes | Yes |
| `__stiDiagEnabled` | Whether timing marks are being recorded. | Set by you | No | No | Yes |
| `__stiDiagBytes` | Whether timing marks include payload sizes. | Set by you | No | No | Yes |

### AI development data

Present only in development builds that include the unreleased AI kernel: a derived org vocabulary, cache counters, spend records and, if a developer configured one, a provider key. The store build contains none of it and deletes these keys when it starts.

- **Why:** Development of an unreleased capability.
- **How long:** Deleted automatically by builds without the AI kernel.
- **If deleted:** Sextant can read it again from Salesforce.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `aiOrgVocabulary` | A derived list of one org's object and label names. | Read from Salesforce | No | Yes | Yes |
| `aiIndexEpoch` | A counter keying the AI cache. | Sextant's own bookkeeping | No | No | Yes |
| `aiPackCache` | Cached context packs. | Read from Salesforce | No | Yes | Yes |
| `aiVerdicts.…` | Cached AI verdicts. | Sextant's own bookkeeping | No | Yes | Yes |
| `aiBudget` | Spend records. | Sextant's own bookkeeping | No | No | Yes |
| `aiTaskRuns` | Task run records. | Sextant's own bookkeeping | No | Yes | Yes |
| `aiProviderKey` | A provider API key a developer configured. A credential: never exported. | Set by you | No | No | **Never** |
| `aiProviderEndpoint` | A provider endpoint a developer configured. | Set by you | No | No | **Never** |

### Data from older versions

Keys written by earlier versions of Sextant before their data moved to its current place. Read once for migration.

- **Why:** Migration only.
- **How long:** Until migrated or deleted.
- **If deleted:** Sextant can read it again from Salesforce.

| Key | Holds | Source | Names of people | Org content | In an export |
|---|---|---|---|---|---|
| `cachedEntries` | The pre-`#206` single translation index. | Read from Salesforce | No | Yes | Yes |
| `orgIdentity` | The pre-`#206` single org identity. | Read from Salesforce | Yes | Yes | Yes |
| `translationHealth` | The removed Translation Health feature's data. | Read from Salesforce | No | Yes | Yes |
| `workspaceEdits` | The pre-v2 Workspace edits list. | Set by you | No | Yes | Yes |

## 4. Exporting your data

Settings → Privacy & security → *Export my Sextant data* saves a JSON file (`sextant.local-data-export`,
version 1). The file carries this notice:

> A copy of the data Sextant keeps in this browser. It never contains your Salesforce session or any credential. It may contain org metadata, translation text and names of people in your orgs: share it only as you would share that.

Never exported: `privacyDeletionNotice`, `aiProviderKey`, `aiProviderEndpoint`, and any key this version of Sextant does not recognise. The file lists
by name what it left out. Credential-shaped text found inside an exported value (a session id, a bearer
token, an API key) is replaced with `[redacted]`.

## 5. Deleting your data

Settings → Privacy & security. Every deletion asks first, removes only Sextant's data in this browser, and
never changes anything in Salesforce.

- **Clear cached org data.** Sextant reads your orgs again the next time you use them. A large org can take a few minutes. Removes: Org metadata cache; AI development data; Data from older versions.
- **Delete history.** Activity history, change baselines, snapshots and deployment records. Salesforce cannot give this history back — export Activity first to keep a copy. Removes: Activity history and change baselines; Translation snapshots; Deployments.
- **Delete the Workspace.** Components you kept, your edits and your groups. Export the Workspace first to keep a copy. Removes: Workspace.
- **Delete diagnostics.** Detection traces and timing records. Removes: Diagnostics.
- **Reset settings.** Your preferences return to their defaults. Removes: Settings and preferences.
- **Delete one org's data.** Everything Sextant keeps about the org you choose: its cache, history, snapshots, deployment records and Workspace items. Other orgs are not touched.
- **Delete all Sextant data.** Everything listed above, for every org. Sextant starts again as if it had just been installed.

A deletion that would remove the record of a deployment Salesforce is still running is refused until it finishes: A deployment is still running. Wait until Salesforce finishes it, then delete — its record is how Sextant reports the outcome.

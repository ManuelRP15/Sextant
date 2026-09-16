# How a Sextant release is built, checked and published

**Version 1 · 2026-09-15** (`DECISIONS.md #320`, `ROADMAP.md` `P46-BR1` … `P46-BR8`, `P46-UP1` … `P46-UP5`,
`P46-TM5`)

An installed extension updates itself. Whoever can publish a Sextant version runs code with Sextant's
permissions in every browser that has it (`THREAT_MODEL.md` T9). This document is how that power is
used: one way to build a release, the checks that must pass, the human decisions that cannot be
automated, and what a published hash does and does not prove.

**Nothing in this repository uploads, publishes, tags or pushes anything.** Every outward step is a
human action, listed here as such.

---

## 1. The one way to build a release

```bash
npm run release:dry-run      # rehearse on the working tree; changes nothing, publishes nothing
npm run release              # a release: requires a clean tree at tag v<version>
```

`scripts/release/release.mjs` is the only supported path. A zip assembled by hand from a `dist/` folder is
not a release, whatever it contains. The script writes everything it did to `release/<version>/`
(ignored by git) and, for a real release, copies the metadata to `release-metadata/releases/<version>/`
for review and commit.

### The gates, in order

The first failure stops the release. Every gate's result is recorded in `release-report.json`.

| # | Gate | What it proves | A real release |
|---|---|---|---|
| 1 | **Preflight** | Version agreement; no AI kernel in the environment | Refuses a dirty tree or a commit without the tag `v<version>` |
| 2 | **Publication readiness** | No open legal, ownership or operational blocker (`npm run publication:check`) | Stops on any blocker; a rehearsal only records them |
| 3 | **Clean checkout** | The build uses only what is committed | `git worktree` at the commit, then `npm ci` with install scripts limited to the reviewed list |
| 4 | **Typecheck** | `tsc --noEmit` is clean | — |
| 5 | **Tests** | The full suite, including the security suite in `src/security/` | — |
| 6 | **Repository leak scan** | The would-be public tree contains no client or personal identifiers, credentials or internal references | Requires the private identity list |
| 7 | **Build** | `SEXTANT_RELEASE=1 vite build`; the bundle inventory is recorded from the module graph | — |
| 8 | **Reproducibility** | A second build of the same checkout is byte-identical, file by file | — |
| 9 | **Artifact scan** | The built package matches the approved manifest, CSP and URL origins; contains no remote code, telemetry, AI provider code, credentials, client identifiers, source maps, local paths or developer affordances (`scripts/release/scan-dist.mjs`) | Requires the client-identity scan |
| 10 | **Browser enforcement** | The built package, loaded into a real browser, cannot reach a non-Salesforce host or evaluate a string (`security-qa/csp-enforcement.pw.ts`) | Cannot be skipped |
| 11 | **SBOM** | CycloneDX 1.5 of the third-party code actually in the bundle, each with its lockfile integrity | — |
| 12 | **Package** | A deterministic zip (sorted entries, the commit's timestamp, independent of time zone) and the SHA-256 of it and of every file | — |
| 13 | **Trust diff** | Permissions, host permissions, CSP, URL origins, bundled packages, install scripts and security-boundary files, against the approved baseline and the previous release (§4) | A security-sensitive result requires `--approve-trust-change "<reason>"` |
| 14 | **Changelog and release notes** | `CHANGELOG.md` has a `## [<version>]` section and is exactly what `release-metadata/releases/*/release.md` generates (§7) | Required; the version's description must carry its date |
| 15 | **Provenance** | `provenance.json`: version, commit, tag, timestamps, Node and npm versions, lockfile hash, artifact hash, manifest permissions, CSP, URL origins, bundled packages, boundary-file hashes, SBOM hash, classification and gate results | — |

### What the outputs are

| File | Contents |
|---|---|
| `sextant-<version>.zip` | The package to upload |
| `sextant-<version>.zip.sha256`, `files.sha256` | Hashes of the package and of each file inside it |
| `sbom.cdx.json` | Software bill of materials for the bundled third-party code |
| `provenance.json` | How, from what and by which checks the package was made |
| `trust-diff.json` | Every trust-relevant difference, measured |
| `release-notes.md` | Permission, network and dependency sections generated from the diff; the privacy section is written by a person, every time |
| `artifact-scan.json`, `bundle-inventory.json`, `release-report.json` | The evidence behind the gates |

### What a hash proves

That a file is byte-for-byte the one these checks examined. **Not that it is harmless**, and not who
made it. The Chrome Web Store re-signs every package it distributes and adds its own metadata
(`_metadata/`), so the package a browser installs is not the uploaded zip: to check an installed version,
compare the extension's files — excluding `_metadata/` — with `files.sha256`.

### Reproducibility, stated exactly

Measured on every release: two consecutive builds of the same checkout, on the same machine and
toolchain, produce identical files. **Not yet measured:** a build on a different machine, operating
system, or Node or npm version. The zip itself is independent of file order and time zone by
construction, and `src/security/release-packaging.test.ts` proves it.

## 2. The publisher account

The Chrome Web Store accepts any package from the publisher account. Everything in §1 can be bypassed by
someone who controls that account, so the account is part of the release's security, not an
administrative detail.

**Human checklist — before the first upload, and reviewed yearly:**

- [ ] A **dedicated Google account** used only for publishing; not a personal or employer mailbox, not
      shared.
- [ ] **2-Step Verification with hardware security keys or passkeys** (the store requires 2-Step
      Verification to publish at all; SMS is not acceptable here). Consider Google's Advanced Protection
      Program for this account.
- [ ] **Recovery options** that are themselves protected: a recovery address with the same protection,
      backup codes stored offline. No recovery phone number that can be ported.
- [ ] **Verified CRX uploads enabled** (Developer Dashboard → Package → *Verified CRX Uploads*). The store
      then rejects any upload not signed with the publisher's private key — so a stolen account session
      cannot publish on its own. The private key is generated and kept **offline**, never on a build or
      CI machine and never in this repository. Losing it means a support request and up to a week
      without releases; back it up offline. *(This changes the upload artifact from a zip to a signed
      CRX; signing is a manual step after gate 15.)*
- [ ] The publishing browser profile has **no other extensions** installed.
- [ ] If the Chrome Web Store API is ever used for automation: credentials scoped to the one item, stored
      in a secret manager, rotated, never committed, never printed in logs — and verified uploads still
      require the offline key.
- [ ] The store's **trader / non-trader declaration** is made deliberately (a trader's contact details are
      shown publicly) — a legal decision, not a form to click through.
- [ ] Store notification emails go to a mailbox that is read, so an unexpected submission is noticed.

## 3. Uploading and after

**Human steps:**

1. Confirm the release report says **RELEASE ARTIFACT READY**, the classification is what you expect, and
   for a security-sensitive release that §4 was followed.
2. Check `sextant-<version>.zip.sha256` against the file you are about to upload.
3. Upload (or sign and upload the CRX, when verified uploads are on). Complete the privacy disclosures
   from `CWS_PRIVACY_DISCLOSURES.md`; do not improvise them.
4. After approval, install the published version in a clean profile and compare its files with
   `files.sha256`; confirm the permissions Chrome shows match `provenance.json`.
5. Commit `release-metadata/releases/<version>/`.

**A bad release** cannot be recalled from browsers that installed it. The response is a new version
(version numbers only go up) built through §1, published as fast as review allows, with the store item
unpublished in the meantime if users are at risk — `INCIDENT_RESPONSE.md` has the playbooks.

## 4. Release classification and Trust Architecture Changes

### Normal and security-sensitive

A release is **security-sensitive** when gate 13 measures any of:

- a permission or host permission added or removed;
- a changed content security policy;
- a URL origin in the bundle that the baseline does not approve;
- a third-party package added to or removed from the bundle;
- a dependency install script outside the reviewed list;
- a change to a security-boundary file (the transport, host validation, message guard, redaction, safe
  navigation, import safety, the data inventory, the trust facts, the permission explanations, the public
  security and privacy documents, or the release pipeline itself);
- no previous release to compare with (the first release).

Otherwise it is **normal**. A normal release's notes may say *"No permission or host access changes"*
because gate 13 measured it; nobody writes that sentence by hand.

### What a security-sensitive release needs

1. The diff read by a person, in full — not the summary.
2. For a new permission, host, destination or dependency: the reason, the narrowest form that works, and
   what users will see (a new permission disables the extension until each user accepts it).
3. `THREAT_MODEL.md`, `DATA_HANDLING.md` (regenerated), `PRIVACY.md` (both languages),
   `SECURITY.md`, the permission explanations and the store disclosures updated **in the same release**.
4. `release-metadata/trust-baseline.json` updated in its own reviewed commit.
5. The release run with `--approve-trust-change "<what changed and why it is acceptable>"`; the reason is
   recorded in `provenance.json`.

### Trust Architecture Changes

Some changes alter what users were told when they installed Sextant: **a server or account system,
telemetry or error reporting, cloud sync, a new external destination, remote configuration or code, an AI
provider, or a new permission.** Each needs, before any code ships:

- the threat model updated, and a privacy and security review recorded in `DECISIONS.md`;
- the privacy policy updated and dated **before** the release that introduces it;
- the Trust Center and store disclosures updated in that same release;
- the change named in the release notes' privacy section, in plain words;
- where the change is material, users told in the product before it takes effect;
- anything that sends data must be off until the user turns it on.

### If telemetry is ever proposed

Sextant has none, and this pass added none. Should a future release propose any — usage counts, error
reports, performance measurements — it is a Trust Architecture Change and, in addition to the list above,
it must be:

- **off until the user turns it on**, with a sentence at the switch saying exactly what is sent;
- **minimal and aggregate** — counts and error categories, never org metadata, labels, translations, API
  names, record or org ids, hostnames, user names or free text;
- **free of identifiers** — no session value, no persistent device or user id;
- **visible** — the payload viewable in Settings before and after it is sent;
- **named** — the destination listed in the CSP, the trust baseline, the privacy policy and the store
  disclosures, in the same release.

## 4a. The pre-launch trust checklist

Every item must be true before a first publication. The **Checked by** column says what makes it
checkable; items marked *human* cannot be automated.

| Area | Item | Checked by |
|---|---|---|
| Application | Typecheck, full test suite, release build | Gates 4, 5, 7 |
| Privacy | Every stored key classified; `DATA_HANDLING.md` current | `data-inventory.test.ts`, `data-handling-doc.test.ts` |
| Privacy | Policy matches the code, in both languages; reviewed by counsel | `trust-claims.test.ts`; **human** (legal review) |
| Privacy | No telemetry | `network-boundary.test.ts`, gate 9 |
| Privacy | View, export and delete work | `privacy-section.test.tsx`, `local-data.test.ts` |
| Security | Threat model current | **human** (reviewed against the release's trust diff) |
| Security | Session never stored, logged, exported | `session-boundary.test.ts` |
| Security | Messages, URLs, HTML sinks | `message-boundary`, `navigation-boundary`, `code-boundary` tests |
| Security | Network boundary enforced in a browser | Gate 10 |
| Supply chain | Lockfile install, reviewed install scripts, SBOM, bundled-package diff | Gates 3, 11, 13 |
| Release | Permission, host, CSP and network diffs; hashes; tag; changelog | Gates 1, 12, 13, 14 |
| Public surfaces | Trust Center built and checked; screenshots from the synthetic harness only | `website.test.ts`; **human** (screenshots) |
| Public surfaces | Security contact that is read; publisher named; store disclosures submitted as written | `npm run publication:check`; **human** |
| Ownership | Publication authorised by whoever owns Sextant (`P46-RG2`) | `npm run publication:check`; **human** |

## 5. Dependencies

- The lockfile is committed; installs use `npm ci`. `.npmrc` sets `strict-allow-scripts`, so a package
  wanting to run an install script fails the install until it is reviewed and added to `allowScripts`.
- **Updates are deliberate**, never automatic: read the changelog of anything that reaches the bundle,
  check the lockfile diff for new transitive packages and install scripts, and let gate 13 classify the
  release.
- `npm audit` findings are triaged by whether the vulnerable code reaches the bundle or only development
  tooling. Current triage: `AUDIT_2026-09-15.md` §6.
- A new runtime dependency must justify its weight: every package in the bundle runs with Sextant's
  permissions.

## 6. The publication gate

Passing every gate here makes a release **possible**, not **authorised**. No Sextant version may be
published — and no public repository created — until the ownership and publication question is resolved
by the people entitled to resolve it (`ROADMAP.md` `P46-RG2`). `npm run publication:check` lists every open
publication blocker; a real release refuses to run while any remain.

## 7. Versions, tags, release notes and the public repository

### One version

`package.json` holds the version; the manifest reads it, the demo compiles it in, the website reads it, the release pipeline records it
and the public repository's README prints it. Nothing else may state a version number (`src/security/version-consistency.test.ts`).
Versions follow Semantic Versioning: a patch for fixes, a minor for anything a user can see that is new, a major for a change in what the
product is or in what it asks the browser for.

### One description per release

A release is described for people in exactly one file, `release-metadata/releases/<version>/release.md` — a headline and up to nine fixed
sections (Highlights, Improvements, Fixes, Privacy, Security, Permissions, Storage, Compatibility, Known issues). `npm run release:notes`
generates from it:

| Output | Audience | Where |
|---|---|---|
| `CHANGELOG.md` | users, the website's changelog page, the public repository | repository root (generated — never edited by hand) |
| `release-metadata/releases/<version>/github-release.md` | the GitHub release body | pasted or uploaded as the release text; hashes appear in it only once the pipeline has produced them |
| `release-metadata/releases/<version>/store-notes.txt` | the Chrome Web Store listing's *what's new* paragraph (the store has no release-notes field) | pasted into the listing on submission |
| `release-notes.md` (from `release.mjs`) | reviewers | the measured permission, network and dependency sections of §1 |

`npm run release:verify` and the CI workflow fail when the committed changelog differs from what the descriptions generate, or when the
version being prepared has no description; a real release (§1 gate 14) also requires the description's `Released:` line to be a date.

### Tags and the sequence of a release

1. Write or finish `release-metadata/releases/<version>/release.md`; set `Released:` to the date; set `package.json`'s version.
2. `npm run release:notes`, then `npm run release:verify`. Commit: *Release <version>*.
3. Tag that commit `v<version>` — annotated, and signed when a signing key is configured (`git tag -s`); the tag is the release's identity and
   is never moved.
4. `npm run release` on the tagged, clean checkout (§1). Upload the package (§3). Commit the pipeline's `release-metadata/releases/<version>/`
   outputs (provenance, SBOM, hashes, trust diff) — a second commit, after the tag, because they record the outcome of building the tag.
5. `npm run public:build` and publish the resulting tree to the public repository (below). Create the GitHub release on the public
   repository from `github-release.md`, pointing at the store listing; attach no compiled artifact.
6. Deploy the website (`docs/HOSTING.md` §10) — the same commit, so the version in the footer, the changelog page and the store agree.

A release candidate is the same sequence stopped after step 2: `release/verify-report.json` and `npm run release:dry-run`'s
`release/dry-run-<version>/release-report.json` + `provenance.json` together state the version, the artifact hash, the demo state the website's
pictures came from, the permission/network/dependency diff, the classification and the open human blockers — which is the summary the final
decision needs.

### The public repository

The public repository is a *projection*, produced by `npm run public:build` (`scripts/release/public-repo.ts`) from an allow-list of files that
already exist here: a README rendered from the product's facts, the policies, `DATA_HANDLING.md`, `PERMISSIONS.md`, this document, the threat
model, the generated changelog, every release's description and evidence, support and issue templates. It contains no source code and no
compiled artifact, and says so. Updating it is one deliberate act per release: build it, review the scan's report, copy the tree into an
independent checkout of the public repository (`npm run public:sync -- <checkout>`, which also removes what the
projection no longer contains and stages the result for review), commit, push. It is never edited in place — anything worth changing is changed here and regenerated —
and `src/security/public-repo.test.ts` fails `npm test` when the projection would drift from the product (a store link while none exists,
a broken link, the old name, an unfilled fact, a changelog that is not the generated one).

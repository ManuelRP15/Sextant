# Security policy

This page is for anyone who has found, or is looking for, a security problem in Sextant. It says how
to report one, what is in scope, what Sextant is built to protect and — just as plainly — what it
cannot protect against.

The precise description of what Sextant accesses, stores and sends, with the code that makes each
statement true, is [`DATA_HANDLING.md`](DATA_HANDLING.md). The threats Sextant is designed against are
in [`THREAT_MODEL.md`](THREAT_MODEL.md), and how a release is built and
checked is in [`RELEASE_SECURITY.md`](RELEASE_SECURITY.md).

---

## 1. Reporting a vulnerability

> **⚠️ DECISION REQUIRED BEFORE PUBLICATION — the reporting channel does not exist yet.**
>
> It is left blank on purpose rather than filled with a guess. Choose one, make sure it is actually
> read, replace the placeholder below, and delete this block:
>
> - **GitHub private vulnerability reporting** (recommended; free; nothing to host) — enable it in the
>   public repository under *Settings → Code security → Private vulnerability reporting*, then the
>   line below becomes a link to *Report a vulnerability*.
> - **A dedicated address**, such as `security@` on a domain you control, with a mailbox someone reads.
>
> The same channel goes into `PRIVACY.md` §14, the Trust Center and `security.txt`.
> `npm run publication:check` fails while this placeholder is present.

**Please do not report security issues in a public issue.**

<!-- DECISION REQUIRED: the private reporting channel, per the block above. -->
`[ SECURITY CONTACT — NOT YET CONFIGURED ]`

What to include: the Sextant version (Settings → About, or `chrome://extensions`), the browser and
its version, what an attacker needs (a malicious web page? another extension? access to the
computer?), the steps, and what happens.

### What happens next

Sextant is maintained by one person. These are targets, not guarantees:

| Step | Target |
|---|---|
| Acknowledgement of your report | 5 working days |
| First assessment (confirmed or not, and how severe) | 15 working days |
| Fix for a confirmed high-severity issue | As fast as a safe release allows; you will be told the plan |
| Public disclosure | Coordinated with you, normally after a fixed version is available |

You will be credited in the release notes if you want to be, and not named if you do not.

### Research that is welcome

> **⚠️ LEGAL REVIEW REQUIRED** — the paragraph below is a statement of intent written by a
> non-lawyer. It must be reviewed before publication and must not be read as a legal promise until
> it has been.

Good-faith security research that follows this page is welcome, and will be treated as such:

- Test only against **your own** Salesforce orgs — a free Developer Edition org is ideal — and your
  own browser profile. Never against an org you are not authorised to test.
- Do not access, modify or keep data that is not yours; stop and report as soon as you find you can.
- Do not degrade a Salesforce service: no load testing, no denial of service.
- Give reasonable time for a fix before disclosing.

### Is it a security problem?

| Report it **privately**, as above | Report it as an ordinary bug or request |
|---|---|
| Something sends data somewhere other than your Salesforce org | Something is wrong, slow or confusing |
| Your session, a credential or org data could be exposed | A label is misidentified or a translation will not save |
| A web page, another extension or a crafted file can make Sextant do something | A feature request or a question |
| A released version behaves maliciously or differently from what it says | Documentation that is unclear |

When unsure, report privately; a bug sent there is simply redirected.

<!-- DECISION REQUIRED: the support channel for ordinary bugs (none exists yet; public issues once a repository is public). -->

## 2. Never include real data in a report

Sextant's whole subject matter is a Salesforce org, so this matters more here than usual.

**Do not send:** Salesforce session ids (`sid` cookie values), OAuth or refresh tokens, passwords;
real org ids, user ids, record ids or My Domain addresses; real metadata, label text or translations;
screenshots or HAR files captured against a real org; exports of Sextant's local data from a real org.

A session id in a report is a live credential in a mailbox. If a reproduction needs org-shaped data,
**make it up** — the shape reproduces the bug, not the values. `npm run harness:workspace` reproduces
every Sextant screen with no org at all.

If you have already sent something you should not have, say so. A Salesforce session can be revoked
in *Setup → Session Management*; rotating it is better than hoping.

## 3. Scope

**In scope**

- The extension as installed from its store listing: the background service worker, the content
  script, the Workspace and toolbar pages, and the built package itself.
- Anything that sends data to a destination other than the user's own Salesforce org, or that lets a
  web page, another extension or a Salesforce page make Sextant act on its behalf.
- The handling of the Salesforce session, local storage, exports, imports and deletion.
- The release process: a way to make a release contain something the checks in
  [`RELEASE_SECURITY.md`](RELEASE_SECURITY.md) should have caught.
- The website (`usesextant.dev`) and the interactive demo it hosts: a way to make either run a script it
  does not ship, send a request anywhere, set a cookie, or keep something in the visitor's browser.

**Out of scope**

- Vulnerabilities in Salesforce itself — report those to Salesforce.
- Vulnerabilities in a dependency with no exploitable path through Sextant (report upstream; a note
  here is welcome when the path *is* exploitable).
- Attacks that require the victim's Salesforce session already, full control of their computer or
  browser profile, or a malicious extension with more access than Sextant has — each is strictly more
  access than Sextant holds (see §5).
- The development harnesses, evaluation tooling and scripts, unless something from them reaches the
  built package.

## 4. What Sextant is built to protect

Each statement is enforced by the browser, checked by tests or checked on every release build —
[`DATA_HANDLING.md`](DATA_HANDLING.md) §1 says which, and where to verify it.

- **One destination.** Sextant's requests go only to Salesforce. The extension's content security
  policy (`connect-src 'self' https://*.salesforce.com`) makes the browser refuse any other
  destination for its pages and service worker; one module performs every request and checks it is
  addressed to the org whose session it uses. The content script makes no requests.
- **The session is used, not kept.** The `sid` cookie is read in the background when a request needs
  it and is never written to storage, logs, messages or exports.
- **No remote code.** Nothing is downloaded and run: no `eval`, no remotely loaded script, no CDN.
- **Messages are checked.** Every message the background accepts is checked against who sent it.
  The part of Sextant running inside Salesforce pages can ask only for what the in-page tooltip needs
  (look-ups, the edits you make in it, adding to your Workspace); deleting data, changing settings or
  forgetting an org is accepted only from Sextant's own pages. Sextant listens for no message from
  web pages or other extensions.
- **Local data is visible and deletable** in Settings → Privacy & security, and an export never
  contains a credential.
- **Writes are deliberate.** Nothing is written to Salesforce unless you ask. A save re-reads the
  org's current value first and refuses to overwrite a newer change. Saving a change straight to a
  production org — or to an org Sextant cannot identify — asks first, and that question cannot be
  switched off; changes collected in the deploy tray are sent when you press Deploy there.
- **Isolation from the page.** The tooltip mounts in a closed shadow root, and the content script runs
  only on `*.lightning.force.com`.

### The AI code in the repository, which is not in the extension

The source contains an unfinished AI subsystem, including an adapter for an AI provider's API. **It is
not in the extension you install**: the production build excludes it at compile time, and the release
scanner fails a build that contains that provider's endpoint. Even if the code were present, the content
security policy would block the request, and no screen exists to configure it. If an AI capability is
ever released, this page, the privacy policy and the store listing will say exactly what is sent, to
whom and under whose key, in the same release — not after.

## 5. What Sextant cannot protect against

Stated so nobody has to discover it.

- **A compromised computer or browser profile.** Sextant's local data is not encrypted by Sextant
  beyond what your operating system and browser provide. Anyone who can use your browser profile can
  read it, as they can read your Salesforce session.
- **Other extensions.** An extension with access to Salesforce pages can read what those pages show,
  including what Sextant displays in them.
- **Salesforce's own behaviour**, and anything your Salesforce user is permitted to do. Sextant acts
  with your user's permissions and cannot exceed them — and cannot stop a mistake your user is allowed
  to make, beyond asking before a production deployment.
- **A malicious update.** Sextant updates through the Chrome Web Store. A compromised publishing
  account or build machine could ship a harmful version; [`RELEASE_SECURITY.md`](RELEASE_SECURITY.md)
  describes the controls against that and their limits.

## 6. Supported versions

Only the latest version published in the Chrome Web Store receives security fixes. **No version has
been published yet.**

## 7. Changes that would weaken what this page says

Adding a server, analytics, a new destination, a new permission, remote code or an AI provider changes
what users were told when they installed Sextant. Each is a *Trust Architecture Change*: it requires
this page, the privacy policy, `DATA_HANDLING.md` and the store disclosures to change first, and a
release that contains one is classified security-sensitive and needs explicit approval
([`RELEASE_SECURITY.md`](RELEASE_SECURITY.md) §4).

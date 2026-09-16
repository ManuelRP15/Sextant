# Licence — DECISION REQUIRED

**This repository currently has no licence, and that is a blocking condition for publication.**

Under copyright law, source code with no licence is **"all rights reserved" by default**: readers
may look at it on GitHub but have no right to use, copy, modify or distribute it. That is a valid
position to hold deliberately — it is not a valid position to hold by accident, and publishing
without choosing is choosing it by accident.

This file exists so that the choice is made once, on purpose, with the trade-offs written down.
**Replace this entire file with the chosen licence text before the repository is made public.**

---

## Decide the prior question first

**Who owns this code?**

The licence question is downstream of the ownership question, and the ownership question is open
(open since 2026-07-31). Sextant was written by an employee of a
consultancy, and was driven and hardened against client and employer orgs. Depending on the
employment contract and the jurisdiction, some or all of it may not be the author's to license.

**Nothing below is legal advice, and no option here resolves that.** Grant of a licence by
someone who does not hold the copyright is void, so answering "which licence" before "whose code"
gets the order wrong. Settle ownership — in writing, from whoever can actually give that answer —
and then pick from the options below.

---

## The options, with the trade-off that actually decides it

The deciding question is **whether this may ever be sold.** The project's own business plan is the
official position on pricing and it does contemplate revenue, which rules out the reflexive
choice.

### MIT — permissive

Anyone may use, modify, sell, or fold this into a closed product, with attribution.

- **For:** maximum adoption; zero friction for contributors and for corporate users, whose legal
  teams approve MIT without a conversation. The default for a browser extension.
- **Against:** **irreversible in practice.** A competitor may fork Sextant, close their fork, and
  sell it — including back into this exact niche. Already-published versions stay MIT forever even
  if the licence changes later.
- **Pick it if** this is a portfolio piece and a gift to the ecosystem, and revenue is off the
  table.

### Apache-2.0 — permissive, with a patent grant

MIT's terms plus an express patent licence and a trademark carve-out.

- **For:** everything MIT offers, plus explicit patent protection for both sides, and it does not
  grant rights to the **Sextant** name. Preferred by larger companies over MIT.
- **Against:** the same irreversibility as MIT. Slightly longer.
- **Pick it if** MIT is right but the name should stay protected — see the trademark note below.

### AGPL-3.0 — strong copyleft

Anyone may use and modify it, but a distributed or network-served derivative must publish its
source under the same terms.

- **For:** a competitor cannot take this closed-source. Keeps the project open while removing the
  "fork it and sell it closed" outcome. Compatible with the author selling proprietary licences
  separately, since the copyright holder is not bound by their own licence.
- **Against:** **many corporate legal departments ban AGPL outright**, which for a tool aimed at
  Salesforce consultancies bites exactly the intended audience. Also unusual for an extension, and
  its network clause is a poor fit for something with no server.
- **Pick it if** open source matters *and* the fork-and-close outcome is unacceptable.

### Source-available / proprietary — all rights reserved

The code is readable on GitHub; no rights are granted. Optionally with a narrow grant (read,
audit, build for personal use, no redistribution).

- **For:** fully compatible with charging later. Delivers the real benefits of publishing —
  auditability, credibility, "you can verify the privacy claims yourself" — which for a product
  whose main promise is that your org's data goes to your org and nowhere else is worth a great deal.
- **Against:** **not open source**, and should never be described as such. GitHub will not show a
  licence badge. No outside contributions without a CLA. Some users will object on principle.
- **Pick it if** the auditability is the point and monetisation stays open. **Given that
  that plan contemplates a price and the IP question is unresolved, this is the option
  that forecloses the fewest futures.**

---

## Two things that are true whichever option is chosen

**1. It is easy to become more permissive later, and effectively impossible to become less.**
Starting restricted and relaxing later is a decision you can still make. Starting MIT and tightening
later is not: every published version stays MIT, and anyone may keep using and forking that version
forever.

**2. A licence is not a trademark.** None of these options gives away the **Sextant** name, but only
Apache-2.0 says so explicitly. Note also that `STORE_LISTING.md`'s blocking list records that a
live analytics company already trades as Sextant and that **no EUIPO or USPTO search has been run
(classes 9 and 42)** — so the name's availability is currently unknown, independently of this file.

---

## Copyright line, for whichever licence is chosen

```
Copyright (c) 2026 [ COPYRIGHT HOLDER — see "Decide the prior question first" above ]
```

The holder is a person or a company, and which one it is is exactly the open question. Do not fill
it in from habit.

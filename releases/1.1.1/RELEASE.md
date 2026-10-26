# Sextant 1.1.1

Released: 2026-10-26
Headline: Sextant does much less work when you move between records in a large org, uses less memory, and can hand a translation problem to your team's tracker as text.

<!--
This file is the ONE place a release is described for people. `npm run release:notes` generates
CHANGELOG.md, the GitHub release body and the store's "what's new" text from it; nothing is written
twice. `Released:` becomes a date in the commit that is tagged v1.1.1.
-->

## Highlights
- **Lighter when you move between records in a large org.** Each time a page shows an object Sextant has not read recently, it reads that page's metadata and saves it. Saving used to go through everything Sextant knew about the org, several times; it now touches only what changed, and does nothing when nothing changed. Measured on a synthetic org of 94,000 components: from about 1.5–2.9 seconds of background work per save to about 0.6 seconds. Real orgs differ; this is the part of Sextant that grew with the size of the org.
- **Translate All on large orgs.** Text on the page that is not a label (record values, names, dates) is now ruled out at once instead of being compared with everything Sextant knows. On the same synthetic org, marking a page went from about 30 seconds of background work to under half a second.
- **Copy as issue.** In the Inspector, under a component's translations: copies its name, API name, type, org and Setup link, and its values in your languages, as text for Jira, Azure DevOps, Teams or an e-mail. Nothing is sent anywhere; it goes to your clipboard.

## Improvements
- Uses less memory for the same answers: the lookup behind hover and Translate All is about a tenth of the size it was on a large org.
- Translate All keeps up on pages that never stop changing (a ticking clock, a spinner) instead of waiting for them to settle.

## Fixes
- After hovering a label, Sextant no longer keeps that part of the page alive in memory once you let go of Shift.
- In the comparison, the summary's tiles sit two to a row on a narrow screen instead of one per row.
- The guided tour of the online demo no longer reports a step ready while its highlight is still moving.

## Storage
- Sextant now keeps an org's revision bookkeeping beside its index instead of inside it, so reading the small part no longer reads the large one. Existing data is converted the first time Sextant saves for that org; nothing is lost and nothing needs to be done.

## Privacy
- Nothing new is collected or sent anywhere. *Copy as issue* writes only to your clipboard, and leaves out the page's address and any record data.

## Known issues
- The Japanese interface has not been reviewed by a native speaker. Dates still follow your computer's settings, a few messages that come from Salesforce or from Sextant's background work are still in English, and lists are joined with commas.
- With the default setting, Search on an org you have not used Sextant on yet starts complete only once you open it: the first opening shows what is already read and fills in as the org is read.
- Editing covers Custom Labels, custom (`__c`) fields and their picklist values, global value sets, record types, custom buttons and links, quick actions, layout sections, custom tabs and apps. Object labels, related lists, field help text and text inside flows are read-only for now; standard buttons and the record page's standard tabs are Salesforce's own translations and stay read-only.
- Text inside a Flow is not identified, on hover or in Search: Salesforce puts nothing on the page that ties it to its metadata, and Sextant does not guess.
- A save through the Metadata API takes about a minute, almost all of it Salesforce's own deploy queue. Custom Labels use a faster path.
- Not validated against Professional Edition or orgs using API Access Control restrictions.
- The performance figures above come from a synthetic org; how much a given org gains depends on its size and how its pages are built.

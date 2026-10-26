# Sextant 1.1.0

Released: 2026-10-25
Headline: Sextant stays out of Salesforce's way when you are not using it, the interface is available in Japanese, and comparing translations across orgs shows what is missing from an org.

<!--
This file is the ONE place a release is described for people. `npm run release:notes` generates
CHANGELOG.md, the GitHub release body and the store's "what's new" text from it; nothing is written
twice. `Released:` becomes a date in the commit that is tagged v1.1.0.
-->

## Highlights
- **Lighter on Salesforce when you are not using Sextant.** An open Salesforce tab now makes Sextant read only what hovering on that page needs: the page's own metadata and your Custom Labels. The rest of an org (the full catalog that Search uses, translation checks, change checks) is read while you use Search, the Workspace or Activity on that org, and for a quarter of an hour after. Sextant also no longer copies what it saves into every open Salesforce tab, no longer checks the page's address twice a second, and does nothing in tabs you are not looking at until you switch to them. Measured on a heavy test page with three Salesforce tabs left idle for two minutes, the processor time of Sextant's background work fell from about half a second to a few hundredths, the browser's own work returned to what it is without Sextant, and nothing was written to storage.
- **The interface in Japanese.** Choose Japanese in Settings → Interface language. It uses Salesforce's own Japanese names for Custom Labels, Translation Workbench, picklist values and the rest, so what Sextant says matches what a Japanese Setup shows. The Japanese translation has not yet been reviewed by a native speaker, and Sextant says so where you choose it; Automatic still means English.
- **Comparing translations across orgs shows what is not in an org.** A component that exists in one org and not in another (a field in development that never reached UAT) is now shown as *not in this org*, in the table, in the sentence that explains each row and in the export. The comparison can also be downloaded as a CSV file that opens correctly in Excel, in every language.

## Improvements
- *Keep reading orgs in the background* (Settings → On the Salesforce page) brings back the previous behaviour for anyone who wants Search complete before opening it, at the cost of more requests while Salesforce is open.
- Language names, the page language and Japanese text are set correctly for Japanese, so kanji use Japanese glyph forms.
- The organisation picker in the comparison now shows which orgs can be compared (those read whole), and when.

## Fixes
- **Typing Japanese, Chinese or Korean no longer saves or closes things by accident.** Pressing Enter to confirm a conversion saved a translation half-typed; Escape under a conversion cancelled the editor or closed the panel around it; confirming a search with Enter opened a result. Keys an input method is still composing with now belong to it, in every editor, search box and panel.
- **Thirteen Spanish texts showed `{etiqueta}`** instead of the component's name (*Añadir {etiqueta} a tu Workspace*). They show the name.
- In the comparison: a lost answer now says nothing was compared and offers Try again (it used to show *Comparing…* as an error); a table cell containing `|` or a line break no longer breaks the Markdown export; an org's *missing* count matches what its filter shows.
- Activity's footnote no longer ends a Japanese sentence with an English full stop.

## Privacy
- Nothing new is collected or sent anywhere. Sextant still keeps what it reads on this device only, in the browser's extension storage.
- Salesforce tabs now learn only the names of the settings and stores that changed and read what they show, instead of receiving every value Sextant saves.

## Security
- One new internal message lets the on-page Search tell Sextant that you are using it on that page's org. Like every message from a Salesforce page, it is accepted only for the page's own org.

## Permissions
- No new permission.

## Compatibility
- Same browsers as 1.0.1. On Chrome or Edge versions older than 102, Sextant falls back to checking the page's address periodically, as before.

## Known issues
- The Japanese interface has not been reviewed by a native speaker. Dates still follow your computer's settings, a few messages that come from Salesforce or from Sextant's background work are still in English, and lists are joined with commas.
- With the default setting, Search on an org you have not used Sextant on yet starts complete only once you open it: the first opening shows what is already read and fills in as the org is read.
- Editing covers Custom Labels, custom (`__c`) fields and their picklist values, global value sets, record types, custom buttons and links, quick actions, layout sections, custom tabs and apps. Object labels, related lists, field help text and text inside flows are read-only for now; standard buttons and the record page's standard tabs are Salesforce's own translations and stay read-only.
- Text inside a Flow is not identified, on hover or in Search: Salesforce puts nothing on the page that ties it to its metadata, and Sextant does not guess.
- A save through the Metadata API takes about a minute, almost all of it Salesforce's own deploy queue. Custom Labels use a faster path.
- Not validated against Professional Edition or orgs using API Access Control restrictions.

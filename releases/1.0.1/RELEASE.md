# Sextant 1.0.1

Released: 2026-09-29
Headline: Every key you type reaches Sextant on a Salesforce page, Refresh recovers after a lost connection and waits for slower orgs, Search answers however soon you start typing, and paste is paste again in New Custom Labels.

<!--
This file is the ONE place a release is described for people. `npm run release:notes` generates
CHANGELOG.md, the GitHub release body and the store's "what's new" text from it; nothing is written
twice. `Released:` becomes a date in the commit that is tagged v1.0.1.
-->

## Highlights

## Improvements

## Fixes
- **Every key reaches Sextant's text fields on a Salesforce page.** Salesforce's own single-key shortcuts — `e` to edit the record, `/` to search, and others in console apps — were taking letters typed into the hover editor, Translate All and Search, most visibly `e`. What you type into Sextant now stays in Sextant; Salesforce's shortcuts keep working everywhere else on the page.
- **Search no longer gets stuck while the org's index loads.** Typing before the index had finished loading left Search on *Reading this org's index…*, without answers, until it was closed. It now answers what you typed as soon as the index is read, over a Salesforce page and in the toolbar menu alike.
- **Refresh works on the first press after the connection to Salesforce comes back.** The first Refresh after a lost connection could finish with most of its checks skipped and had to be pressed again. It now checks the connection first: if Salesforce answers, the Refresh runs in full; if it still does not, the Refresh stops at once and says so.
- **Translation Workbench edits are checked on slower orgs.** Refresh waited only about 30 seconds for Salesforce to hand over the files that show an edit made in Translation Workbench. On an org that takes longer, every Refresh ended *Translation Workbench edits couldn't be checked*, and such an edit was never picked up. Refresh now waits as long as Salesforce is working on them.
- **A Refresh that loses the connection says it couldn't refresh.** It used to finish as a partial check that named whichever part came first; now it says *Couldn't refresh* and keeps showing the last read. A Refresh that finds nothing changed is still *Up to date*.
- **Paste is paste again in New Custom Labels.** Ctrl+V (Cmd+V on a Mac) in the grid could be taken for a spreadsheet import: a value with a `|` was split into columns, one cell copied from Excel replaced the rows below, and two lines of text were refused as not being a spreadsheet. Pasting into a cell now always pastes the text. To bring in rows from Excel or Google Sheets, use *Paste from spreadsheet*: the rows are added after the ones already in the grid.

## Privacy

## Security

## Permissions

## Compatibility

## Known issues
- Editing covers Custom Labels, custom (`__c`) fields and their picklist values, global value sets, record types, custom buttons and links, quick actions, layout sections, custom tabs and apps. Object labels, related lists, field help text and text inside flows are read-only for now; standard buttons and the record page's standard tabs are Salesforce's own translations and stay read-only.
- Text inside a Flow is not identified, on hover or in Search: Salesforce puts nothing on the page that ties it to its metadata, and Sextant does not guess. The flow itself is found in Search by its name.
- A save through the Metadata API takes about a minute, almost all of it Salesforce's own deploy queue. Custom Labels use a faster path.
- Not validated against Professional Edition or orgs using API Access Control restrictions.

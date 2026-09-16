# Support

Sextant is maintained by one person. Issues are read; there is no guaranteed response time.

## Before you open an issue

- **A security problem — anything that could expose a session, a credential or org data, or make Sextant send data anywhere but your org — goes through [SECURITY.md](SECURITY.md), never a public issue.**
- Check the [known limitations](CHANGELOG.md) for the version you are on: some behaviour is deliberate (standard fields are read-only, hover shows four types, a Metadata API save takes about a minute).
- Check that your org meets the requirements: Lightning Experience, *API Enabled* on your user, and *Setup → Session Settings → Lock sessions to the domain in which they were first used* disabled. With that setting on, Sextant cannot connect, and a fresh install looks broken.

## Reporting a bug

Open a **Bug report** issue. What helps: the Sextant version (Settings → About), the browser and its version, what you did, what you expected, what happened instead, and whether it happens in [the interactive demo](https://usesextant.dev/demo/) too — a reproduction there is worth more than a description, because it contains no real data.

**Never include real org data.** No session ids (`sid` cookie values), no org ids, user ids or record ids, no My Domain addresses, no real metadata API names or translation values, and no screenshots or exports from a live org. The *shape* of the data reproduces a bug; the values do not, and a session id in an issue is a live credential on a public page. If you have already posted something you should not have, edit it out and revoke the session in *Setup → Session Management*.

## Requesting a feature

Open a **Feature request** issue. Say what you are trying to get done and where Sextant stops short, rather than the control you would like to see — the problem is what gets built, and the control may end up somewhere else.

## Documentation feedback

If a policy or a document here is unclear or wrong, an issue is welcome; say which document and which sentence.

## Code contributions

Sextant's source is not open for contributions at this time; see [CONTRIBUTING.md](CONTRIBUTING.md).

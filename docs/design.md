# Content Editor: Design

> **In short:** content-editor is the editing layer for Publisher's documents. It draws the screens for people who edit, saves through Publisher's own server functions, and owns no content. Every action is still decided by Publisher and Authorisation. Rich text is edited in a formatted editor and saved as a safe tree, never as HTML. Images come from a new media capability, designed separately. Agreed with the owner on 25 October 2026; nothing is built yet.

## What it is for

People who run a site, and later people who run a group, need to write and publish pages, articles, events and the rest without touching the repository. Publisher already holds documents, versions, drafts, schedules and approvals ([contract](https://github.com/nuxt4-layers/publisher/blob/4410cab7b84c1b0395de2bdf657e3bcc3d476b87/docs/contracts.md)), and draws nothing (its §15 leaves editing to this layer). content-editor supplies the missing part: the editing screens and the little server code behind them.

## Decisions

| Decision | Means |
|---|---|
| The host passes Publisher's server functions | The host's one composition call hands content-editor `createPublisherDocument`, `transitionPublisherDocument`, `getPublisherDocument`, `listPublisherDocuments`, `listPublisherHistory` and the type registry. content-editor never imports Publisher, and Publisher's HTTP API can stay off |
| It owns no content | No tables for documents or drafts. Publisher keeps every version. Any record of its own would need a design change first |
| It decides nothing | Every action goes to Publisher, which checks permission, step-up, approval and `expectedVersion`. The editor only hides what the caller cannot do, using the shared `decide` shape |
| A formatted editor | Built on ProseMirror (through TipTap). Each rich-text field offers only what its [rich-text](https://github.com/nuxt4-layers/rich-text/tree/68ac5b008e60ae0150171324eedeb22eb37110cc) profile allows. It saves the safe tree. Pasted content is turned into the tree, never kept as HTML |
| Forms come from each type | One editor serves every registered type, built from the type's schema: plain fields as text inputs with their length limits, rich-text fields as the formatted editor. A new type needs no editor code |
| Explicit save | Save makes a new version; there are no autosave versions. Unsaved work is kept in the browser until saved, and leaving with unsaved changes asks first |
| Previews on the site's own pages | Preview opens the host's page for the document in Publisher's preview mode, marked `noindex`, so it looks exactly as it will when published |
| No second person by default | Publisher's floors apply: publishing is `medium`, needing a second step at sign-in but no approver. Every action is an event for the future audit capability (logging after the fact). A type can raise its risk to require an approver; the editor then shows submit and approval states |
| Images from a media capability | A new layer, `nuxt4-layers/media`, designed in its own note before this one is built. content-editor offers its picker and inserts `asset:<id>` references, which Publisher resolves through its asset port |
| Its own screens, for editors only | A presentation part (amended ADR-0009) at `/editor/...`, movable or switched off with `contentEditor: { presentation: false }` |

## Screens

| Path | Shows |
|---|---|
| `/editor` | The types the caller may edit, and recent documents |
| `/editor/:type` | That type's documents, with search and a status filter |
| `/editor/:type/new`, `/editor/:type/:documentId` | The form, its status and the actions the caller holds: save, submit, withdraw, publish (with an optional schedule), unpublish, archive, reopen |
| `/editor/:type/:documentId/history` | Versions newest first, each with who saved it and when, and restore |

Screens are styled only with Theme Manager's vocabulary, with wording from an en-GB message catalogue the host can override, and meet WCAG 2.2 AA in light and dark mode. Administration rules from Publisher's §15 hold: never framed, cached or sent as a Referer. People are named through the host's name component, never looked up by the editor.

## Composition

Within [ADR-0012](https://github.com/nuxt4-layers/platform-architecture/blob/249cd48af72e0a3fc232d40c8aa05e8da3b32069/docs/decisions/ADR-0012-trivial-composition.md)'s budget from the start: one install, one call passing Publisher's functions, `resolveCaller` and `decide` (from iam-integration once it complies), and the media picker if composed. At most 50 lines of host code. The layer ships fakes of every port, `examples/minimal-host` and `CONTRACT.md` (at most 200 lines).

## The media capability

Its own repository and design note. Agreed so far:

- Each media item's record (owner group, alt text, size, type) is in its own PostgreSQL schema; the files are in a storage bucket the host passes in.
- For Westray-Papa-Westray the bucket is Vercel's storage, chosen to be as near free as possible. Its region must be confirmed as the UK before use, because the site's data stays in the UK.
- To settle in its note: upload limits, resizing, removing location data from photos, alt text being required, who may upload, and deleting an item that documents still use.

## Not in version 1

Simultaneous editing by two people (the second save gets `conflict` and reloads), comments on drafts, moderation queue screens (Publisher's submissions), translation, and editing the site's facts (Configuration's administration, a later step).

## Order of work

1. The media capability's design note, with the owner.
2. content-editor's core: contract, composition call, endpoints over the passed functions, fakes and tests.
3. Its presentation part: the screens and the formatted editor.
4. `examples/minimal-host`, `CONTRACT.md` and a release.
5. Westray-Papa-Westray composes it and grants the editing roles.

# Grabba Security and Privacy

Grabba is intentionally local-only and follows a least-privilege design.
This document describes what the extension can access, where sensitive data
goes, and what it does not protect against.

## Data flow

1. Before the first recording, the user must review the bundled local
   processing, unencrypted-file, and authorization notices, select an
   affirmative checkbox, and accept the current consent version.
2. A user clicks the Grabba toolbar icon on an active HTTP or HTTPS tab, or
   invokes the locally configured **Start recording** keyboard shortcut.
3. The background worker independently verifies the accepted consent version,
   then checks the tab URL against bundled protected-service
   rules and the local UT1 policy snapshot. A blocked site never reaches the
   capture request.
4. Chrome grants temporary `activeTab` access for that user-selected tab.
5. The background worker requests a tab-only media stream from
   `chrome.tabCapture`.
6. The bundled offscreen recording engine holds video and optional tab audio
   in memory while recording.
7. The user can pause and resume without moving captured data outside the
   recording engine.
8. When the user selects **Stop and save**, or when navigation protection
   detects that the recorded tab is navigating, the engine repairs Chromium's
   temporary fragmented MP4 and locally remuxes its encoded H.264/AAC packets
   into a conventional Fast Start MP4. No decoding or recompression occurs.
   The engine then sends a same-extension Blob URL to Chrome's downloads API.
9. Alternatively, the user may select **Discard** and affirm a destructive
   confirmation. This immediately stops the recorder, skips MP4 finalization,
   Blob URL creation, and the downloads API, then clears chunks and tracks
   from the hidden recording context.
10. Chrome writes a saved file to `Downloads/Grabba`. Only after Chrome
   confirms that the download is complete does the engine revoke the Blob URL,
   clear recording chunks and media tracks, and close its offscreen document.

There is no network step in this flow.

### Local editor data flow

1. The user opens the bundled editor and explicitly selects an MP4 file.
2. Chrome provides that local file only to the editor page. Grabba does not
   store a file handle, path, video content, or editing history in extension
   storage.
3. The bundled MediaBunny code reads MP4 metadata and encoded packets locally.
   Cuts are aligned to existing video keyframes. Video is not decoded,
   recompressed, sent to a server, or passed to another extension context.
4. When exporting, the user chooses a new local filename through Chrome's
   save-file interface when available. Grabba rejects the original filename.
   If that interface is unavailable, a local Blob URL is sent directly to the
   downloads API under `Downloads/Grabba/Edited`.
5. The output is a new ordinary, unencrypted MP4. The selected source file is
   never modified. Temporary Blob URLs are revoked after use, and no file
   handle is persisted.

There is no network step in the editor flow.

## Permission rationale

| Permission | Purpose |
| --- | --- |
| `activeTab` | Temporarily identifies and authorizes only the active tab selected by an explicit toolbar or keyboard-command gesture. |
| `tabCapture` | Captures video and, when selected, audio from that tab. |
| `offscreen` | Runs `MediaRecorder` in a bundled hidden extension page after the popup closes. |
| `downloads` | Writes the completed MP4 to the local Downloads folder. |
| `storage` | Stores recording preferences and non-sensitive completion status. |

The extension does not request `tabs`, `<all_urls>`, host permissions,
microphone access, camera access, cookies, browsing history, clipboard access,
or native messaging.

## Security controls

- Manifest V3 service worker architecture.
- Explicit Content Security Policy with `connect-src 'none'` and no remote
  sources. `media-src blob:` is narrowly allowed for previewing the local file
  selected by the user; HTTP, HTTPS, data, object, frame, and worker media
  sources remain unavailable.
- No `fetch`, `XMLHttpRequest`, WebSocket, telemetry, analytics, or upload
  implementation.
- No content scripts, page injection, `eval`, `new Function`, `innerHTML`, or
  remotely hosted code.
- No externally connectable API and no web-accessible resources.
- Recording starts only on the current active tab and only after an explicit
  toolbar or configured keyboard-command gesture.
- The keyboard shortcut reuses the last locally saved quality, tab-audio, and
  navigation-protection settings. It independently rechecks the current
  consent version and protected-domain policy before requesting capture. If
  consent is missing, the tab is protected, or a recording already exists,
  Grabba opens its popup instead of bypassing the relevant control.
- A versioned first-use notice requires an affirmative checkbox before
  acceptance. Both the popup and the background worker prevent recording
  without the current consent version. The locally stored value contains no
  identity, timestamp, tab information, or captured content.
- Known meeting, messaging, webmail, banking, and financial services are
  rejected before `chrome.tabCapture` receives a request.
- Protected-domain matching uses only bundled extension files. There are no
  runtime classification requests, API keys, service accounts, or remote
  policy downloads.
- Optional navigation protection, enabled by default, automatically stops and
  saves when the recorded tab begins loading another page.
- Discarding requires a second affirmative action. The offscreen engine has a
  dedicated discard path that does not build a final MP4, create a Blob URL,
  invoke `chrome.downloads`, or persist a completion result. The background
  worker clears any prior session result before closing the offscreen context.
- The navigation-protection information dialog explains six operational
  details in every bundled language: which tab is monitored, what Chrome load
  events trigger it, automatic MP4 finalization, exclusion of the destination
  page, the difference between navigation and tab switching, and the expanded
  capture boundary when protection is disabled.
- Protected-site rules are evaluated before capture begins. If navigation
  protection is disabled, a later navigation can therefore carry the existing
  capture into a sensitive page. Users are instructed to disable it only for
  an explicitly authorized multi-page workflow.
- Only HTTP and HTTPS tabs are accepted. Chrome internal pages, other
  extensions, local files, data URLs, DevTools, and the Chrome Web Store are
  rejected.
- Incognito operation is disabled in the manifest.
- Internal message senders, actions, stream identifiers, tab metadata, and
  quality values are validated before privileged operations.
- Download URLs must be Blob URLs created by this extension's own origin.
- Tab titles are used only in the live popup status and are not included in
  saved filenames.
- `chrome.storage.local` and `chrome.storage.session` are restricted to trusted
  extension contexts. Persistent storage holds only quality, tab-audio,
  navigation-protection, interface-language preferences, and the accepted
  consent notice version. Session storage may hold a generic filename, file
  size, duration, download identifier, stop reason, or bounded error message
  so the popup can report a completed automatic save.
- Captured media, tab URLs, and tab titles are never written to extension
  storage. The temporary Blob URL is revoked immediately after the local
  download completes, and the offscreen recording document is then closed.
- The editor never persists the selected source, file-system handle, edit
  intervals, or output video in extension storage. File names are inserted
  into the interface only through `textContent`, and download names are
  sanitized.
- The editor accepts MP4 only, validates the video track, codec, duration,
  decoder configuration, and keyframes, and permits export only when at least
  one interval remains.
- All code is bundled locally. The pinned MediaBunny dependency is compiled
  into `editor-runtime.js`; the installed extension neither downloads nor
  dynamically imports package code.

## Protected-domain policy

Grabba includes a curated permanent list for known services such as
Google Meet, Microsoft Teams, Zoom, Webex, Gmail, Outlook, and selected
messaging and financial platforms. It also includes a normalized release-time
snapshot of the public Université Toulouse Capitole UT1 `bank`, `chat`, and
`webmail` categories.

The snapshot source, hashes, counts, transformations, and license are recorded
in the generated data and in `THIRD_PARTY_NOTICES.md`. The updater validates
minimum category sizes, removes entries that cannot be represented as valid
Chrome hostnames, and binds every snapshot to one exact extension version.

This policy is intentionally a denylist:

- Domains not present in a protected category remain recordable.
- A missing entry does not prove that a page is safe or non-sensitive.
- Public categorization can contain false positives, false negatives, or stale
  entries.
- Domain matching cannot determine whether a meeting is currently in progress
  or whether an allowed page embeds sensitive content from another service.

These limitations avoid persistent access to browsing data and preserve the
extension's local-only architecture.

## Local file responsibility

Grabba does not encrypt the completed MP4. After download, its protection
depends on the operating system, account permissions, disk encryption,
security software, backups, and any cloud synchronization configured for the
Downloads folder.

For confidential recordings:

- Use a managed device with full-disk encryption.
- Move the file promptly to an approved encrypted storage location.
- Avoid recording passwords, private keys, recovery codes, or unrelated
  confidential tabs when possible.
- Remember that tab audio may contain notifications or spoken sensitive data.
- Securely delete recordings according to your organization's retention
  policy.
- Do not install modified builds from untrusted sources.

## Verification

Run the static security audit:

```powershell
node .\tests\domain-policy.mjs
node .\tests\security.mjs
node .\tests\i18n.mjs
node .\tests\editor.mjs
```

Run the end-to-end recording suite:

```powershell
node .\tests\e2e.mjs
```

The static audit verifies the permission allowlist, absence of host access and
external entry points, restrictive CSP, complete offline translations, the
explicit English default, forbidden network APIs, dangerous DOM sinks, and
remote executable resources. The domain-policy audit verifies the release-bound UT1 snapshot,
minimum category counts, protected-service decisions, hostname spoofing
resistance, and denylist-only behavior for unlisted domains.

The end-to-end suite loads the extension into an isolated Chrome profile,
verifies that the initial consent is unchecked, blocks direct recording before
acceptance, persists acceptance locally, opens and validates the complete
navigation-protection dialog, records every quality/audio combination, closes
and reopens the popup, saves the MP4, validates its H.264/AAC structure, and
requires a conventional Fast Start layout with no `moof` or `mfra`, then
verifies timeline seeking at 10%, 50%, and 90%. It also verifies pause/resume behavior, automatic
stop-and-save on navigation, download completion, removal of the offscreen
recording context, confirmed discard with no download or saved result, the
protected-domain policy loaded by Chrome's service worker, actual recording
startup through the configured keyboard command using saved settings, and a complete
editor workflow that removes intervals and the audio track while preserving a
seekable MP4 and the original source file.

## Reporting a security issue

Do not include real recordings, credentials, personal data, or confidential
tab contents in a public issue. Provide the smallest synthetic reproduction
possible and identify the affected version and Chrome version.

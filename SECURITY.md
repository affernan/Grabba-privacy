# Grabba Security and Privacy

Grabba is intentionally local-only and follows a least-privilege design.
This document describes what the extension can access, where sensitive data
goes, and what it does not protect against.

## Data flow

1. Before the first recording, the user must review the bundled local
   processing, unencrypted-file, and authorization notices, select an
   affirmative checkbox, and accept the current consent version.
2. A user clicks the Grabba toolbar icon on an active HTTP or HTTPS tab and
   selects **Start recording** in the popup.
3. The background worker independently verifies the accepted consent version,
   then checks the tab URL against bundled protected-service
   rules and the local UT1 policy snapshot. A blocked site never reaches the
   capture request.
4. Chrome grants temporary `activeTab` access for that user-selected tab.
5. The background worker requests a tab-only media stream from
   `chrome.tabCapture`.
6. The bundled offscreen recording engine streams video, optional tab audio,
   and separately opted-in microphone audio
   fragments to Grabba's extension-private Origin Private File System instead
   of retaining the complete recording in memory.
7. The user can pause and resume without moving captured data outside the
   recording engine.
8. When the user selects **Stop and save**, or when navigation protection
   detects that the recorded tab is navigating, the engine repairs Chromium's
   temporary fragmented MP4 and locally remuxes its encoded H.264/AAC packets
   into a conventional Fast Start MP4. No decoding or recompression occurs.
   The engine then sends a same-extension Blob URL to Chrome's downloads API.
9. Alternatively, the user may select **Discard** and affirm a destructive
   confirmation. This immediately stops the recorder, skips MP4 finalization,
   Blob URL creation, and the downloads API, then deletes the temporary session
   and clears media tracks from the hidden recording context.
10. Chrome writes a saved file to `Downloads/Grabba`. Only after Chrome
   confirms that the download is complete does the engine revoke the Blob URL,
   delete the temporary session, clear media tracks, and close its offscreen
   document.

There is no network step in this flow.

## Permission rationale

| Permission | Purpose |
| --- | --- |
| `activeTab` | Temporarily identifies and authorizes only the active tab selected through the toolbar popup. |
| `tabCapture` | Captures video and, when selected, audio from that tab. |
| `offscreen` | Runs `MediaRecorder` in a bundled hidden extension page after the popup closes. |
| `downloads` | Writes the completed MP4 to the local Downloads folder. |
| `storage` | Stores recording preferences and non-sensitive completion status. |

The extension does not request `tabs`, `history`, `<all_urls>`, host permissions,
camera access, cookies, clipboard access, or native
messaging. `activeTab` provides the selected tab's current title and URL only
after the user invokes Grabba; this information is processed locally and is not
retained as browsing history.

Microphone capture is a separate, off-by-default choice. Chrome's microphone
permission is requested when it is enabled. Grabba does not store that choice,
does not route microphone audio to speakers, and stops the microphone track on
save, discard, or recording failure. The browser may retain its origin-level
permission until the user revokes it in Chrome settings.

## Security controls

- Manifest V3 service worker architecture.
- Explicit Content Security Policy with `connect-src 'none'`, `media-src
  'none'`, and no remote sources.
- No `fetch`, `XMLHttpRequest`, WebSocket, telemetry, analytics, or upload
  implementation.
- No content scripts, page injection, `eval`, `new Function`, `innerHTML`, or
  remotely hosted code.
- No externally connectable API and no web-accessible resources.
- Recording starts only on the current active tab after the user opens the
  toolbar popup and selects **Start recording**. This release declares no
  keyboard commands.
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
- Captured media is written only to a dedicated temporary directory in the
  extension-private Origin Private File System; it is never written to
  `chrome.storage`. The session is deleted after download or discard. A session
  left by an unexpected Chrome shutdown is removed before the next recording.
  Tab URLs and titles are never written into the temporary media.
- The temporary Blob URL is revoked immediately after the local download
  completes, and the offscreen recording document is then closed.
- The current package has no editor entry point, editor page, editor stylesheet,
  editor runtime bundle, or keyboard command. Static and packaged-extension
  tests enforce their absence.
- All executable code is bundled locally. The pinned MediaBunny dependency is
  compiled into `recorder-remux-runtime.js`; the installed extension neither
  downloads nor dynamically imports package code.

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
navigation-protection dialog, records every video quality and the no-audio,
tab-audio, microphone-only, and combined-audio cases, closes
and reopens the popup, saves the MP4, validates its H.264/AAC structure, and
requires a conventional Fast Start layout with no `moof` or `mfra`, then
verifies forward and backward timeline seeking at 10%, 50%, and 90%. It also
verifies pause/resume behavior, automatic
stop-and-save on navigation, download completion, removal of the offscreen
recording context, confirmed discard with no download or saved result, the
protected-domain policy loaded by Chrome's service worker, and the absence of
editor assets, editor entry points, and keyboard commands in both the source
tree and final Store package.

## Reporting a security issue

Do not include real recordings, credentials, personal data, or confidential
tab contents in a public issue. Provide the smallest synthetic reproduction
possible and identify the affected version and Chrome version.

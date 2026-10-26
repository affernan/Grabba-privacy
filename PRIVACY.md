# Grabba Privacy Policy

Effective date: October 26, 2026

Grabba records a Chrome tab selected by the user. Recording, processing, and
MP4 creation take place locally in the user's Chrome browser. Grabba has no
developer-operated server and does not upload recordings or captured tab
information.

## Information handled by Grabba

Grabba handles the following information only to provide its user-facing
recording features:

- The video displayed in the tab that the user explicitly selects for
  recording.
- Audio playing in that tab, only when the user enables tab audio.
- Microphone audio, only when the user separately enables Microphone and Chrome
  grants access. The microphone is off by default on each popup opening.
- The selected tab's title and URL, used locally to identify the tab and enforce
  the protected-site policy before recording starts.
- Local preferences such as language, video quality, tab-audio choice,
  Navigation protection, and the accepted privacy-notice version.

The content visible or audible in a selected tab may itself contain personal,
confidential, or sensitive information. The user controls which eligible tab
is processed.

## Local processing and network use

Grabba does not transmit the information above to the developer or to third
parties. It does not use analytics, advertising SDKs, tracking pixels, remote
APIs, cloud storage, accounts, telemetry, or remotely hosted code. Its extension
Content Security Policy blocks network connections from extension pages.

The protected-site policy and its complementary UT1 snapshot are bundled with
the extension and evaluated locally. Grabba does not look up the selected URL
on a remote service.

## Storage and retention

Temporary recording chunks are written to Grabba's extension-private local
browser storage (the Origin Private File System) while a recording is active.
They are never uploaded and are deleted after the recording is saved,
discarded, or fails. If Chrome terminates unexpectedly, Grabba removes any
unfinished session before the next recording. Temporary recording data is not
separately encrypted by Grabba. Completed MP4 files are written to the user's
device through Chrome's download system. Grabba does not retain a
developer-accessible copy.

The downloaded MP4 is not encrypted by Grabba. Its protection depends on the
device, operating-system account, disk encryption, Downloads folder, backups,
and any service to which the user later moves or uploads the file.

Preferences and the privacy-consent version remain in Chrome local extension
storage until the user changes them, clears extension data, or removes Grabba.
They do not contain recorded media.
The microphone selection is not stored.

## Sharing and sale

Grabba does not sell, rent, share, transfer, or disclose recorded content,
browsing information, or preferences. It does not use this information for
advertising, profiling, credit decisions, or purposes unrelated to tab
recording.

Grabba's use of information received from Chrome APIs complies with the Chrome
Web Store User Data Policy, including its Limited Use requirements.

## Permissions

Grabba uses only the Chrome permissions required for its single purpose:

- `activeTab` identifies the active tab after the user explicitly opens Grabba
  from the toolbar.
- `tabCapture` captures video and optional audio from that tab.
- `offscreen` keeps local media processing active after the popup closes.
- The standard browser microphone permission is requested from the visible
  popup only when the user enables Microphone. The bundled offscreen recorder
  then opens the microphone for that recording; it does not play microphone
  sound through the speakers. No additional host or remote-service permission
  is requested.
- `downloads` saves the completed MP4 to the user's device.
- `storage` saves local preferences and the privacy-consent version.

Grabba requests no host permissions, provides no content scripts, and exposes
no externally connectable API.

## Protected sites and user responsibility

Grabba blocks known meeting, messaging, webmail, banking, payment, and other
sensitive services before capture starts. This local denylist reduces accidental
recording but cannot identify every sensitive page. Users must have authorization
to record and must follow applicable laws, notices, consent requirements, and
organizational policies.

## Security

Grabba uses Manifest V3, a restrictive local Content Security Policy, bundled
code, minimum permissions, affirmative first-use consent, protected-site
blocking, and optional Navigation protection. No software can guarantee that a
local recording will remain confidential if the device, browser profile, file
system, or downloaded MP4 is accessed by another person or program.

Security details and vulnerability-reporting guidance are available in
[SECURITY.md](SECURITY.md).

## Changes and contact

Material changes to this policy will be published with a new Grabba version and
reflected in the Chrome Web Store privacy disclosures before the changed data
practice is used.

Privacy and security questions can be submitted through the support link on
Grabba's Chrome Web Store listing or through the project's GitHub issue tracker:
<https://github.com/affernan/Grabba-privacy/issues>.

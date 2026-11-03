# RadioTunes — Kodi publication checklist

## Release status — v1.0.2

This checklist tracks the RadioTunes v1.0.2 maintenance release. Checked implementation items reflect the work carried forward from earlier versions and the v1.0.2 candidate; they are not a substitute for the final source review or on-device release tests.

### Implementation and packaging

- [x] Python 3 add-on; V1 scope remains focused on linear playback
- [x] Native Kodi player controls only; no direct Kodi database access or forced skin view
- [x] Original community artwork, with icon and fanart declared in `addon.xml`
- [x] English and French localisation, including user-facing API/network errors
- [x] Add-on state stored in its Kodi profile directory
- [x] Atomic playback-state writes and kernel-managed locking
- [x] Cross-platform state-file locking: `msvcrt.locking()` on Windows, `fcntl.flock()` on Unix-like systems
- [x] Playback-instance identification to reduce stale Now Playing metadata after restarts
- [x] Credential-specific session cache and re-authentication on HTTP 401/403
- [x] Root menu requires configured credentials
- [x] Authenticated HTTP requests do not automatically follow redirects that could expose credentials or session keys
- [x] Playlist destination validation and bounded streamed `GET` probing with stream fallback
- [x] No blind fallback to the first PLS server
- [x] Defensive normalisation of favourite/channel IDs to strings, including `list_favorites()`
- [x] Audio playback starts without waiting unnecessarily for track metadata enrichment
- [x] Retry handling for transient `/tracks/{id}` metadata failures
- [x] Useful logging for current-track / track-detail lookup failures without exposing secrets
- [x] No analytics

### Final v1.0.2 publication checks

- [ ] Confirm that `addon.xml` declares `version="1.0.2"` and no longer refers to a `v0.9 test build`
- [ ] Confirm both `strings.po` `Project-Id-Version` headers refer to v1.0.2
- [ ] Confirm `<news>` accurately describes changes in v1.0.2
- [ ] Confirm Kodi platform metadata matches supported operating systems (no obsolete Linux-only restriction)
- [ ] Verify that `LICENSE.txt` exists both at the source-repository root and inside `plugin.audio.radiotunes/`
- [ ] Verify the official package excludes `PUBLICATION_CHECKLIST.md`, README/development files, `.git`, `__pycache__` and `.pyc`
- [ ] Run `git diff --check` and review the final source diff
- [ ] Run the official Kodi addon-checker on the final add-on directory
- [ ] Test a clean installation of the final v1.0.2 package on Kodi
- [ ] Test several stations and dynamic Now Playing metadata
- [ ] Test adding/removing favourites, including integer/string channel-ID cases and contextual **Remove from favourites**
- [ ] Test stopping/restarting the same stream and rapidly changing streams (no stale metadata)
- [ ] Test stream fallback and handling of temporarily unavailable stations
- [ ] Inspect `kodi.log` for unexpected errors or leaked credentials / signed URLs
- [x] Confirm that the current v1.0.2 candidate works in Kodi (user-reported; not a clean-install release test)
- [ ] Commit and push the source release to the RadioTunes repository
- [ ] Open a dedicated v1.0.2 PR against `xbmc/repo-plugins` `omega`, based on the current upstream branch
- [ ] Verify the PR's automated checks and address reviewer comments

### Deferred work — after v1.0.2

- [ ] Validate authentication JSON type before calling `.get()`: non-dictionary JSON (`null`, `[]`, strings, etc.) must raise an explicit `AudioAddictError`; test `null`, `[]`, and `{}` while preserving required-field checks
- [ ] Explicitly test redirected playback URLs against `Kodi.Player().getPlayingFile()`, including CDN redirects and service metadata updates
- [ ] Test recovery after temporary network loss and long CDN timeouts; evaluate worst-case probe latency
- [ ] Evaluate dynamic changes to configured credentials while the background service is running (not required for v1.0.2)
- [ ] Consider entry-point simplification only if justified by an actual maintenance need; avoid unrelated refactoring

**Repository rule:** Keep this checklist at the RadioTunes source-repository root. Never include it in the official `plugin.audio.radiotunes/` package or Kodi PR.

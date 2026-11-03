# RadioTunes for Kodi — v1.0.2

Unofficial community RadioTunes / AudioAddict music add-on for Kodi.

## V1 scope

The V1 deliberately provides a simple linear-radio experience using Kodi's
native player:

- AudioAddict account login with cached/reused sessions
- current RadioTunes channel list and style filters
- Popular and New views
- favourites synchronised with the AudioAddict account
- Premium stream quality selection
- multiple stream-server fallback
- dynamic title, artist and track-cover updates
- channel artwork exposed as Kodi fanart
- English and French localisation
- no custom player UI

The RadioTunes channel directory is fetched from AudioAddict when its Kodi view
is opened; the add-on does not maintain a local channel catalogue cache.

## Architecture

`default.py`
: Kodi plugin entry point. Builds directories and resolves a selected station
  to a linear Premium stream.

`service.py`
: Lightweight background service. While a RadioTunes stream is playing, it polls
  Now Playing metadata and updates Kodi's actual playing `ListItem`, allowing
  title/artist/cover changes to propagate to Estuary, the Kodi web interface
  and remote controls such as Kore.

`resources/lib/client.py`
: AudioAddict network client. Handles authentication/session renewal, channel
  metadata, favourites and Premium PLS stream resolution.

`resources/lib/helpers.py`
: Image URL normalisation and Kodi `ListItem` helpers.

`resources/lib/state.py`
: Small state file stored only inside this add-on's Kodi profile directory.
  State writes are atomic and protected by portable, OS-managed cross-process
  file locking (`msvcrt.locking()` on Windows, `fcntl.flock()` on Unix-like systems).
  Lock files are retained; stale-lock-file deletion is not used.

## Authentication and session handling

The user's AudioAddict email/password remain Kodi settings. A successful
AudioAddict session is cached in the add-on profile and reused across restarts
only while it matches the currently configured credentials.

The add-on requires configured credentials before opening its root menu. If the
server rejects a cached session with HTTP 401/403, the add-on deletes the cached
session and performs one fresh login automatically. Authenticated requests do not
automatically follow HTTP redirects, to avoid forwarding credentials or session
headers to another origin.

## Premium quality mapping

- Medium → `premium_medium` → AAC-HE 64 kb/s
- High → `premium` → AAC 128 kb/s
- Ultra → `premium_high` → MP3 320 kb/s

Changing this setting affects the next time a stream is opened. V1 does not
interrupt/restart an already playing stream when the quality setting changes.

## Stream-server fallback

RadioTunes returns several candidate streaming servers in its PLS playlist.
The add-on probes candidates using a lightweight streamed `GET`, rather than
relying on `HEAD`, because some streaming nodes may reject `HEAD` while still
being playable. If no candidate is reachable, playback fails cleanly instead of
blindly selecting the first playlist entry. Playlist destinations and HTTP
redirect targets are checked before probing, including checks against local and
private network destinations. As with any DNS-based check, this cannot guarantee
that a downstream player will never follow a different redirect later.

## Playback metadata and state

The background service polls Now Playing metadata approximately every 15 seconds
and enriches track details when available. Playback instances are distinguished
so that delayed updates from an earlier playback cannot overwrite the current
stream's metadata, including when the same station is restarted. Transient track
detail errors can be retried after a cooldown without blocking audio playback.

## Deferred robustness work

- Validate that authentication JSON responses are dictionaries before accessing
  their fields (including handling JSON `null`, `[]` and `{}` explicitly).
- Test stream redirects against `Kodi.Player().getPlayingFile()` under real CDN
  conditions.
- Exercise temporary network loss, stream recovery and worst-case probe latency.
- Dynamic handling of credential changes while the background service is active
  is out of scope for the current release.

## V2 candidates

- per-track progress using a track-by-track playback engine
- apply a quality change without manually stopping/restarting playback
- optional interactive RadioTunes functions (Like / Dislike / Next) only if there
  is enough user value to justify the additional Kodi UI complexity
- sleep timer
- improved behaviour/recovery after a temporary network loss
- further simplification of Kodi entry points by moving logic into `resources/lib/`

## Publication

- Maintainer: Édouard Duliège
- Source: https://github.com/edouardduliege/kodi-addon-radiotunes
- License: GPL-3.0-or-later
- Current release: 1.0.2
- Platform: Kodi Omega / Python 3 (cross-platform locking; Windows playback still requires platform-specific validation)

This is an unofficial community add-on. It is not affiliated with, endorsed by,
or supported by RadioTunes or AudioAddict.

The add-on uses original community artwork and does not redistribute the
official RadioTunes logo.

For Kodi repository submission, `LICENSE.txt` must be present inside the
`plugin.audio.radiotunes` folder as well as being kept at repository root.

## Changelog

### 1.0.2

Maintenance release focused on stability, security and consistent behaviour
across the AudioAddict add-on family.

- Portable playback-state file locking on Windows and Unix-like systems.
- Playback-instance tracking to avoid stale Now Playing metadata.
- Safer authentication and authenticated HTTP redirect handling.
- Playlist URL and stream redirect checks before probing.
- Retry of transient track-detail enrichment errors with a cooldown.
- Normalised favourite/channel IDs, including integer and string API IDs.
- Improved logging and stream handling without delaying audio startup.

### 1.0.0

Initial public-release version of the V1 feature set.

### 0.9.0

Initial pre-release test build.

- Linear RadioTunes playback through Kodi's native player
- AudioAddict account authentication with cached session reuse
- channel browsing, style filters, Popular and New views
- favourites synchronisation
- Premium stream quality selection
- multiple stream-server fallback
- dynamic Now Playing title, artist and artwork updates
- English and French localisation
- session renewal on HTTP 401/403
- cached-session fingerprint tied to the configured credentials
- lightweight streamed `GET` probes for stream-server fallback
- atomic playback-state writes with OS-managed file locking
- reduced unnecessary Now Playing API calls
- original community artwork clearly distinguishing the add-on from an official release

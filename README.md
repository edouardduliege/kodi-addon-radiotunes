# RadioTunes for Kodi — v0.9.0

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
  State writes are atomic and protected by a cross-process lock with stale-lock
  recovery.

## Authentication and session handling

The user's AudioAddict email/password remain Kodi settings. A successful
AudioAddict session is cached in the add-on profile and reused across restarts
only while it matches the currently configured credentials.

The add-on requires configured credentials before opening its root menu. If the
server rejects a cached session with HTTP 401/403, the add-on deletes the cached
session and performs one fresh login automatically.

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
blindly selecting the first playlist entry.

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
- Current state: v0.9.0 pre-release test build

This is an unofficial community add-on. It is not affiliated with, endorsed by,
or supported by RadioTunes or AudioAddict.

The add-on uses original community artwork and does not redistribute the
official RadioTunes logo.

For Kodi repository submission, `LICENSE.txt` must be present inside the
`plugin.audio.radiotunes` folder as well as being kept at repository root.

## Changelog

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
- atomic playback-state writes with stale-lock recovery
- reduced unnecessary Now Playing API calls
- original community artwork clearly distinguishing the add-on from an official release

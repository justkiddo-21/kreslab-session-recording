# KResLab Session Recording

A fork of [Cockpit](http://www.cockpit-project.org)'s
[session-recording](https://github.com/Scribery/cockpit-session-recording) module,
patched to browse [tlog](https://github.com/Scribery/tlog) recordings that were
shipped in from *other* hosts via `systemd-journal-remote`, instead of only the
local machine's own journal.

All credit for the original module, UI, and player goes to
[Scribery](https://github.com/Scribery) / Red Hat — see [LICENSE](LICENSE)
(LGPL-2.1, unchanged from upstream). This fork exists solely to fix one
limitation for a centralized-log-server deployment; everything else is
untouched upstream code.

## What's different from upstream

Cockpit's built-in `journal.journalctl()` helper only ever queries the local
system journal by default. That's fine when tlog and Cockpit run on the same
box the user logs into — but it breaks a *centralized* setup where many edge
hosts run tlog and ship their recordings over the network to one dedicated
log server, and Cockpit runs only on that log server to browse everything in
one place. The remote recordings are real, readable files, but invisible to
this module and to plain `journalctl` alike, because they live outside the
journal's default discovery paths and `--merge` only merges journals laid
out in the standard per-machine-ID directory structure, not arbitrary flat
files.

The fix is two one-line changes in
[`src/recordings.jsx`](src/recordings.jsx): both places that build the
options object for `journal.journalctl()` now also set
`directory: "/var/log/journal/remote"`, which `journal.journalctl()` already
supports (it maps straight to `journalctl --directory=`). That's the
directory `systemd-journal-remote` writes into by default — if you're
serving recordings from a different path, bind-mount it there, or search
this repo for `/var/log/journal/remote` and change both occurrences.

## Building

Needs Node.js >= 16, `make`, `gettext`. Fetches upstream Cockpit's `pkg/lib`
at build time (network required):

```
make
```

Produces the built module in `dist/`.

## Installing

Copy `dist/` over the existing package on your log server (this replaces the
distro-packaged version at the same path, so it keeps working with the
existing Cockpit menu entry):

```
sudo cp -r dist/* /usr/share/cockpit/session-recording/
sudo systemctl restart cockpit.socket
```

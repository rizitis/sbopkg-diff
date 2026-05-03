# sbopkg-diff — HowTo

## What is it?

`sbopkg-diff` is a companion tool for `sbopkg`. It allows you to inspect the
latest upstream changes for any SlackBuilds.org package directly from the
GitHub API, without performing a full repository sync. It works with both the
official SBo repository (SlackBuildsOrg) and Ponce's `SBo-git` repo for current.

## Dependencies

- `jq` — for JSON parsing
- `sbopkg` - :)

**jq** is available on SlackBuilds.org for Slackware-15.0 and already installed on current.

**sbopkg** you know were to find it...

## Installation

```bash
cp sbopkg-diff /usr/local/bin/sbopkg-diff
chmod +x /usr/local/bin/sbopkg-diff
```

## Usage

```bash
sbopkg-diff <package-name>
```

### Examples

```
sbopkg-diff electron-bin
>>> Config: REPO_NAME=SBo  REPO_BRANCH=15.0
>>> Local tree: /var/lib/sbopkg/SBo/15.0
>>> API: https://api.github.com/repos/SlackBuildsOrg/slackbuilds

>>> Found: development/electron-bin

=== Recent commits ===
2026-04-28T06:09:31Z  fe1c4b8  development/electron-bin: Updated for version 41.3.0
2026-04-25T07:28:37Z  b6306a6  development/electron-bin: Updated for version 41.2.1.

=== Version check ===
Installed : 41.3.0
SBo latest: 41.3.0
>>> UP TO DATE

=== Latest diff ===
--- development/electron-bin/electron-bin.SlackBuild
@@ -29,7 +29,7 @@ cd $(dirname $0) ; CWD=$(pwd)
 
 PRGNAM=electron-bin
 PKGNAM=electron
-VERSION=${VERSION:-41.2.1}
+VERSION=${VERSION:-41.3.0}
 BUILD=${BUILD:-1}
 TAG=${TAG:-_SBo}
 PKGTYPE=${PKGTYPE:-tgz}
--- development/electron-bin/electron-bin.info
@@ -1,10 +1,10 @@
 PRGNAM="electron-bin"
-VERSION="41.2.1"
+VERSION="41.3.0"
 HOMEPAGE="https://www.electronjs.org/"
 DOWNLOAD="UNSUPPORTED"
 MD5SUM=""
-DOWNLOAD_x86_64="https://github.com/electron/electron/releases/download/v41.2.1/electron-v41.2.1-linux-x64.zip"
-MD5SUM_x86_64="dc9603fceb6caafab58c277b177a9941"
+DOWNLOAD_x86_64="https://github.com/electron/electron/releases/download/v41.3.0/electron-v41.3.0-linux-x64.zip"
+MD5SUM_x86_64="e9929581d7ec4c0ab86e1185c0d3effb"
 REQUIRES=""
 MAINTAINER="Antonio Leal"
 EMAIL="antonioleal@yahoo.com"
```

```
sbopkg-diff vlc
>>> Config: REPO_NAME=SBo-git  REPO_BRANCH=current
>>> Local tree: /var/lib/sbopkg/SBo-git
>>> API: https://api.github.com/repos/Ponce/slackbuilds

>>> Found: multimedia/vlc

=== Recent commits ===
2026-05-03T08:58:15Z  631258e  20260503.1 global branch merge.
2026-01-22T06:58:17Z  1c6a49a  multimedia/vlc: Updated for version 3.0.23

=== Version check ===
Installed : 3.0.23
SBo latest: 3.0.23
>>> UP TO DATE

=== Latest diff ===
--- multimedia/vlc/vlc.SlackBuild
@@ -10,6 +10,7 @@
 # Copyright (c) 2022  Bill Kirkpatrick, Bay City, Texas, USA
 # Copyright (c) 2023  Tim Dickson, Scotland
 # Copyright (c) 2024  Steven Voges <Oregon, USA>
+# Copyright (c) 2026  Antonio Leal, Porto Salvo, Oeiras, Portugal
 # All rights reserved.
 #
 #   Redistribution and use of this script, with or without modification is
@@ -48,8 +49,8 @@
 cd $(dirname $0) ; CWD=$(pwd)
 
 PRGNAM=vlc
-VERSION=${VERSION:-3.0.20}
-BUILD=${BUILD:-2}
+VERSION=${VERSION:-3.0.23}
+BUILD=${BUILD:-1}
 TAG=${TAG:-_SBo}
 PKGTYPE=${PKGTYPE:-tgz}
 
--- multimedia/vlc/vlc.info
@@ -1,10 +1,10 @@
 PRGNAM="vlc"
-VERSION="3.0.20"
+VERSION="3.0.23"
 HOMEPAGE="https://www.videolan.org/vlc/"
-DOWNLOAD="https://get.videolan.org/vlc/3.0.20/vlc-3.0.20.tar.xz"
-MD5SUM="e8337fcd2df92f3901dad091fb85f545"
+DOWNLOAD="https://get.videolan.org/vlc/3.0.23/vlc-3.0.23.tar.xz"
+MD5SUM="ebc3f0d0a94785fd2b2df4087516938e"
 DOWNLOAD_x86_64=""
 MD5SUM_x86_64=""
 REQUIRES="libass libdc1394 libdvbpsi libmpeg2 lua portaudio twolame gsm libtar libkate faac libdca libshout avahi projectM jack libsidplay2 zvbi faad2 libavc1394 libmodplug musepack-tools vcdimager dirac gnome-vfs live555 rtmpdump libdvdcss schroedinger libminizip chromaprint x264 x265 libnfs protobuf3"
-MAINTAINER="Steven Voges"
-EMAIL="svoges.sbo@gmail.com"
+MAINTAINER="Antonio Leal"
+EMAIL="antonioleal@yahoo.com"

```
> If you command as user (not root) it read /etc/sbopkg/sbopkg.conf only for your sbopkg repo url
> If you run as root it first look at /root/.sbopkg.conf and only if not found look at /etc/sbopkg/sbopkg.conf
> So be sure you have setup a valid sbopkg.conf on both cases if you use /root/.sbopkg.conf.

## What it does

1. **Reads your sbopkg configuration** — checks `/root/.sbopkg.conf` first,
   falls back to `/etc/sbopkg/sbopkg.conf`. Automatically detects `REPO_NAME`
   and `REPO_BRANCH`, and selects the correct GitHub API endpoint accordingly.

2. **Finds the package category locally** — uses your existing local SBo tree
   (e.g. `/var/lib/sbopkg/SBo-git`) to determine the category (e.g.
   `development`, `multimedia`) without any API call.

3. **Fetches recent commits** — retrieves the last 5 commits that touched the
   package path, displays the 2 most recent.

4. **Checks installed version** — compares the version in `/var/lib/pkgtools/packages/`
   against the latest version found in the commit messages, and reports whether
   an update is available.

5. **Shows the latest diff** — fetches and displays the unified diff of the
   most recent package-specific commit, filtered to show only files belonging
   to the requested package.

## Configuration auto-detection

| Config file | REPO_NAME | Local tree | API used |
|---|---|---|---|
| `/root/.sbopkg.conf` | `SBo-git` | `/var/lib/sbopkg/SBo-git` | Ponce/slackbuilds |
| `/etc/sbopkg/sbopkg.conf` | `SBo` | `/var/lib/sbopkg/SBo/15.0` | SlackBuildsOrg/slackbuilds |

## su requirement

`sbopkg-diff` must be run as root **only** when your active
configuration is `/root/.sbopkg.conf`, since that file is readable by root
only. If your configuration lives in `/etc/sbopkg/sbopkg.conf` only, then you can run
it as a regular user.

## GitHub API rate limiting

The tool uses exactly **2 API calls** per invocation:

1. Fetch recent commits for the package path
2. Fetch the diff for the latest package-specific commit

The GitHub API allows **60 unauthenticated requests per hour**, which is more
than sufficient for checking 1–3 packages at a time — the intended use case.

## Known limitations

- **Mass commits**: Some historical commits (e.g. "All: Support
  $PRINT_PACKAGE_NAME") touched every package in the repository. The GitHub
  API returns a maximum of 300 files per commit, so packages whose path falls
  outside the first 300 alphabetically will show no diff. In that case,
  `sbopkg-diff` will display an informative message and suggest using
  `git log` (in /var/lib/sbopkg/repo_path) locally instead.

- **Global branch merges**: Ponce's repo includes periodic global merge
  commits (e.g. `20260503.1 global branch merge`). These are automatically
  skipped when looking for the relevant diff.

- **Version detection**: Version numbers are extracted from commit messages.
  If a commit message does not follow the standard `Updated for version X.Y.Z`
  format, the version comparison may be inconclusive.

## Security

`sbopkg-diff` is designed to run safely as root or not:

- Config files are **never sourced** — only specific keys are parsed with `grep`
- Config files are checked for **symlink attacks** before reading
- All **input is sanitized** with strict character whitelists
- **REPO_NAME** and **REPO_BRANCH** are validated with regex before use
- The local tree is checked for **symlinks** (`find ! -type l`)
- All **SHA values** from the API are validated as 40-character hex strings
  before being used in subsequent requests
- `curl` enforces **HTTPS only**, **TLS 1.2 minimum**, request timeouts,
  rate limits, and maximum file sizes
- API output rendered to the terminal is passed through `cat -v` to neutralize
  any **terminal escape sequence injection** in commit messages

## Fallback for packages with no diff available

```
>>> No package-specific commit found in last 5 commits.
>>> Run: git log --oneline -- system/cpulimit (in local tree)
```

Use the suggested `git log` command in your local SBo tree for a full history.

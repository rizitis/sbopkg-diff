# sbopkg-diff — HowTo

## What is it?

`sbopkg-diff` is a companion tool for `sbopkg`. It allows you to inspect the latest upstream changes for any SlackBuilds.org package directly from the GitHub API **before** you perform a repository sync.

It works seamlessly with both the official SBo repository (`SlackBuildsOrg`) and Ponce's `SBo-git` repository for Slackware current.

## Dependencies

- **jq** — for JSON processing (installed by default on Slackware current).
- **sbopkg** — the best SlackBuilds manager.

## Installation
```
# Copy the script to your path
cp sbopkg-diff /usr/local/bin/sbopkg-diff
chmod +x /usr/local/bin/sbopkg-diff
```

## Usage
```
sbopkg-diff <package-name>
```

## Examples

#### Checking a package on 15.0:

```
sbopkg-diff electron-bin
>>> Config: REPO=SBo BRANCH=15.0
>>> Found locally: development/electron-bin

=== Version check (Live from GitHub) ===
Installed : 41.3.0
SBo latest: 41.3.0
>>> UP TO DATE

=== Recent commits (15.0) ===
2026-04-28T06:09:31Z  fe1c4b8  development/electron-bin: Updated for version 41.3.0
2026-04-25T07:28:37Z  b6306a6  development/electron-bin: Updated for version 41.2.1.
2026-04-17T07:30:12Z  cfcb74c  development/electron-bin: Updated for version 41.2.0

=== Latest diff for electron-bin ===
index c39a61d7971..cad8b75980f 100644
--- a/development/electron-bin/electron-bin.SlackBuild
+++ b/development/electron-bin/electron-bin.SlackBuild
@@ -29,7 +29,7 @@ cd $(dirname $0) ; CWD=$(pwd)

 PRGNAM=electron-bin
 PKGNAM=electron
-VERSION=${VERSION:-41.2.1}
+VERSION=${VERSION:-41.3.0}
 BUILD=${BUILD:-1}
 TAG=${TAG:-_SBo}
 PKGTYPE=${PKGTYPE:-tgz}
```

#### Checking a package on current:

```
sbopkg-diff zenity
>>> Config: REPO=SBo-git BRANCH=current
>>> Found locally: desktop/zenity

=== Version check (Live from GitHub) ===
Installed : 4.0.2
SBo latest: 4.0.2
>>> UP TO DATE

=== Recent commits (current) ===
2026-05-03T08:58:15Z  631258e  20260503.1 global branch merge.
2024-11-02T12:19:43Z  c8bb093  desktop/zenity: Updated for version 3.44.5
2024-05-25T04:47:43Z  f4df306  desktop/zenity: Updated for version 3.44.4.

=== Latest diff for zenity ===
index 125286c90e5..79ad8648343 100644
--- a/desktop/zenity/README
+++ b/desktop/zenity/README
@@ -4,7 +4,8 @@ similar to the classic `dialog` program, but with a GUI interface.

 OPTIONAL DEPENDENCIES:

-* webkit2gtk-4.1
+* webkit2gtk-6.0 (ie, webkitgtk for gtk4; NOT yet part of sbo-ponce as
+  of 2024-07-28)

   To enable: pass `WEBKITGTK=true` as an option to the slackbuild.
   If this is not specified, it will default to `false`.
index c32ae1081eb..5cd4c8868ea 100644
--- a/desktop/zenity/zenity.info
+++ b/desktop/zenity/zenity.info
@@ -1,10 +1,10 @@
 PRGNAM="zenity"
-VERSION="3.44.5"
+VERSION="4.0.2"
 HOMEPAGE="https://gitlab.gnome.org/GNOME/zenity"
-DOWNLOAD="https://download.gnome.org/sources/zenity/3.44/zenity-3.44.5.tar.xz"
-MD5SUM="69f4a4fdce7217231207019a6e27636b"
+DOWNLOAD="https://download.gnome.org/sources/zenity/4.0/zenity-4.0.2.tar.xz"
+MD5SUM="08ba19bb3fe5c180402690d5c40c6cc3"
 DOWNLOAD_x86_64=""
 MD5SUM_x86_64=""
-REQUIRES=""
+REQUIRES="libadwaita"
 MAINTAINER="Logan Rathbone"
 EMAIL="poprocks@gmail.com"
```
### What it does

1.    **Configuration Auto-detection**: It reads
- a) /root/.sbopkg.conf (if you are root and if exist its first choise)
- b) /etc/sbopkg/sbopkg.conf.  (if  `/root/.sbopkg.conf`  exist then you must have same REPO_NAME and REPO_BRANCH on both)<br>
It automatically detects your REPO_NAME and REPO_BRANCH to select the correct GitHub API endpoint.

2.   ** Category Discovery**: Uses your local tree to find the package category (e.g., system, multimedia) so you don't have to provide it.

3.    **Live Version Check**: Fetches the .info file directly from the GitHub branch raw content. This bypasses API caching and ensures you see the real version before you sync.

4.    **Intelligent Diffing**: Fetches the unified diff of the most recent commit and uses sed to isolate changes strictly related to your package, filtering out unrelated changes from global merges.

4.    Version Comparison: Uses sort -V logic to compare versions, correctly identifying if SBo is newer or if your installed version is ahead.

---

### Security & Reliability

#### sbopkg-diff is designed with defensive programming:

-   No Sourcing: Configuration files are never executed (sourced); they are parsed safely with grep and cut.

-    Symlink Protection: Refuses to read config files or local trees that are symlinks to prevent redirection exploits.

-    Input Sanitization: Package names are whitelisted to prevent command injection.

-    Connection Hardening: Enforces HTTPS, TLS 1.2+, and strict timeouts.

-    Terminal Safety: Passes remote commit messages through cat -v to neutralize potential terminal escape sequence injections.

---

#### GitHub API Rate Limiting

The tool is optimized to use only 2-3 API calls per package check. The GitHub API allows 60 unauthenticated requests per hour, which is ample for standard usage.


#### Known Limitations

    Mass Commits: If a commit touches more than 300 files (GitHub's API limit), the diff may not be available via API. In such cases, the script will suggest a local git log command.

    Case Sensitivity: The package name must match the case used in the SBo repository (usually lowercase).

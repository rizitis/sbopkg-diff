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

```bash
sbopkg-diff electron-bin
sbopkg-diff vlc
sbopkg-diff sl
sbopkg-diff xephem
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

# RxRescue update manifest

`latest.json` is the canonical source of truth for the in-app update
notifier. The running app polls
`https://raw.githubusercontent.com/SkyboundSystems/rxrescue-manifest/main/latest.json`
once per day and surfaces a dashboard banner when a strictly newer
version (by semver-then-build-number) is available.

## Schema

| field                | type    | required | meaning                                      |
|----------------------|---------|----------|----------------------------------------------|
| `latestVersion`      | string  | yes      | Pure semver, e.g. `"0.17.0"` (no `-rc1` suffixes — `compareVersions` falls back to lexical for non-numeric segments) |
| `latestBuildNumber`  | int     | yes      | Used as a tiebreak when versions are equal   |
| `releaseDate`        | string  | yes      | ISO `YYYY-MM-DD`                             |
| `downloadUrl`        | string  | yes      | Public URL the user's browser opens when they tap Download. OneDrive Personal share URL (`https://1drv.ms/u/c/...`) for `RxRescue-windows-latest.zip` — serves an HTML preview page that has a Download button on it. |
| `releaseNotes`       | string  | yes      | Free-text plain. Banner truncates to 2 lines on the dashboard. |
| `minSupportedVersion`| string  | no       | Reserved; not yet enforced by the app.       |

## Releasing a new version (post-v0.17 stable-filename pattern)

The OneDrive build artifacts use STABLE FILENAMES (`RxRescue-windows-latest.zip`,
`RxRescue-android-latest.apk`) and the OneDrive share URLs for those
files are stable forever. Each release overwrites the file content
in place, the share URL stays the same. So `downloadUrl` in this
manifest also stays the same across releases — what changes per
release is `latestVersion`, `latestBuildNumber`, `releaseDate`, and
`releaseNotes`.

1. **Build + publish to OneDrive in one shot:**
   ```powershell
   flutter build windows --release
   .\tools\package_windows.ps1 -Publish
   ```
   The `-Publish` flag overwrites `RxRescue-windows-latest.zip` in the
   OneDrive Builds folder. The version-tagged copy still goes to
   `dist/` for archival ("which build did Alex install?"
   reconstruction).

2. **For Android:** the trial uses Firebase App Distribution
   (`tools/release_to_firebase.ps1`). Manifest `downloadUrl` is
   currently Windows-only — Android users get updates through Firebase
   directly, not through this manifest. When we add cross-platform
   manifest support, `downloadUrl` will split into
   `downloadUrl.windows` / `downloadUrl.android` keys.

3. **Bump this manifest:**
   - Edit `latest.json` — bump `latestVersion`, `latestBuildNumber`,
     `releaseDate`, and write release notes.
   - `git add latest.json && git commit -m "Bump to v<X>.<Y>.<Z>+<N>" && git push`
   - The CDN propagates within ~60s; installed apps will see the new
     version on their next daily poll.

## Why this repo is separate from the main rxrescue repo

The main rxrescue repo is private — `raw.githubusercontent.com`
returns 404 on private repos without an auth token, which we can't
ship in the binary. This single-file manifest repo is public so the
running app can fetch `latest.json` over plain HTTPS without auth.
The manifest contains nothing sensitive (version numbers, an
OneDrive share URL, plain-text release notes); a public posture is
fine.

## Why this repo doesn't host the build artifacts

Repo size. The Windows zip is ~17 MB and the APK is ~80 MB. Hosting
multiple versions of those in git history would inflate the repo
quickly. OneDrive absorbs that storage at no repo cost. If we ever
want a one-click download (no preview page), GitHub Releases on a
public repo is the cleaner answer.

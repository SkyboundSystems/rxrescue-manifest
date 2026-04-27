# RxRescue update manifest

`latest.json` is the canonical source of truth for the in-app update
notifier. The running app polls the `raw.githubusercontent.com` URL of
this file once per day and surfaces a dashboard banner when a strictly
newer version (by semver-then-build-number) is available.

## Schema

| field                | type    | required | meaning                                      |
|----------------------|---------|----------|----------------------------------------------|
| `latestVersion`      | string  | yes      | Pure semver, e.g. `"0.16.1"` (no `-rc1` suffixes — `compareVersions` falls back to lexical for non-numeric segments) |
| `latestBuildNumber`  | int     | yes      | Used as a tiebreak when versions are equal   |
| `releaseDate`        | string  | yes      | ISO `YYYY-MM-DD`                             |
| `downloadUrl`        | string  | yes      | Public URL the user's browser opens when they tap Download. Currently OneDrive Personal share URLs (`https://1drv.ms/u/c/...`) — they serve an HTML preview page that has a Download button. |
| `releaseNotes`       | string  | yes      | Free-text plain. Banner truncates to 2 lines on the dashboard. |
| `minSupportedVersion`| string  | no       | Reserved; not yet enforced by the app.       |

## Releasing a new version

1. Build the new Windows zip + Android APK locally (`flutter build windows --release`, `flutter build apk --release`, `tools/package_windows.ps1`).
2. Upload the zip to the OneDrive Builds folder. Get a per-file share link (Anyone with link, Can view) for the new zip.
3. Update `manifest/latest.json` in this repo:
   - Bump `latestVersion` and `latestBuildNumber`.
   - Update `releaseDate` to today.
   - Replace `downloadUrl` with the new share URL.
   - Update `releaseNotes` (keep it concise — banner shows ~2 lines).
4. Commit and push to `main`. The next time any installed app polls the
   manifest (≤24h later), users see the update banner.

## Why not host the zip in this repo too?

Repo size. The Windows zip is ~17 MB and the APK is ~80 MB. Hosting
multiple versions of those would inflate the git history quickly. OneDrive
absorbs that storage with no repo cost. If we ever want a one-click
download (no preview page), GitHub Releases is the right answer — clean
direct-download URLs and asset uploads via `gh release create`.

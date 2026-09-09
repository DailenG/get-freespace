# Get-FreeSpace - Dev Notes

## Provenance

Consolidated 2026-09-09 from the GitHub repo (authoritative, newest) plus files scattered
across the SAPIEN PowerShell Studio workspace at
`C:\Syncs\Resilio\Code\Local\PowerShell Studio` (`Projects\`, `Files\`, `Builds\`).
The scattered `get-freespace.ps1` there matched commit `e02b1d9` exactly, so no source was
lost. The only unique recovery was the unfinished module rewrite. Those workspace folders
have since been deleted; everything salvaged that does not belong in a public repo lives
in `legacy/`, which is gitignored. Build/installer identity is in `docs/BUILD.md`.

## Downstream fork

`pesengineers/get-freespace` is a fork used by a client via RMM automation. It is 2
commits ahead of upstream and the entire delta is the default `-Source` value pointing at
`pesengineers/FreespacePaths` instead of `DailenG/FreespacePaths`. Keep that fork
mergeable: do not restructure the parameter block without checking it, and treat the two
`FreespacePaths` repos as one schema.

## Engine constraint

`Get-FreeSpace.ps1` targets Windows PowerShell 5.1, not PowerShell 7. Two hard
dependencies drove that:

- `Out-GridView` for the selection UI (absent from PS7 unless `Microsoft.PowerShell.ConsoleGuiTools` or the WPF gridview module is present).
- `Add-Type -AssemblyName PresentationFramework` for the completion popup.

Commit `7332b3b` reverted a PS7 packaging attempt for exactly this reason. Any move to
PS7 must replace both surfaces first.

## Cleanup list is a separate repo

Path data lives in `DailenG/FreespacePaths` (`paths.json`) and is fetched at runtime via
`-Source`. `paths.json` here is a snapshot. Adding a cleanup target means a PR there, not
here. Schema: `name`, `path` (wildcards allowed), `description`, `aged` (minimum age in
days by `LastAccessTime`, `0` disables).

## Known issues worth fixing in a rewrite

- Freed-space math round-trips through formatted strings (`"1.23 GB"` parsed back to
  bytes) in both the script and the legacy module. Track raw bytes; format only at output.
- `Get-FolderSize` enumerates every file recursively twice per path (before and after
  deletion). Slow on large temp trees.
- Errors are swallowed with `-ErrorAction SilentlyContinue` everywhere, so locked or
  access-denied files are invisible in the report.
- `Invoke-WebRequest` failure only calls `Write-Error` and then falls through into
  `ConvertFrom-Json` on `$null`.
- No `SupportsShouldProcess` in the shipped script, so there is no `-WhatIf`.
- `aged` retention is folder-level and effectively inert. Verified 2026-09-09 on a
  synthetic tree: because every list entry ends in a backslash, `Get-ChildItem -Path
  '...\Journals\' -Directory` returns the matched `Journals` container itself, one per
  user profile, not its children. So `aged` compares the LastAccessTime of that container,
  which Revit keeps fresh, and `Remove-FolderContents` then deletes the entire container
  recursively with no age check at all. Net effect: `aged` either skips a whole path or
  purges it wholesale, and never protects recent files inside a path. This is the gap
  behind the "engineer could not read Tuesday's journals on Wednesday morning" complaint.
  A real fix filters files by timestamp at deletion time, not directories at scan time.
- Unelevated runs silently see only the current user's profile (`Resolve-Path` on
  `C:\Users\*` throws access denied on other profiles, swallowed by
  `-ErrorAction SilentlyContinue`). There is no elevation check or warning.

## Packaging

Read `docs/BUILD.md` before touching packaging. The MSI ProductGUID/UpgradeGUID must stay
stable so existing installs upgrade rather than side-install.

## Commit style

- Atomic commits per logical change, one concern per commit.
- No co-author attribution lines.
- Imperative subject line, body explains the why.

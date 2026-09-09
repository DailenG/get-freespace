# Get-FreeSpace

Open source utility to free up temporary space consumed by Autodesk and Windows.

It scans a list of well known cache/temp locations, reports how much space each one
holds, lets you pick what to purge (grid view, console prompt, or unattended), then
reports how much was actually reclaimed.

## Usage

```powershell
# Interactive: grid view selection (default)
.\Get-FreeSpace.ps1

# Console prompt instead of grid view
.\Get-FreeSpace.ps1 -NoDisplay

# Unattended: purge everything on the list, no prompts, no popup
.\Get-FreeSpace.ps1 -Force

# Use a different cleanup list (URL or local path reachable by Invoke-WebRequest)
.\Get-FreeSpace.ps1 -Source "https://raw.githubusercontent.com/DailenG/FreespacePaths/main/paths.json"
```

| Parameter | Type | Effect |
| --- | --- | --- |
| `-Force` | switch | Purge every path in the list without prompting; suppresses the completion popup. |
| `-NoDisplay` | switch | Replace `Out-GridView` selection with a console Y/N prompt. |
| `-Source` | string | Cleanup list to load. Defaults to the `DailenG/FreespacePaths` `paths.json` on GitHub. |

Run elevated. Most of the target paths span all user profiles plus `C:\Windows\Temp`.

## Cleanup list

The path list is data, not code. It lives in
[DailenG/FreespacePaths](https://github.com/DailenG/FreespacePaths) and is fetched at
runtime; [`paths.json`](paths.json) in this repo is a snapshot of that list for offline
use and reference.

Each entry:

```json
{
  "name": "Revit Journals",
  "path": "C:\\Users\\*\\AppData\\Local\\Autodesk\\Revit\\*Revit*202*\\Journals\\",
  "description": "Revit journal files for all users and 202x versions",
  "aged": 0
}
```

`aged` is a minimum age in days: only child directories whose `LastAccessTime` is older
than `now - aged` are considered. `0` means no age filter.

`C:\Windows\Temp\` is appended by the script itself and is not part of the list.

## Repository layout

| Path | What it is |
| --- | --- |
| `Get-FreeSpace.ps1` | The shipped script. Windows PowerShell 5.1 compatible. |
| `paths.json` | Snapshot of the FreespacePaths cleanup list. |
| `get-freespace.ps1.psbuild` | SAPIEN PowerShell Studio packaging settings (exe + MSI). |
| `get_freespace.ico` | Icon used by the packaged exe/MSI. |
| `docs/BUILD.md` | Packaging identity: GUIDs, versions, engine, installer settings. |
| `legacy/` | Local-only archive (gitignored, never published): unfinished PowerShell Studio module rewrite, PowerShell Studio project scaffolding, and the last shipped `get-freespace.exe`/`.msi`. |

## Status

Last shipped build: `1.3.5.0` (SAPIEN PowerShell Studio, Windows PowerShell engine,
64-bit, MSI installs to `[ProgramFiles]\Get-FreeSpace` for all users, elevated).
See [`docs/BUILD.md`](docs/BUILD.md) before changing packaging.

## Author

Dailen Gunter

## License

See [LICENSE](LICENSE).

# Termux-WiFi-map

## About
Termux-WiFi-map is a comprehensive Bash utility for Termux on Android that scans nearby Wi-Fi access points (APs), records their metadata and optionally geolocates them using Termux location providers. It is intended for network hobbyists, security researchers, and anyone who needs a lightweight, non-root Wi-Fi mapping tool on mobile devices.

This README documents how to install, run and configure the tool, explains modes and options in detail, and describes output formats and internal behaviour.

## Features
- Passive Wi-Fi scanning via `termux-wifi-scaninfo`.
- GPS/location integration via `termux-location` (supports providers: gps, network, passive).
- Save/export scans in multiple formats: JSON (single-line), pretty JSON (PJSON), JSONL (newline-delimited JSON), CSV, KML.
- Cache of collected APs for later export or analysis (default: $HOME/.cache/BuriXon-code/wifi_scan_cache.jsonl).
- Continuous scanning mode with adjustable delay (supports "none" for 0s / fastest possible).
- Vibration feedback control (can be disabled).
- File validation mode: checks structure of saved files for correctness.
- Robust error handling: automatic Termux API restart routine (unless configured to exit).
- Lightweight: written in POSIX/Bash and relies on Termux utilities — no root required.

## Quick overview
- **Runner** (`--run`): triggers Wi-Fi scan(s), appends results to the internal cache (JSONL). Can run once or in continuous loop.
- **Saver** (`--save`): reads the cache and writes it to a chosen export format/file.
- **Cache manager** (`--cache`): view, count or purge the internal cache.
- **Validator** (`--check`): checks a provided file for structure/format errors.
- **Helpers**: version info, about, and help output.

## Output formats
- **JSON** — single-line JSON object or array (compact).
- **PJSON** — pretty-printed, human-friendly JSON (useful for inspection).
- **JSONL** — newline-delimited JSON; each line is one JSON object (ideal for streaming / incremental writes).
- **CSV** — flattened comma-separated values for spreadsheets.
- **KML** — geographic points suitable for Google Earth / GIS tools.

## Installation
See commands listed under "Commands — installation & examples" (below this README block).  
Notes:
- You must install Termux and Termux:API support (`termux-api` package) before running scans.
- `jq` is required for JSON processing.
- The script creates a cache directory under `$HOME/.cache/BuriXon-code` by default.

## Usage
Run the script with a **mode** and its **options**.  
Modes control the main action (scan/save/cache/validate), options tune behaviour (provider, delay, filename, format, etc.).

### Modes
| Mode | Aliases | Purpose |
|------|---------|---------|
| Run  | `R` `-R` `--run` | Perform Wi-Fi scan(s) and append results to cache. Can be single-shot or continuous. |
| Save | `S` `-S` `--save` | Export cached data to a file in chosen format (json/pjson/jsonl/csv/kml). Requires `--name` and `--format`. |
| Cache | `C` `-C` `--cache` | Cache management: list (`--list`), count (`--count`), purge (`--purge`). |
| Check/Verify | `V` `-V` `--check` | Validate structure of a provided file (pass path as parameter). |
| Help/Info | `-h` `--help` `-v` `--version` `-i` `--version-info` | Show help, version or extended about info. |

### Options
#### Run options
- `-p|--provider [gps|network|passive]`  
  Sets location provider for `termux-location`. Default: `gps`.  
  - `gps` — uses device GPS (most accurate outdoors).  
  - `network` — cell/Wi-Fi based (faster, less accurate).  
  - `passive` — low-power passive updates.

- `-d|--delay [seconds|none]`  
  Delay between scans in continuous mode. Accepts integer 1–3600 or `none` for the shortest possible delay (script treats `none` as zero delay). Default: `10` seconds.

- `-c|--countinuous`  
  Runs scanning loop continuously (keeps appending to cache). Use with `--delay` to set frequency.

- `-q|--quiet`  
  Disable vibration feedback (`termux-vibrate`) during scanning.

- `-e|--exit-on-fail`  
  If Termux:API calls fail (e.g., `termux-location` or `termux-wifi-scaninfo`), the script will immediately exit instead of attempting to restart the API.

Behavioural notes:
- The location fetch routine will attempt up to 3 tries (with a 15s timeout per try) to get coordinates. If it fails and `--exit-on-fail` is not set, the script will attempt an API restart (`termux-api-stop` + `termux-api-start`) and retry.
- Cache is newline-delimited JSON (JSONL) so each scan is appended as a JSON object on its own line.

#### Save options
- `-n|--name [filename]`  
  Output filename (without extension). Required for `--save`.

- `-f|--format [json|pjson|jsonl|csv|kml]`  
  Output format. Required for `--save`.  
  Example: `kml` produces a `.kml` file usable in Google Earth.

- `-o|--override`  
  Allow overwriting an existing output file. If omitted and the file exists, the script will abort to avoid accidental data loss.

#### Cache options
- `-l|--list` — list all cached APs (prints a summary).
- `-c|--count` — show number of distinct APs in cache.
- `-p|--purge` — clear the cache (removes internal cache file).

#### Validation
- `V file` — pass a path to the file you want to validate (e.g., exported JSON). The script checks structure and basic content consistency.

### Examples & recommended workflows
(Examples / literal commands are provided below the code block — **not** inside this README block.)

#### Typical workflows
- Quick scan & inspect: run a single scan (`--run`) and then `--cache --list` to see what was captured.
- Continuous mapping ride: use `--run --countinuous -d 5` while walking/driving to build a geo-tagged dataset (adjust `-d` to your needs).
- Export to Google Earth: after building cache, `--save -n map_export -f kml` then open the `.kml` in Google Earth.
- Data analysis: export to `csv` or `jsonl` then import to your analysis tool (Excel, Pandas, etc.).

## Configuration & file locations
- **Cache directory:** `$HOME/.cache/BuriXon-code` (script will create it if missing).  
- **Cache file:** `$HOME/.cache/BuriXon-code/wifi_scan_cache.jsonl` (newline-delimited JSON objects).
- **Log / output files:** saved to current working directory unless you pass a path as part of `--name`.

## Dependencies (full list the script checks)
The script checks presence of these commands and will abort if any are missing:
- `termux-wifi-scaninfo` (Termux:API)
- `jq` (JSON parsing)
- `termux-location` (Termux:API)
- `termux-api-start` / `termux-api-stop` (API control)
- `timeout` (coreutils or busybox)
- `awk`, `sed`, `tr`, `date` (shell utilities)
- `termux-vibrate` (optional unless you rely on vibrations)

Install `jq` explicitly — many Termux installs omit it by default.

## Troubleshooting & tips
- If scans return no GPS coords: try switching provider to `network` (faster indoors) or increase GPS warmup time.  
- If Termux:API calls start failing after some time, use `--exit-on-fail` to force immediate exit (helps when running via systemd-like wrappers) or let the script attempt API restart.  
- For large continuous captures, rotate/export cache regularly to avoid huge files. JSONL is stream-friendly — you can tail it while scanning.
- When using CSV exports, check how special characters are escaped (SSID characters) — prefer JSON/JSONL if you need full fidelity.

## License
GPL-3.0 — see the LICENSE file in the repository.

## Files of interest (in repo)
- The main script (executable) — primary interface.
- README.md (this file) — user guide.
- LICENSE — GPL-3.0.

## Final notes
This is a mobile-first mapping tool — designed for flexibility on Android/Termux. If you want, I can produce an extended README variant that includes:
- sample JSON/JSONL lines produced by the script,
- a ready-to-copy `google-earth` KML snippet preview,
- or a short tutorial on converting JSONL → GeoJSON for GIS workflows.

## Support
### Contact me:
For any issues, suggestions, or questions, reach out via:

- *Email:* support@burixon.dev  
- *Contact form:* [Click here](https://burixon.dev/contact/)
- *Bug reports:* [Click here](https://burixon.dev/bugreport/#Termux-WiFi-map)

### Support me:
If you find this script useful, consider supporting my work by making a donation:

[**Donations**](https://burixon.dev/donate/)

Your contributions help in developing new projects and improving existing tools!

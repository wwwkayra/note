# kayralrics notes

a lightweight, responsive, red-and-black progressive web app with an english-only, lowercase interface. notes stay in the browser; note content is not sent to an application server. the app has no analytics, remote fonts, third-party javascript, or external api calls.

## run locally

from this directory, run:

```bash
python3 -m http.server 8000
```

open <http://localhost:8000>. `localhost` is treated as a secure context by modern browsers, allowing service-worker and pwa features.

## deploy

upload the contents of this directory to a static https host, such as cloudflare pages. no application server or database service is required. the `_headers` file configures security headers on hosts that support cloudflare pages-style headers. other hosts need equivalent http response headers configured in their settings.

## backup and export formats

### full-restoration backups

- `.json` — the complete kayralrics notes backup, including note metadata and local recovery snapshots. this is the most interoperable full backup.
- `.zip` — contains `backup.json` (the full restorable backup), `backup.crc32` (an integrity sidecar), plus readable `.txt`, `.md`, and `.html` copies. the app can import this zip directly. the zip uses the standard uncompressed storage method for broad compatibility.
- `.tar` — a standard ustar archive with `backup.json`, a `backup.crc32` integrity sidecar, recovery history, and readable copies. the app can import this tar directly.
- `.tar.gz` and `.tgz` — the same full tar backup compressed with gzip. export/import requires browser gzip-stream support; choose `.json`, `.zip`, or `.tar` if the browser does not provide it.

### portable notes-only archives

the backup window also exports notes-only archives in these common formats and extensions: `.txt`, `.text`, `.log`, `.md`, `.markdown`, `.mkd`, `.mdown`, `.html`, `.htm`, `.xhtml`, `.rtf`, `.json5`, `.jsonc`, `.csv`, `.tsv`, `.jsonl`, `.ndjson`, `.yaml`, `.yml`, `.xml`, `.toml`, `.properties`, `.ini`, `.conf`, `.cfg`, `.org`, `.rst`, `.adoc`, `.asciidoc`, `.textile`, `.wiki`, `.mediawiki`, `.tex`, and `.latex`.

each individual note can be downloaded separately in the same text-oriented format family, plus `.json`. the per-note `.json`, `.json5`, and `.jsonc` files contain a one-note backup using strict json syntax; `.json` is the recommended extension for maximum compatibility with the restore workflow. `.jsonl` and `.ndjson` contain one note record. csv/tsv output prefixes formula-like cells to reduce spreadsheet formula injection risk; as a result, those formats may add a leading apostrophe to cells beginning with formula characters. xml exports replace invalid xml control characters with the replacement character so the output remains parseable.

these are a broad set of commonly used text formats, not literally every format ever invented. application support differs by device and installed software. formats other than full `.json`, `.zip`, `.tar`, `.tar.gz`, and `.tgz` backups contain notes only and do not contain recovery snapshots, so they are not accepted as full backups. for reliable restoration, choose `.json` or `.zip`; use `.tar` or `.tar.gz` when those archive types fit your workflow.

## data protection

- uses indexeddb as the main database, verifies writes, and checks committed note versions before reporting success.
- requests persistent browser storage where supported; browsers may decline.
- keeps up to three automatic local snapshots per note (at most one every 5 minutes while editing; restore and delete always snapshot first) and verifies recovery copies before destructive actions.
- detects conflicting edits between tabs and avoids silently overwriting a newer note version in the primary indexeddb backend.
- adds a crc32 integrity checksum to new json backups and verifies it during import; legacy json backups without this field remain supported. crc32 helps detect accidental corruption but is not a cryptographic signature.
- validates imported records, rejects duplicate identifiers, checks zip directory and tar header bounds, limits compressed and expanded sizes, verifies archive integrity, and never overwrites an existing note during import; conflicting note ids are imported as separate copies.
- supports importing the app's own `.json`, `.zip`, `.tar`, `.tar.gz`, and `.tgz` full backups. zip64, encrypted archives, ambiguous duplicate `backup.json` entries, malformed tar headers, and unsupported compression methods are rejected.
- uses a service worker with network-first asset refresh and cached fallback for offline opening. query-string variants of shell assets are not cached under canonical asset keys.
- uses a restrictive content security policy and deployment security headers. `_headers` is honored by cloudflare pages and compatible hosts; other hosts must configure equivalent response headers.

## important limitations

**no browser-only app can guarantee zero data loss or absolute security.** the browser can deny persistent storage, clear site data, run out of space, or lose data after device failure. automatic saving and local snapshots reduce risk but are not a substitute for independent backups. download full backups regularly and keep copies on a different drive or in a trusted backup service.

notes are stored without application-level encryption in the browser profile. anyone with access to the unlocked profile or its site data may be able to read them. backup files contain readable note text unless you encrypt them separately.

notes are local to the browser profile and website origin. they do not sync across devices or browsers automatically; use the full backup import/export workflow to move them.

the app imposes no application-level note-count limit. actual capacity depends on browser storage quotas and device space. the zip and tar backup builders limit uncompressed package payload to 100 mb for memory safety. full json export remains available if zip export is too large, subject to browser memory and download limitations.

# Telegram Storage Engine v0.1

Node.js chunked file storage using the direct Telegram Bot API and a local SQLite metadata index.

- Chunk size: **10 MiB** (10,485,760 bytes)
- Upload through `sendDocument`
- Download through `getFile` and Telegram's file URL
- Per-chunk and whole-file SHA-256 verification
- SQLite journal for upload state and recovery

## Requirements

- Node.js 20+
- Telegram bot token
- Private Telegram channel where the bot is an administrator with permission to post
- Channel ID (typically begins with `-100`)

## Setup

```bash
npm install
cp .env.example .env
```

Export the values from `.env` in your shell (or use your preferred environment-variable loader). The project intentionally has no dotenv dependency.

```bash
export TELEGRAM_BOT_TOKEN="..."
export TELEGRAM_CHAT_ID="-100..."
node src/index.js upload ./large-file.zip
node src/index.js status
node src/index.js download FILE_ID ./restored-file.zip
node src/index.js recover
```

PowerShell:

```powershell
$env:TELEGRAM_BOT_TOKEN="..."
$env:TELEGRAM_CHAT_ID="-100..."
node src/index.js upload .\\large-file.zip
```

## Commands

- `upload <path>`: calculates hashes, splits the source into 10 MiB chunks, uploads each as a Telegram document and journals every confirmed result in SQLite.
- `download <file-id> [output-path]`: downloads chunks in order, verifies each chunk, verifies the reconstructed file, then renames the temporary output.
- `status [file-id]`: shows local file/session metadata and chunk states.
- `recover [file-id]`: resumes known-pending chunks and reports uncertain chunks without blindly retrying.

## Recovery caveat

The Bot API cannot search channel history or fetch an arbitrary channel message by message ID. If Telegram accepts a chunk but the network fails before the bot receives the response, the result is ambiguous. The engine marks that chunk `uncertain` and does not automatically retry, because retrying could create a duplicate. Reconcile that chunk manually before proceeding.

SQLite is local (default `.data/index.sqlite`); Telegram stores file bytes, SQLite stores the index. Back up the database. Losing it loses the file-to-Telegram mapping.

## Limits and security

- A 10 MiB chunk is below the standard cloud Bot API's documented upload and download file limits.
- The client handles Telegram HTTP 429 responses with `retry_after`.
- Keep the bot token secret.
- This is experimental storage, not a guaranteed archival service. Keep independent backups of important files.
- Only store files you have permission to store.

## Scope

v0.1 is a local CLI engine, not yet an HTTP API or FTP server.

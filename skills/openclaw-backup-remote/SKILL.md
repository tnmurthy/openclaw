---
name: openclaw-backup-remote
description: Create, verify, and schedule OpenClaw backups, then upload archives to remote storage with rclone (Google Drive, S3, Dropbox, OneDrive, WebDAV, and similar remotes). Use when users ask to automate daily/weekly OpenClaw backups, migrate OpenClaw state safely, keep retention policies, or push backup archives to cloud/NAS storage.
---

# OpenClaw Backup + Remote Storage

Run reliable backup automation for OpenClaw with `openclaw backup create` and `rclone`.

## 1) Validate prerequisites

- Check `openclaw` is installed and working.
- Check `rclone` is installed when remote upload is required.
- Check at least one rclone remote exists (`rclone listremotes`).

If a prerequisite is missing, stop and provide exact install/config commands.

## 2) Create and verify backup

Use:

```bash
openclaw backup create --verify --output <backup-dir>
```

Defaults:

- Use a dedicated backup directory.
- Keep `--verify` enabled unless user explicitly opts out.
- Prefer full backups unless user requests `--only-config` or `--no-include-workspace`.

## 3) Upload to remote storage

Use rclone copy/sync patterns with explicit include filters:

```bash
rclone copy "<backup-dir>" "<remote>:<path>" --include "*openclaw-backup*.tar.gz"
```

Examples:

- `gdrive:OpenClawBackups`
- `s3:company-backups/openclaw`
- `onedrive:OpenClawBackups`

When uncertain, default to `copy` (safer) instead of `sync`.

## 4) Add retention

Apply retention in both locations when requested:

- Local: keep last N archives (default 30).
- Remote: keep last N archives (default 30) if user wants cloud pruning.

Always sort by modified time and delete oldest first.

## 5) Schedule recurring backups

Create cron jobs with explicit timezone and reminder-like summary text when relevant:

- Daily at 2 AM local timezone for typical setups.
- Use isolated `agentTurn` cron jobs for backup automation.
- Include concise success/failure output in delivery.

## 6) Verify end-to-end after scheduling

After creating automation:

1. Trigger one manual run.
2. Confirm local archive exists.
3. Confirm archive appears in remote destination.
4. Report status and next scheduled run time.

## Failure handling

On failure, report:

- Failing step
- Exact command and error
- Minimal fix
- Whether local backup succeeded even if remote upload failed

Never silently ignore upload or verification failures.

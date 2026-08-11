---
name: server-backup-pull
description: Take backups of files before editing them on a remote server, then pull those backups down to this machine, verify by checksum, and only then remove the server-side copies. Load before editing any file on a remote host, when the user asks for a backup or dump, and at the end of any session that modified files on a server.
---

# Backups live on this machine, not on the box

A backup sitting on the same disk as the original only protects against a bad edit. It does not protect against losing the box. Keeping the only copy off-box is the whole point.

## During the work: cheap local reverts

Before editing a file on the server, take a timestamped copy **on the server** so a mid-session revert is one command:

```bash
ssh <host> 'cp -a /etc/nginx/sites-available/mysite /etc/nginx/sites-available/mysite.bak.$(date +%Y%m%d-%H%M%S)'
```

These are working copies. They are temporary and must not survive the session.

## At the end of the work: pull, verify, then delete

Destination on this machine — never the session scratchpad, which is temporary:

```
/home/user/backups/<project>-<ip>/
```

### 1. Collect into a tar on the server

```bash
ssh <host> 'cd / && tar czf /tmp/backup-<project>-$(date +%Y%m%d).tar.gz \
  etc/nginx/sites-available/mysite.bak.* \
  var/www/mysite/index.html.bak.*'
```

### 2. Checksum on the server

```bash
ssh <host> 'md5sum /tmp/backup-<project>-<date>.tar.gz'
```

### 3. Transfer

```bash
mkdir -p /home/user/backups/<project>-<ip>/
scp <host>:/tmp/backup-<project>-<date>.tar.gz /home/user/backups/<project>-<ip>/
```

Add the `ProxyCommand` flag if the host needs it.

### 4. Verify — this step is not optional

```bash
md5sum /home/user/backups/<project>-<ip>/backup-<project>-<date>.tar.gz
```

Compare against step 2. On slow or flaky links a plain `scp` **can truncate silently and still exit 0** — this has happened twice, once with a file as small as 1.7 MB. A zero exit code is not proof of a complete transfer.

**A mismatch means delete nothing.** Re-transfer, re-verify. Report the mismatch rather than working around it.

### 5. Only now, remove the server-side copies

```bash
ssh <host> 'rm -f /tmp/backup-<project>-<date>.tar.gz /etc/nginx/sites-available/mysite.bak.* ...'
```

Then confirm locally that the archive actually opens: `tar tzf <local-archive> | head`.

## Boundaries

- This covers **backup copies you created**. Never delete pre-existing files the user did not ask about — their original assets are not yours to clean up.
- Databases: no dumps against a production database without asking first. Default to a read-only or scratch DB.
- Report the local path, the archive size, and the verified checksum when done. "Backed up" without those is an unverified claim.

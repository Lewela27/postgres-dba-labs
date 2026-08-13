# Lab 01 — working notes

## What broke

### Lost WSL account password
Three failed sudo attempts during the postgresql-18 install.
Recovered via `wsl -u root` from PowerShell, then `passwd lewela`.
Root inside WSL needs no password — the security boundary is the Windows
account, not the Linux one.

### Peer authentication failure
    $ psql -U testuser -d labdb
    FATAL: Peer authentication failed for user "testuser"
No -h flag means the Unix socket, which matches the `local` line in
pg_hba.conf using `peer`. Peer asks the kernel who owns the connecting
process (lewela), sees it doesn't match the requested role (testuser),
and refuses. The password is never consulted. Adding -h localhost takes
the TCP path, matches a `host` line, and the password works.

### postgresql.service looked dead but wasn't
`systemctl status postgresql` shows `active (exited)` — it's an empty
wrapper unit. The real one is `postgresql@18-main.service`.
`pg_lsclusters` is the more reliable check.

## Observations

- PG18 enables data page checksums at initdb by default; older versions
  needed --data-checksums explicitly.
- PG18 runs `io worker` background processes (async I/O, `io_method`).
  No equivalent in earlier versions.
- Connections to localhost negotiate TLSv1.3 by default.
- PGDG build won over Ubuntu's without pinning: 18.4-1.pgdg26.04+1 beats
  18.4-0ubuntu0.26.04.1 at equal priority 500, on version number alone.

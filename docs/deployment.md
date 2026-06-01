---
title: Deployment
layout: default
---

# Deployment

SequenceHub supports three deployment modes. Same binary, same REST API, same
processing pipeline — mode selected by configuration, not by code.

| | Desktop | Single-node web | Distributed |
|---|---|---|---|
| Packaging | Electron + `.exe` / `.AppImage` | `docker run` · native binary | `docker compose` |
| Job queue | `Channel<T>` | `Channel<T>` | Redis List |
| SSE events | In-process | In-process | Redis Streams |
| User store | Not required | JSON file | PostgreSQL |
| Extra deps | **None** | **None** | Redis + PG |
| Scale | 1 user | N users, 1 node | N workers |
| Install UX | Next → Finish | One-line command | One-line command |

## Desktop

Electron manages the backend as a child process on `localhost`. No server, no
cloud account, no network required. File associations (`.ab1 .srd .dan .scf`)
open directly in the app.

Packages: **Windows** — NSIS / Squirrel `.exe` · **Linux** — `.AppImage` · `.deb` · `.rpm`

## Single-node web

```bash
docker run -p 80:80 sequencehub/procsvc
```

## Distributed

```bash
docker compose up --scale app=4
```

Workers connect via Redis. SSE fan-out via Redis Streams — any worker instance
can serve any client.

---

*Full deployment documentation coming soon.*

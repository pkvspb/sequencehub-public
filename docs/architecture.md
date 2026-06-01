---
title: Architecture
layout: default
---

# Architecture

SequenceHub is built on four decoupled layers.

| Layer | Components | Role |
|---|---|---|
| **Browser** | SeqView (Vanilla JS) · XSeq (React + Vite) · webcomponents Canvas | Visualization — two independent frontends, one rendering library |
| **Service** | ASP.NET Core 9 · REST + SSE · file store · job queue · auth | Orchestration — deployment-mode agnostic |
| **Processors** | SangerBasecaller · ProcessStarter → BV / AIP | Basecalling — each job runs as an isolated OS process |
| **Data** | SequenceLogicData — AB1, SRD, DAN, SCF | File I/O — normalised to ACGT channel order in memory |

`SequenceLogicData.dll` is loaded at runtime by the service and compiled into
SangerBasecaller — one library, no duplication.

**Key properties:** isolated process execution · normalised signal model ·
deployment-agnostic service · decoupled frontends via REST + SSE

The Browser layer also ships as an **Electron desktop application** — same
frontend code, backend managed as a child process on `localhost`, no server
required.

---

*Full architecture documentation coming soon.*

---
title: Architecture
layout: doc
---

# Architecture

SequenceHub is a Sanger DNA sequencing analysis platform with four layers:

1. **File I/O** — `SequenceLogicData` reads and writes SRD, AB1, DAN, and SCF sequencing files
2. **Basecalling** — `SangerBasecaller` (PkvBasecallerN) and native processors (BV, AIP) convert raw fluorescence signals to base calls
3. **Web service** — `ProcSvc` (prochub) manages file storage, job queuing, user auth, and HTTP endpoints
4. **Visualization** — browser-based electropherogram viewer built on the `webcomponents` canvas library

## Component Dependencies

```
SequenceLogicData  (.NET 9 — SRD/AB1/DAN/SCF I/O)
  ├──► PkvBasecallerN  bundles SequenceLogicData.dll at compile time
  └──► prochub/Storage  loads SequenceLogicData.dll at runtime

PkvBasecallerN  →  published binary  →  prochub/back/ExternalBasecallers/PkvBasecallerN/
ProcessStarter  →  published binary  →  prochub/back/ExternalBasecallers/ProcessStarter/

webcomponents  (no build step)
  ├──► prochub/front/  installed as local file: package
  └──► xseq/  installed as local file: package
```

## Processing Flow

1. User uploads sequence files through the web UI
2. `POST /process/{processorName}` enqueues a job and returns a `jobId`
3. The backend shells out to a basecaller binary for each file:
   - **SangerBasecaller** — called directly
   - **BV processor** — routed via `ProcessStarter {file} bv`
   - **AIP processor** — routed via `ProcessStarter {file} aip {binaryPath}`
4. The client streams live progress via `GET /process/{jobId}/events` (Server-Sent Events)
5. Results — called bases, quality scores, electropherogram traces — are rendered on HTML5 Canvas by `webcomponents`

## Deployment Modes

Two modes selected at startup via `appsettings.json` or environment variables:

**Single-node** (default — no external dependencies):
- Job queue: in-process `Channel<T>`
- User store: JSON file
- SSE events delivered in-process; client must reach the same instance that runs the job

**Distributed** (requires Redis + PostgreSQL):
- Job queue: Redis List, polled every 100 ms
- User store: PostgreSQL
- SSE events: Redis Streams — any instance can serve events for any job
- Cancellation: Redis key, polled at a configurable interval

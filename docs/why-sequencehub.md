---
title: Why SequenceHub
layout: doc
---

# Why SequenceHub

Most Sanger sequencing software forces a choice: a desktop application tied to
one machine, or a web tool with none of the responsiveness of native software.
SequenceHub is built so that choice doesn't have to be made — and so that the
processing behind it isn't capped by a single machine's CPU.

## One codebase, two deployment shapes

SequenceHub runs as a browser-native web application *or* as a self-contained
Electron desktop app (Windows NSIS/Squirrel installer, Linux `.AppImage` /
`.deb` / `.rpm`) — from the same ASP.NET Core backend, the same REST + SSE
job API, and the same processing pipeline. Nothing is reimplemented or
stripped down for the desktop build; it's the identical service running
locally with file associations for `.ab1`, `.srd`, `.dan`, and `.scf`. Most
sequencing tools are one or the other: a legacy desktop viewer, or a web
front end that can't be installed and run offline. SequenceHub is both,
without maintaining two products.

## Canvas rendering, not a lowest-common-denominator web view

The chromatogram viewer renders traces and vendor-calibrated quality overlays
(~Q0–24) directly on HTML5 Canvas, purpose-built for large multi-channel
time-series data. It's
the same responsiveness expected from a native desktop viewer, not the
sluggish panning/zooming typical of DOM- or SVG-based sequencing viewers in
the browser.

## Linux is a first-class target, not an afterthought

Because the backend is ASP.NET Core and the desktop shell is Electron,
SequenceHub packages natively for Linux (`.AppImage`, `.deb`, `.rpm`) as well
as Windows — a gap in most Sanger analysis software, which tends to assume a
Windows-only desktop environment. The same Docker image also runs on any
Linux host as the web deployment.

## Linear scaling across machines, not just cores

Single-node mode already parallelizes basecalling across files with
`Parallel.ForEachAsync`. Distributed mode goes further: the job queue moves
to a Redis List, job events fan out over Redis Streams, and workers scale
horizontally with `docker compose up --scale app=4` — including across
separate physical machines, not just additional threads on one box. Any
worker can pick up any job and any instance can serve that job's SSE stream,
so throughput scales linearly by adding nodes. This is a distributed
compute architecture, not a single powerful server with a queue in front of
it.

## An architecture open to other vendors' basecallers

The processing layer is plugin-based: basecallers are invoked as external OS
processes through `ProcessStarter`, not linked into the service. The
in-house `SangerBasecaller` runs this way today, alongside third-party native
basecallers (BV, AIP) called through the same mechanism. Because a basecaller
is just an external process the queue dispatches to, a vendor can bring their
own basecaller in whatever language or runtime it's built in — and it
automatically inherits SequenceHub's parallel and distributed scaling, since
the scheduler doesn't care what's inside the process it started. Vendors
aren't locked out or required to re-architect for scale-out; they get it by
plugging into the existing job pipeline.

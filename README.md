Browser-native platform for Sanger DNA sequencing analysis — built to replace
legacy desktop sequencing tools with a modern, observable, and extensible
alternative.

It reads raw capillary electrophoresis output (AB1, SRD, DAN v2–6, SCF v3), runs
a 9-stage basecalling pipeline, and renders results through a high-performance
Canvas chromatogram viewer with Q0–Q60 quality overlays. The backend is
ASP.NET Core 9 with a plugin-based processor architecture, REST + SSE job API,
and a full observability stack (Prometheus, Grafana, Loki, OpenTelemetry) bundled
in `docker-compose`. It deploys as a single Electron desktop application or scales
to a distributed multi-worker setup — same codebase, same API, mode selected by
configuration.

The core architecture is domain-agnostic. The same processing orchestration,
file I/O model, and visualization stack apply to any multi-channel time-series
signal source — analytical instruments, industrial telemetry, or scientific
visualization SaaS.

## [▶ Platform Presentation (EN)](presentation/platform.html)

## [▶ Platform Presentation (RU)](presentation/platform.limited.ru.html)

## Documentation

- [Architecture](docs/architecture.md)
- [Deployment](docs/deployment.md)

## This repository

Public documentation for the SequenceHub platform. Open-source components —
including the core service, file format I/O library, and Canvas visualization
layer — are coming soon.

## License

Licensed under the [Apache License 2.0](LICENSE).

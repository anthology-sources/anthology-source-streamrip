# anthology-source-streamrip

Optional source service for [Anthology](https://github.com/anthology-music/anthology), based on [streamrip](https://github.com/nathom/streamrip).

- **Provides:** Tidal (initially); streamrip also supports Qobuz, Deezer and SoundCloud.
- **License: GPL-3.0-only** (see `LICENSE`). streamrip is licensed under GPL-3.0-only. The Anthology core (MIT) does not import this code; it calls the service through the versioned source service interface and reads finished files from the shared `imports` volume.
- Runs as its own, **optional** container (`anthology-source-streamrip`) in the Anthology pod. Without it, Anthology works normally and the sources it provides are hidden.
- Users are responsible for using it in line with the terms of service of the services it accesses.

## Interface

Implements the source service interface v1 defined by the core: [`docs/architecture/source-service-api.md`](https://github.com/anthology-music/anthology/blob/main/docs/architecture/source-service-api.md) (decision: [ADR 0017](https://github.com/anthology-music/anthology/blob/main/docs/adr/0017-source-services-outside-core.md)).

| Endpoint | Purpose |
|---|---|
| `GET /v1/info` | service id, display name, interface version, capabilities (video: no) |
| `POST /v1/search` | search albums, tracks, playlists |
| `POST /v1/resolve` | resolve a link into items with ISRC, duration and offered quality |
| `POST /v1/download` | download items into the shared `imports` volume; returns a task ID |
| `GET /v1/tasks/{id}` | progress and result (also as server-sent events) |
| `GET /v1/health` | health, version, login state |

## Running

Add the container to the Anthology pod and register it with the core:

```
# Podman with Quadlet
cp deploy/quadlet/* ~/.config/containers/systemd/
systemctl --user daemon-reload && systemctl --user restart anthology-pod

# Anthology core configuration (containers in a Quadlet pod share localhost)
ANTHOLOGY_SOURCE_SERVICES=http://localhost:8420,...
```

Docker/Compose users merge `deploy/compose.snippet.yaml` into Anthology's `compose.yaml` and use `http://anthology-source-streamrip:8420` in `ANTHOLOGY_SOURCE_SERVICES`.

The service listens on port 8420 (`ANTHOLOGY_SOURCE_PORT`); ports 8420–8429 are reserved for source services.

Status: placeholder – nothing built yet.

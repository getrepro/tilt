# Repro Tilt (Full Local Dev)

This Tilt setup always runs the full local stack for repos in `../`:

- Infra: MongoDB, Kafka, Kafka UI, ClickHouse
- SDK pipelines: `node-sdk` and `react-sdk` in watch mode
- Apps: `repro-api`, `repro-web`, `log-collector`, `repro-demo` backend + frontend

## Prerequisites

1. Docker
2. Tilt
3. Node.js + npm

## Quick Start

1. `cd tilt`
2. `tilt up`

No profiles/modes are required. This always starts the full environment.

## Resources

- `infra` (docker compose): MongoDB + Kafka + Kafka UI + ClickHouse
- `node-sdk`: initial build + TypeScript watch build
- `react-sdk`: initial build + TypeScript watch build
- `log-collector`: Fastify ingest service (`http://localhost:8080/health/live`)
- `repro-api`: Nest API (`http://localhost:4010/api/docs`)
- `repro-web`: web console (`http://localhost:5174`)
- `repro-demo-backend`: TaskFlow backend (`http://localhost:3008`)
- `repro-demo-web`: TaskFlow frontend (`http://localhost:5173`)

## SDK Live Dev Behavior

- `repro-demo/backend` depends on `node-sdk` and restarts when `node-sdk/dist/index.js` changes.
- `repro-demo` frontend depends on `react-sdk` and auto-restarts in Tilt when `react-sdk/dist` changes.

## Ports

- MongoDB: `27017`
- Kafka: `9092`
- Kafka UI: `8081`
- ClickHouse HTTP/native: `8123` / `9000`
- Log Collector: `8080`
- Repro API: `4010`
- Repro Web: `5174`
- Repro Demo backend/frontend: `3008` / `5173`

## Notes

- Each app command sources local `.env` and `.env.local` before Tilt overrides local dev defaults.
- Edit ports and URLs via the `PORTS` map in `tilt/Tiltfile`.

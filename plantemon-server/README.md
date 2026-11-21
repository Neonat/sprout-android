# Plantemon API

A TypeScript API for the [Expo client](../plantemon-app/README.md).
Vercel functions verify Clerk tokens, proxy Plant.id and NVIDIA calls, and
save each user's plant garden. The native [Java app](../README.md) has its
own direct-provider and local-storage implementation.

## Stack and architecture

| Technology | Role |
|---|---|
| Vercel functions | HTTP handlers under `api/` and local development through `vercel dev`. |
| Clerk | Verifies bearer tokens and supplies the user identity. The first request provisions a `users` row. |
| Neon Postgres and Drizzle | Store users and owned plants. The TypeScript schema defines columns and generates SQL migrations. |
| Vercel Blob | Stores public sprite PNGs and caches them by normalized plant name. |
| Plant.id and NVIDIA | Identify a photo, craft a sprite description with Gemma, and render it with Flux. |

The API separates provider credentials from the Expo bundle. Postgres holds
structured garden data, while Blob holds image bytes. This is the port's
cloud persistence model; it does not reuse the Java app's JSON files.

## Endpoints

Supported application requests require `Authorization: Bearer <clerk-jwt>`.

| Method | Path | Purpose |
|---|---|---|
| `POST` | `/api/identify` | Identify `{ imageBase64 }` with Plant.id. |
| `POST` | `/api/sprite` | Generate or reuse a sprite for `{ name, imageBase64 }`. |
| `GET` | `/api/me` | Read the current user and garden. |
| `PATCH` | `/api/me` | Update username, active plant, or heal charges. |
| `GET` | `/api/plants` | List the user's plants. |
| `POST` | `/api/plants` | Create a plant, with a six-plant cap. |
| `PATCH` | `/api/plants/[id]` | Update an owned plant. |
| `DELETE` | `/api/plants/[id]` | Release an owned plant. |

## Local setup

From the repository root:

```bash
cd plantemon-server
npm install
npx vercel link
cp .env.example .env.local
```

Configure the values in [`.env.example`](.env.example):

| Variables | Service |
|---|---|
| `PLANT_API_KEY` | Plant.id. |
| `NVIDIA_API_KEY`, `FLUX_API_KEY` | NVIDIA description and image generation. Flux also accepts `NVIDIA_API_KEY` as its fallback. |
| `CLERK_SECRET_KEY`, `CLERK_PUBLISHABLE_KEY` | The Clerk project used by the Expo client. |
| `DATABASE_URL` | Neon Postgres connection. |
| `BLOB_READ_WRITE_TOKEN` | Vercel Blob store. |

Provider keys and `CLERK_SECRET_KEY` stay on the server. If the Vercel project
already has its environment configured, use `npx vercel env pull .env.local`
instead of filling the file manually.

The Drizzle configuration reads `DATABASE_URL` from the process environment.
Export the connection string in your shell before generating and applying the
schema to your development database:

```bash
export DATABASE_URL='your_development_postgres_connection_string'
npm run db:generate
npm run db:migrate
npm run dev
```

`db:generate` writes SQL under `drizzle/`; `db:migrate` applies it. Vercel dev
serves the API locally, normally at [localhost:3000](http://localhost:3000).
Set the client's `EXPO_PUBLIC_API_URL` to a reachable API address.

## Checks and limits

```bash
npm run build
```

`build` runs `tsc --noEmit`; it does not package a server or test integrations.
There is no automated test script in this package. A missing bearer token is
an `AuthError` with a 401 response. Other failures are handled by the shared
[response helper](src/lib/respond.ts).

Plant updates include an ownership filter, but the PATCH handler passes request
fields to Drizzle without a separate input schema. Provider calls, cloud
persistence, and client journeys require further validation. The Expo battle
runs on the client; this API is not an authoritative battle engine.

## Source map

- `api/` contains the HTTP handlers.
- `src/db/schema.ts` defines the user and plant tables.
- `src/db/client.ts` creates the Neon connection.
- `src/lib/auth.ts` verifies tokens and provisions users.
- `src/lib/plantid.ts` and `src/lib/sprite.ts` implement provider calls.
- `drizzle.config.ts` configures migration generation and application.

The [root README](../README.md) records the project context, team provenance,
and repository license.

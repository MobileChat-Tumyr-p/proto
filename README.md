# @tumy/mobile-chat-grpc

The **single source of truth for gRPC contracts** in the MobileChat project.
Versioned `*.proto` files are compiled with **ts-proto** into typed NestJS clients
used by every service.

---

## Contents

| Proto file            | Package      | Purpose                                        |
| --------------------- | ------------ | ---------------------------------------------- |
| `proto/auth.proto`    | `auth.v1`    | Auth microservice contract (`AuthService`)     |
| `proto/chat.proto`    | `chat.v1`    | Chat list / concrete chat (`getChatListService`)|
| `proto/successgmail.proto` | `success.v1` | (legacy) `SuccessService.SendSuccess`       |
| `proto/reg.proto`     | `reg.v1`     | Stub / reserved for registration               |

## What it generates

Running the `generate` script produces decorated TS classes under `gen/`:

- gRPC service clients ready for **`@nestjs/microservices`** gRPC transport
- `keepCase` handled by Nest, proto casing converted to camelCase (via `package=omit`)
- `nestJs=true` option → Nest-safe client types (e.g. `AuthServiceClient`)

Generated package layout (`ts_proto`):

```
gen/
├── auth.js · auth.ts
├── chat.js · chat.ts
├── successgmail.js · successgmail.ts
└── reg.js · reg.ts
```

## Usage in services

```text
proto (this repo) ──protoc + ts-proto──► @tumy/mobile-chat-grpc (published, public)
                                                │
            ┌───────────────────────────────────┼───────────────────────────┐
            ▼                                   ▼                           ▼
   MobileChat-Auth                       MobileChatGateway           MobileChatBackend
   (serves auth.v1)                      (grpc clients)              (legacy success.v1)
```

Consumers add the package as a dependency and reference the proto at:

```text
node_modules/@tumy/mobile-chat-grpc/proto/*.proto
node_modules/@tumy/mobile-chat-grpc/gen/*.js
```

## Generating after a contract change

```bash
yarn generate        # protoc -I ./proto ... --ts_proto_out=./gen
```

Then:

1. Bump the version in `package.json`.
2. Publish (public package): `yarn publish`.
3. Update `@tumy/mobile-chat-grpc` in the consuming services.

### Prerequisites

- `protoc` (Protocol Buffers compiler) on `PATH`
- ts-proto (`yarn install`)

## CI / release

- On every push to `main` the package is published automatically so all services pick up
  the new contract (see the repository's CI workflow).
- Publishing is enabled via `publishConfig.access: "public"`.

## Scripts

| Command       | Description                                |
| ------------- | ------------------------------------------ |
| `yarn install`| Install dependencies (protoc, ts-proto…)   |
| `yarn generate` | (Re)compile `.proto` -> `gen/`          |
# SECURITY

Owns the security posture: secure-coding rules, security controls and their configuration values, trust-boundary enforcement, and open security decisions. Behavior: `REQUIREMENTS.md`. Visual design and user interactions: `DESIGN.md`. Components, interfaces, data flows, technology choices, and trust-boundary definitions: `ARCHITECTURE.md`.

## Secure-Coding Inputs

- **Sources:** `REQUIREMENTS.md`, `DESIGN.md`, `ARCHITECTURE.md`, `THREAT_MODEL.md` (threats `T-*`), and the owner's security notes (one user at a time, no admins, no social features). Each rule cites the requirement (`FR-*`, `NFR-*`), trust boundary (`TB-*`), dependency rule (`DR-*`), or decision (`OD-*`, `AQ-*`) it rests on.
- **Material gaps:**
  - No evaluated pack covers the Azure Key Vault SDK or managed-identity client. The official `@azure` SDK is used under the general Node.js 26 and ES2026 rules (OD-014).
  - No evaluated pack covers `fast-xml-parser`. SC-038 relies on its documented entity and DTD settings.

## Selected Manicode Prompts

**Corpus root:** `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge`

Resolved with `python3 scripts/resolve_prompt.py js/nodejs/26 js/react/19 js/typescript/7 js/fastify/5.12 js/zod/4 js/prisma/7.10 infra/dockerfile/1 python/python/3.13 --json`.

**Direct selection evidence** (all from ARCHITECTURE "Required Architecture Inputs")

- `js/nodejs/26`: **SELECTED**. "Fastify 5.12 on Node.js 26".
- `js/react/19`: **SELECTED**. "React 19". Candidate input `js/react/` resolved to 19.
- `js/typescript/7`: **SELECTED**. "written in TypeScript 7" for server and client. Also included by Fastify, Zod, and Prisma.
- `js/fastify/5.12`: **SELECTED**. "Fastify 5.12".
- `js/zod/4`: **SELECTED**. "Validation: Zod 4".
- `js/prisma/7.10`: **SELECTED**. "Data access: Prisma 7.10".
- `infra/dockerfile/1`: **SELECTED**. "One Azure VM running Docker. Three containers".
- `python/python/3.13`: **SELECTED**. "Python ML worker (Python 3.13)". It was not in the candidate list; it was added to cover the worker (former OD-012).

**Version substitutions reported by the resolver**

- `js/fastify/5.12` declares `js/nodejs/24`; `js/nodejs/26` is composed because it is named directly.
- `js/prisma/7.10` declares `js/nodejs/24`; `js/nodejs/26` is composed because it is named directly.

**Preamble:** `content/preamble.md`

**Ordered composition**

1. `js/javascript/ES2026`: transitive (via Node.js, React, TypeScript). `content/js/javascript/ES2026/prompt.md`, sources `content/js/javascript/ES2026/sources.md`
2. `js/nodejs/26`: direct. `content/js/nodejs/26/prompt.md`, sources `content/js/nodejs/26/sources.md`
3. `js/react/19`: direct. `content/js/react/19/prompt.md`, sources `content/js/react/19/sources.md`
4. `js/typescript/7`: direct, and also transitive via Fastify, Zod, Prisma. `content/js/typescript/7/prompt.md`, sources `content/js/typescript/7/sources.md`
5. `js/fastify/5.12`: direct. `content/js/fastify/5.12/prompt.md`, sources `content/js/fastify/5.12/sources.md`
6. `js/zod/4`: direct. `content/js/zod/4/prompt.md`, sources `content/js/zod/4/sources.md`
7. `js/prisma/7.10`: direct. `content/js/prisma/7.10/prompt.md`, sources `content/js/prisma/7.10/sources.md`
8. `infra/dockerfile/1`: direct. `content/infra/dockerfile/1/prompt.md`, sources `content/infra/dockerfile/1/sources.md`
9. `python/python/3.13`: direct. `content/python/python/3.13/prompt.md`, sources `content/python/python/3.13/sources.md`

## Secure-Coding Rules

Status values: `ACTIVE` rules are specified and ready to implement; this is not a claim that they are implemented. `BLOCKED` rules cannot be implemented until the cited open item is decided. Rules SC-039 onward come from `THREAT_MODEL.md` and cite their threats (`T-*`).

**SC-001 — Session required on every non-public route**
- **Status:** ACTIVE
- **Rule:** Every API route except login MUST reject a request without a valid server-side session. The session check MUST run in a Fastify hook registered in the scope that owns the protected routes. It MUST NOT run in path-prefix middleware.
- **Enforcement:** API (Fastify routes) and Auth module.
- **Evidence:** FR-6.2; ARCHITECTURE Components (API: "Login before use"), TB-1. Fastify 5.12 prompt, "Authentication, hooks, and encapsulation" and "Authorization and lifecycle hooks".
- **Verification:** A `fastify.inject()` test calls each protected route, including automatic `HEAD` routes, with no session and with an expired session. Each call returns 401 with no side effects.

**SC-002 — Session cookie attributes and server-side lifetime**
- **Status:** ACTIVE
- **Rule:** Session cookie requirements:
  - The cookie MUST use a `__Host-` name with `path: '/'` and no `domain`, plus `Secure`, `HttpOnly`, and `SameSite=Strict`.
  - The server MUST end a session after 30 minutes idle and 8 hours total, checked against the session record in Postgres. `@fastify/session` rolling renewal MUST NOT extend the 8-hour limit.
  - The default memory store MUST NOT be used, and `saveUninitialized` MUST be `false`.
- **Enforcement:** Auth module (`@fastify/session` with a Postgres store).
- **Evidence:** ARCHITECTURE "Authentication and session model"; OD-006. Fastify 5.12 prompt, "Browser state and OAuth".
- **Verification:**
  - The `Set-Cookie` header on login matches the attributes above.
  - A session idle for more than 30 minutes is rejected.
  - A continuously active session older than 8 hours is rejected.

**SC-003 — Session regeneration at login and privilege change**
- **Status:** ACTIVE
- **Rule:** The server MUST await `request.session.regenerate()` after a successful login and after a password change. A session ID from before login MUST NOT stay valid.
- **Enforcement:** Auth module.
- **Evidence:** FR-6.2, FR-6.4; OD-006. Fastify 5.12 prompt, "Browser state and OAuth".
- **Verification:** A session cookie that existed before login returns 401 after login. The cookie value changes after a password change.

**SC-004 — Argon2id password storage**
- **Status:** ACTIVE
- **Rule:** The owner password MUST be stored only as an Argon2id hash with a unique random salt of at least 16 bytes. Parameters MUST be at least `m=19456` KiB, `t=2`, `p=1`, stored with the hash. A fast digest or reversible encryption MUST NOT be used.
- **Enforcement:** Auth module and the account-creation CLI.
- **Evidence:** NFR-5; ARCHITECTURE "Authentication and session model". ES2026 prompt, "Secrets and crypto".
- **Verification:** The stored verifier shows Argon2id with parameters at or above these minimums. Two hashes of the same password differ.

**SC-005 — Login rate limiting**
- **Status:** ACTIVE
- **Rule:** The login route MUST enforce two limits:
  - An early per-IP limit of 5 attempts per 15 minutes with `@fastify/rate-limit`, keyed by the default canonical IP key generator.
  - A separate account limit of 20 failed attempts per hour, created with `fastify.createRateLimit()` and called from `preHandler`.
  - Both limits use the in-memory store, because there is one server instance.
  - The two limits MUST use distinct key prefixes. `skipOnError` MUST stay `false`. Keys MUST NOT come from forwarding headers.
- **Enforcement:** Auth module, login route.
- **Evidence:** OD-007. Fastify 5.12 prompt, "Availability and operations".
- **Verification:** Test cases:
  - The 6th attempt from one IP within 15 minutes gets 429.
  - The 21st failed attempt within an hour from rotating IPs gets 429.
  - Each budget is tested on its own.

**SC-006 — CSRF protection on state-changing routes**
- **Status:** ACTIVE
- **Rule:** `@fastify/csrf-protection` MUST be registered after the session plugin with `sessionPlugin: '@fastify/session'`. `fastify.csrfProtection` MUST run in `onRequest` with a header-only `getToken` on every state-changing route: trade entry, CSV import, approve, reject, restore, proposal request, password change, logout.
- **Enforcement:** API (Fastify routes).
- **Evidence:** OD-006; FR-4.2, FR-5.2, FR-5.4, FR-5.6, FR-6.4, FR-6.5. Fastify 5.12 prompt, "Browser state and OAuth". ES2026 prompt, "Storage and messages".
- **Verification:** Each listed route returns 403 when the CSRF header is missing or wrong, even with a valid session.

**SC-007 — Strict request validation with unknown-field rejection**
- **Status:** ACTIVE
- **Rule:** Request validation requirements:
  - Every route MUST declare Zod 4 schemas for `params`, `querystring`, `headers`, and `body`, compiled through the Zod validator compiler.
  - Object schemas MUST use `z.strictObject`.
  - Header schema property names MUST be lowercase. Because a custom validator compiler gets no Fastify header normalization, the compiler MUST apply the same header constraints itself.
  - Handlers MUST use only the parsed output.
  - `attachValidation` MUST stay disabled.
- **Enforcement:** API (Fastify routes, validator compiler).
- **Evidence:** ARCHITECTURE "API"; OD-013; TB-1. Fastify 5.12 prompt, "Request schemas and validation". Zod 4 prompt, "Object and collection schemas" and "Parsing, coercion, and defaults".
- **Verification:** Test cases for each route:
  - A body with an extra field returns 400 with no side effect.
  - Header checks pass and fail correctly under mixed wire casing.

**SC-008 — No security-relevant coercion or defaults**
- **Status:** ACTIVE
- **Rule:** Trade fields (`side`, `date`, `quantity`, `price`) and approve, reject, and restore identifiers MUST NOT be coerced with `z.coerce.*` or filled with `.default()` or `.catch()`. Invalid values MUST fail validation.
- **Enforcement:** User Trades, Adaptation, Strategy, and Model Lifecycle request schemas.
- **Evidence:** FR-4.1, FR-4.2.1, FR-5.4, FR-5.6. Zod 4 prompt, "Parsing, coercion, and defaults".
- **Verification:** These inputs each return 400: `quantity: "10"` as a string, `side: "BUY "`, a missing `price`.

**SC-009 — CSV import validation and whole-file rejection**
- **Status:** ACTIVE
- **Rule:** The CSV importer MUST:
  - Require a header row that is exactly the FR-4.2.1 column names, in FR-4.2.1 order.
  - Validate every row with a strict schema for the FR-4.2.1 column formats. Quantity and price MUST be finite and in range.
  - Write nothing if any row fails (FR-4.2.1), inside one transaction.
  - Return the row number and cause without echoing the payload.
- **Enforcement:** User Trades module (CSV import).
- **Evidence:** FR-4.2, FR-4.2.1. Zod 4 prompt, "Output and disclosure". ES2026 prompt, "Text and numbers". Prisma 7.10 prompt, "Transactions and concurrency".
- **Verification:** A file with one bad middle row stores zero rows and names that row number. A wrong header is rejected.

**SC-010 — Request and CSV size limits**
- **Status:** ACTIVE
- **Rule:** Size limits:
  - The server `bodyLimit` MUST be 64 KiB.
  - The CSV route MUST accept only `text/csv`, with a route `bodyLimit` of 1 MiB and a cap of 10,000 data rows. The parser MUST be registered with `parseAs: 'string'`.
  - Other media types MUST be rejected.
- **Enforcement:** Fastify server configuration; User Trades CSV route.
- **Evidence:** OD-008. Fastify 5.12 prompt, "Parsing and uploads".
- **Verification:**
  - A 65 KiB JSON body returns 413.
  - A 1.1 MiB CSV returns 413.
  - A CSV with 10,001 data rows is rejected.
  - A `multipart/form-data` request to the CSV route returns 415.

**SC-011 — Prototype-poisoning protection kept on**
- **Status:** ACTIVE
- **Rule:** The JSON parser MUST keep `onProtoPoisoning: 'error'` and `onConstructorPoisoning: 'error'`. Fastify's protected parser MUST NOT be replaced with bare `JSON.parse`.
- **Enforcement:** Fastify server configuration.
- **Evidence:** TB-1. Fastify 5.12 prompt, "Parsing and uploads". ES2026 prompt, "Objects".
- **Verification:** A body containing `"__proto__"` returns 400.

**SC-012 — Prisma queries built only from validated scalars**
- **Status:** ACTIVE
- **Rule:** Prisma `where`, `data`, `orderBy`, `select`, and `include` arguments MUST be built from validated primitives and server-side allowlists. A request object or field MUST NOT be passed into them directly. `strictUndefinedChecks` MUST be enabled.
- **Enforcement:** All domain modules using Prisma (DR-3 owning modules).
- **Evidence:** ARCHITECTURE inputs ("Prisma 7.10"), "Persistence". Prisma 7.10 prompt, "Query construction".
- **Verification:** A lookup that receives `{ "not": "" }` as an ID returns 400 and runs no query. Code review finds no request objects spread into Prisma arguments.

**SC-013 — No unsafe raw SQL**
- **Status:** ACTIVE
- **Rule:** Code MUST NOT call `$queryRawUnsafe` or `$executeRawUnsafe`, or pass request-derived text to `Prisma.raw()`. Raw SQL MUST use tagged `$queryRaw`, `$executeRaw`, or TypedSQL, with interpolated data values only.
- **Enforcement:** All domain modules using Prisma.
- **Evidence:** ARCHITECTURE ("Prisma 7.10"). Prisma 7.10 prompt, "Raw queries".
- **Verification:** A CI grep for `queryRawUnsafe`, `executeRawUnsafe`, and non-constant `Prisma.raw(` returns no matches.

**SC-014 — Password hash never returned**
- **Status:** ACTIVE
- **Rule:** Account queries MUST use an explicit `select` that leaves out the password hash. A client-level `omit` for the hash MUST also be configured. Every route MUST define `schema.response`.
- **Enforcement:** Auth module; API response schemas.
- **Evidence:** NFR-5. Prisma 7.10 prompt, "Models and data exposure". Fastify 5.12 prompt, "Replies, routing, and errors". Zod 4 prompt, "Output and disclosure".
- **Verification:** No API response in the test suite contains the hash field. Every route has a response schema.

**SC-015 — Envelope encryption of trade data**
- **Status:** ACTIVE
- **Rule:** Encryption requirements:
  - All user trade fields (symbol, side, date, quantity, price) and trade success results MUST be encrypted with AES-256-GCM before Prisma writes them. Each value MUST get a unique random nonce. `authTagLength` MUST be 16.
  - Decrypted output MUST NOT be used until `decipher.final()` succeeds.
  - The Key Vault wrapping key MUST rotate yearly, and the data key MUST be rewrapped after each rotation.
  - The data key MUST be stored only wrapped by the Azure Key Vault key, and MUST NOT be logged or written unwrapped to disk.
- **Enforcement:** User Trades and Trade Evaluation modules, through the encryption adapter (DR-4; component open: AQ-13).
- **Evidence:** NFR-5; TB-3; ARCHITECTURE "Persistence" and "Azure Key Vault"; OD-005. Node.js 26 prompt, "Cryptography and secrets". ES2026 prompt, "Secrets and crypto".
- **Verification:**
  - A direct SQL `SELECT` on the trade and result tables shows no plaintext field values.
  - A tampered ciphertext fails decryption, and nothing is returned.
  - Two encryptions of the same value produce different ciphertexts.

**SC-016 — No user trade data leaves the host**
- **Status:** ACTIVE
- **Rule:** User trade records and trade success results MUST NOT be included in any request to SEC EDGAR, Alpha Vantage, Azure, or any other external endpoint. Only Provider Adapters and the Key Vault adapter (AQ-13) may make outbound calls. Certificate-renewal calls are open (AQ-14). The Key Vault adapter sends only the data key for wrapping and unwrapping.
- **Enforcement:** Provider Adapters and the Key Vault adapter (DR-4, AQ-13); User Trades, Trade Evaluation, and Adaptation modules.
- **Evidence:** NFR-6; TB-2; DR-4; ARCHITECTURE "Persistence" (envelope encryption). ES2026 prompt, "Secrets and crypto".
- **Verification:**
  - A test with outbound HTTP intercepted runs trade evaluation and adaptation and sees no external request carrying trade fields.
  - A dependency check confirms only adapter modules import HTTP or Azure clients.

**SC-017 — Fixed outbound destinations with TLS verification, deadlines, and byte caps**
- **Status:** ACTIVE
- **Rule:** Provider Adapters MUST:
  - Call only the configured SEC EDGAR and Alpha Vantage HTTPS origins.
  - Set `redirect: 'error'` or check each redirect target.
  - Attach an `AbortSignal` deadline of 60 s to every request.
  - Cap each response at 50 MiB while streaming.
  - Never set `rejectUnauthorized: false` or `NODE_TLS_REJECT_UNAUTHORIZED=0`.
- **Enforcement:** Provider Adapters.
- **Evidence:** TB-2; FR-7.2; OD-008 (size), OD-016 (deadline). Node.js 26 prompt, "Outbound requests, fetch, and TLS" and "Streams, buffers, and limits". ES2026 prompt, "URLs and SSRF".
- **Verification:** Test cases against mock servers:
  - A cross-host redirect errors.
  - A stalled server makes the call abort.
  - A response over 50 MiB is cut off and rejected.
  - A grep finds no TLS-verification override.

**SC-018 — Provider responses validated before storage**
- **Status:** ACTIVE
- **Rule:** Every EDGAR and Alpha Vantage response MUST be parsed with a strict Zod schema before any write to Postgres. A failing response MUST NOT be stored, and the job step MUST be recorded as failed.
- **Enforcement:** Provider Adapters and Market Data, Buffett History, and Fundamentals modules.
- **Evidence:** TB-2. Zod 4 prompt, "Parsing, coercion, and defaults". ES2026 prompt, "Structured data and queries".
- **Verification:** A fixture with a non-numeric price or a missing field causes no write and a failed job-step record.

**SC-019 — ML worker reachable only by Unix domain socket**
- **Status:** ACTIVE
- **Rule:** The server MUST reach the worker only through the Unix socket on the shared volume. The socket file MUST have mode 0660 and be owned by the group shared only by the server and worker. The worker container MUST NOT listen on any TCP port, and MUST NOT publish ports.
- **Enforcement:** ML Worker adapter (Model Lifecycle, Scoring); worker container and Docker configuration.
- **Evidence:** TB-4; ARCHITECTURE "ML worker". Node.js 26 prompt, "Filesystem and paths". Python 3.13 prompt, "File access and cleanup".
- **Verification:**
  - Inside the worker container, `ss -ltn` shows no listeners.
  - `stat` on the socket shows mode 0660 and the shared group.
  - `docker inspect` shows no published ports for the worker.

**SC-020 — Minimal, validated worker messages**
- **Status:** ACTIVE
- **Rule:** Each worker job message MUST contain only the fields that job needs. Account, session, and credential data MUST NOT be sent. Owner trades MUST be sent only to proposal-training jobs. Both sides MUST validate the length prefix against the 16 MiB message cap before reading, and MUST check the JSON against a strict schema. The worker MUST reject duplicate keys and non-finite numbers.
- **Enforcement:** ML Worker adapter (server) and worker message handler (Python).
- **Evidence:** TB-4; ARCHITECTURE "Proposals" and "ML worker"; OD-016. Zod 4 prompt, "Object and collection schemas". Python 3.13 prompt, "Data formats and deserialization".
- **Verification:**
  - Contract tests reject extra fields, duplicate keys, and `NaN`.
  - A scoring payload contains no trade records.
  - An oversized length prefix is rejected before any read.

**SC-021 — Model file integrity and safe loading**
- **Status:** ACTIVE
- **Rule:** The worker MUST compute the model file's SHA-256 and compare it with the checksum in the job message, taken from Postgres, before loading. On mismatch it MUST refuse to load. Model files MUST be in a non-executable format and MUST NOT be loaded with `pickle`, `joblib`, or any API that reconstructs Python objects.
- **Enforcement:** ML Worker (Python) model loader; Model Lifecycle (records the checksum).
- **Evidence:** ARCHITECTURE "Persistence" and "Trade model" (LightGBM text format); OD-010, OD-019. Python 3.13 prompt, "Data formats and deserialization".
- **Verification:**
  - A model file with one changed byte fails to load and returns an error.
  - A grep of the worker finds no `pickle`, `joblib.load`, or `shelve`.

**SC-022 — Generic error responses and redacted logs**
- **Status:** ACTIVE
- **Rule:** Client error responses MUST NOT include stack traces, Prisma error metadata, or validation payload contents. Logs MUST redact:
  - `cookie`, `set-cookie`, and `authorization` headers.
  - Passwords.
  - The Alpha Vantage API key, including in request URLs.
  - Key Vault secrets and the data key.
  - User trade fields.
- **Enforcement:** Fastify `setErrorHandler`, `schemaErrorFormatter`, and logger configuration; worker logging.
- **Evidence:** NFR-5, NFR-6. Fastify 5.12 prompt, "Replies, routing, and errors" and "Availability and operations". Node.js 26 prompt, "Diagnostics, inspector, and logging". Prisma 7.10 prompt, "Logging, migrations, and supply chain".
- **Verification:**
  - A forced exception returns a generic body.
  - Logs from login and daily-update tests contain no password, cookie, or API key value.
  - Prisma query-parameter logging is off.

**SC-023 — Security headers and CSP with no third-party origins**
- **Status:** ACTIVE
- **Rule:** HTML responses MUST carry a Content-Security-Policy that allows no third-party script, style, or font origins. Responses MUST carry `Strict-Transport-Security`. Security headers MUST be applied at the root scope.
- **Enforcement:** Fastify server (root-scope header hook) serving the React build.
- **Evidence:** NFR-5, NFR-8; ARCHITECTURE (same-origin SPA). Fastify 5.12 prompt, "Replies, routing, and errors". Exact directive and max-age values are not set here.
- **Verification:** The SPA HTML response has a CSP with no external hosts and an HSTS header.

**SC-024 — Public-only static serving**
- **Status:** ACTIVE
- **Rule:** The static server MUST serve only the React build output from a dedicated root with dotfiles off and directory listing off. Model files, the socket volume, and config MUST stay outside that root.
- **Enforcement:** Fastify static asset registration.
- **Evidence:** ARCHITECTURE (SPA served by Fastify; model files on disk). Fastify 5.12 prompt, "Authorization and lifecycle hooks".
- **Verification:** Requests for `/.env`, encoded traversal paths, and the model directory return 404.

**SC-025 — No unsafe HTML rendering in React**
- **Status:** ACTIVE
- **Rule:** The client MUST render API and provider-derived values (symbols, feature reasons, causes, CSV errors) as JSX text. It MUST NOT use `dangerouslySetInnerHTML`, and it MUST NOT spread untrusted objects into DOM props.
- **Enforcement:** Web Client (React).
- **Evidence:** DESIGN Components (score row, no-result row); DR-1. React 19 prompt, "Rendering and HTML" and "URLs, props, and components".
- **Verification:** A grep or lint finds no `dangerouslySetInnerHTML`. A symbol containing `<img onerror>` renders as inert text.

**SC-026 — No authorization or secrets in the client**
- **Status:** ACTIVE
- **Rule:** The client MUST NOT hold secrets or session tokens in `localStorage` or `sessionStorage`, or in public build variables. It MUST clear cached user data on logout and session expiry. The server MUST enforce every check.
- **Enforcement:** Web Client; API.
- **Evidence:** DR-1; FR-6.5; SC-002 (HTTP-only cookie). React 19 prompt, "State and browser trust". ES2026 prompt, "Storage and messages".
- **Verification:**
  - A built-bundle scan finds no key or DB string.
  - Browser storage holds no session value after login.
  - After logout, client state contains no trade data.

**SC-027 — React Server Components off; React kept current**
- **Status:** ACTIVE
- **Rule:** The client build MUST NOT include `react-server-dom-*` packages or `'use server'` functions. `react` and `react-dom` MUST be the same version, on the newest 19.x minor and patch.
- **Enforcement:** Web Client build.
- **Evidence:** ARCHITECTURE ("React 19"; static SPA); OD-003. React 19 prompt, "Refs, errors, and dependencies".
- **Verification:** `npm ls react react-dom react-server-dom-webpack react-server-dom-turbopack react-server-dom-parcel` (package manager: AQ-15) shows matching current versions and no RSC packages.

**SC-028 — Startup configuration and secrets validated and frozen**
- **Status:** ACTIVE
- **Rule:** At startup, the server MUST read its secrets (DB password, Alpha Vantage API key, session secret, CSRF key) from Azure Key Vault with the managed identity, validate all configuration with Zod, and freeze it. If any value is missing or invalid, it MUST exit non-zero before listening. Secrets MUST NOT be passed in command-line arguments, env files, or image layers, or stored in the repository.
- **Enforcement:** Fastify server bootstrap.
- **Evidence:** NFR-4; ARCHITECTURE "Azure Key Vault"; OD-011. Node.js 26 prompt, "Startup configuration and environment". Zod 4 prompt (`z.stringbool`).
- **Verification:** Startup with Key Vault unreachable, or with a missing secret, exits non-zero before listening.

**SC-029 — HTTP server timeouts, strict parsing, no proxy trust**
- **Status:** ACTIVE
- **Rule:** The server MUST set nonzero timeouts: `connectionTimeout` 10 s, `requestTimeout` 30 s, `handlerTimeout` 25 s. It MUST NOT enable `insecureHTTPParser`. It MUST keep `trustProxy: false`, because Fastify terminates TLS with no proxy.
- **Enforcement:** Fastify server configuration.
- **Evidence:** ARCHITECTURE "TLS and certificates" (Fastify serves HTTPS directly); OD-016. Fastify 5.12 prompt, "Availability and operations" and "Proxy trust and outbound requests". Node.js 26 prompt, "HTTP server hardening".
- **Verification:** A config test asserts all three timeouts are greater than 0, `trustProxy === false`, and there is no insecure-parser flag.

**SC-030 — HTTPS only, no plain HTTP listener**
- **Status:** ACTIVE
- **Rule:** Fastify MUST listen only with HTTPS, using the public CA certificate. No plain HTTP listener MUST exist on any container, including for redirects or certificate challenges.
- **Enforcement:** Fastify server startup; Docker port configuration; certificate renewal (ACME DNS-01; component open: AQ-14).
- **Evidence:** NFR-5; TB-1; ARCHITECTURE "TLS and certificates"; OD-004, OD-017. Node.js 26 prompt, "Outbound requests, fetch, and TLS" (keep TLS defaults). Fastify 5.12 prompt, "Proxy trust and outbound requests".
- **Verification:**
  - A plain HTTP connection to every published VM port fails.
  - `docker ps` shows only the HTTPS port published.
  - Each container's listeners show no HTTP-only socket.

**SC-031 — Verified TLS to Postgres; Postgres not published**
- **Status:** ACTIVE
- **Rule:** The `PrismaPg` connection MUST use TLS with `sslmode=verify-full` and the private CA in `sslrootcert`. It MUST NOT use `ssl: { rejectUnauthorized: false }`, `no-verify`, `require`, or `prefer`. The Postgres container MUST NOT publish ports outside the internal Docker network. Postgres MUST reject non-TLS connections.
- **Enforcement:** Prisma datasource configuration; Postgres server configuration (`hostssl` only); Docker network configuration.
- **Evidence:** NFR-5; TB-3; ARCHITECTURE "TLS and certificates"; OD-004. Prisma 7.10 prompt, "Connections and tenant boundaries".
- **Verification:**
  - A non-TLS `psql` connection is refused.
  - A connection with the wrong hostname or CA fails.
  - `docker inspect` shows no published Postgres port.

**SC-032 — Non-root, digest-pinned, multi-stage, read-only images**
- **Status:** ACTIVE
- **Rule:** Each Dockerfile MUST:
  - Pin base images by digest on a supported release line (Node.js 26, Python 3.13).
  - Use multi-stage builds, with `npm ci --omit=dev` for the server (package manager: AQ-15).
  - Run as a numeric non-root `USER`.
  - Use exec-form `ENTRYPOINT` or `CMD`.
  - Run with a read-only root filesystem.
- **Enforcement:** Server and worker Dockerfiles; container run configuration.
- **Evidence:** ARCHITECTURE "Runtime and deployment environment". Dockerfile 1 prompt, "Syntax and base images", "Build stages", "Users and filesystem", and "Runtime command". Node.js 26 prompt, "Deployment and operations". Python 3.13 prompt, "Supported builds and deployment".
- **Verification:**
  - `docker image inspect` shows a non-root numeric user.
  - Each `FROM` line has `@sha256:`.
  - Running containers report `ReadonlyRootfs: true`.

**SC-033 — No secrets in images or build args**
- **Status:** ACTIVE
- **Rule:** Credentials (DB, Alpha Vantage, session, CSRF, Azure) MUST NOT appear in `ARG`, `ENV`, `COPY`ed files, labels, or image layers. Build-time credentials MUST use `--mount=type=secret`. `.dockerignore` MUST exclude env files and keys.
- **Enforcement:** Dockerfiles and `.dockerignore`.
- **Evidence:** ARCHITECTURE "Azure Key Vault"; OD-011. Dockerfile 1 prompt, "Build context", "Build secrets and SSH", and "Arguments and environment".
- **Verification:** `docker history --no-trunc` and a layer secret scan find no credentials.

**SC-034 — Image vulnerability scanning**
- **Status:** ACTIVE
- **Rule:** Every built image MUST be scanned with Trivy. A high or critical finding MUST fail the build.
- **Enforcement:** Image build pipeline (GitHub Actions with Trivy).
- **Evidence:** ARCHITECTURE "CI"; OD-015. Dockerfile 1 prompt, "Verification and deployment".
- **Verification:** The build log shows the scan step. A seeded known-vulnerable base fails the build.

**SC-035 — TypeScript strictness at trust boundaries**
- **Status:** ACTIVE
- **Rule:** TypeScript requirements:
  - Server and client MUST compile with `strict`, `noUncheckedIndexedAccess`, `exactOptionalPropertyTypes`, and `useUnknownInCatchVariables`, with a full type check (`noCheck: false`) before release.
  - Request bodies, provider responses, worker replies, and configuration MUST be typed `unknown` until Zod parses them.
  - Code MUST NOT use type assertions to promote untrusted data, or `@ts-ignore` and `@ts-nocheck`.
- **Enforcement:** `tsconfig` for server and client; code review.
- **Evidence:** ARCHITECTURE ("written in TypeScript 7"). TypeScript 7 prompt, "Compiler checks" and "Trust boundaries".
- **Verification:**
  - `tsc --noEmit` passes with the flags above.
  - A grep finds no `@ts-ignore`, `@ts-nocheck`, or `as unknown as` at boundary modules.

**SC-036 — Account provisioning, password change, logout**
- **Status:** ACTIVE
- **Rule:** Account lifecycle requirements:
  - No HTTP route MUST create an account.
  - The host CLI MUST refuse to run when an account exists, and MUST enforce a password of at least 15 characters.
  - Password change MUST verify the current password, enforce the same minimum, rehash with SC-004, and end all other sessions.
  - Logout MUST await `request.session.destroy()` and clear the cookie with matching attributes.
- **Enforcement:** Auth module; account-creation CLI.
- **Evidence:** FR-6.3–6.5; ARCHITECTURE "Authentication and session model"; OD-009. Fastify 5.12 prompt, "Browser state and OAuth".
- **Verification:**
  - No route matches a sign-up path.
  - A second CLI run fails.
  - A password change with a wrong current password returns 403.
  - A different session is rejected after a password change.
  - A logged-out session cookie returns 401.

**SC-037 — Python worker runtime hardening**
- **Status:** ACTIVE
- **Rule:** The worker MUST run on the newest Python 3.13 patch in isolated mode (`-I`), as a non-root user. It MUST NOT unpickle any data from messages, files, or Postgres-derived inputs, and it MUST NOT expose development consoles or `http.server`.
- **Enforcement:** ML worker container entrypoint and code.
- **Evidence:** ARCHITECTURE ("Python ML worker (Python 3.13)"); TB-4. Python 3.13 prompt, "Supported builds and deployment" and "Data formats and deserialization".
- **Verification:**
  - The container entrypoint shows `python -I`.
  - `python --version` reports the newest 3.13 patch.
  - A grep finds no `pickle`, `shelve`, or `http.server` imports.

**SC-038 — 13F XML parsing**
- **Status:** ACTIVE
- **Rule:** 13F information tables MUST be parsed with `fast-xml-parser` using `processEntities: false` and no DTD support. Input over 50 MiB MUST be rejected before parsing, and documents nested deeper than 32 levels MUST be rejected.
- **Enforcement:** Parser options set in one shared module.
- **Evidence:** FR-2.1–2.3; TB-2; ARCHITECTURE "13F XML parser"; OD-018. ES2026 prompt, "Structured data and queries" (bounded parsing).
- **Verification:** Tests feed an entity-expansion document and a deeply nested document and expect rejection.

**SC-039 — Module-boundary build check for version writes**
- **Status:** ACTIVE
- **Rule:** An automated check MUST fail the build when a module other than Model Lifecycle imports the version repository or writes the strategy version, model version, or current-version tables (DR-9). The check MUST also fail on raw SQL (SC-013) that names these tables outside Model Lifecycle.
- **Enforcement:** Build and CI checks (SC-034 pipeline); Model Lifecycle.
- **Evidence:** T-001; DR-3, DR-7, DR-9; FR-5.4. THREAT_MODEL TR-01.
- **Verification:** A test module outside Model Lifecycle that imports the version repository, or issues a raw write to a version table, fails the build.

**SC-040 — Database privileges for version tables**
- **Status:** BLOCKED (OD-020)
- **Rule:** The database principal used by modules other than Model Lifecycle MUST NOT have write privileges on the strategy version, model version, or current-version tables (DR-9). The role design and the migration runner identity are open: OD-020.
- **Enforcement:** Postgres roles and grants; Prisma datasource configuration.
- **Evidence:** T-001, T-022; DR-3, DR-7; TB-3. THREAT_MODEL TR-01, TR-22.
- **Verification:** With the non-owning principal, a write to each version table is refused and version state is unchanged. The owning approve path succeeds.

**SC-041 — Approval bound to the displayed candidate**
- **Status:** ACTIVE
- **Rule:** Approval requirements:
  - The approve request MUST carry the immutable model version identifier, the model-file SHA-256, the strategy version identifier, and the backtest result identifier that the owner was shown (FR-5.8).
  - Model Lifecycle MUST compare each value with the stored candidate inside the same transaction that sets the current version, and MUST refuse the approval if any value differs.
  - Candidate model files and backtest records MUST NOT change after the proposal is shown. A retrain makes a new candidate with a new version identifier.
  - These identifiers follow SC-008 (no coercion or defaults).
- **Enforcement:** Adaptation and Model Lifecycle; approve route schema.
- **Evidence:** T-003; FR-5.3, FR-5.4, FR-5.7, FR-5.8; DR-7. THREAT_MODEL TR-03.
- **Verification:** A test changes the candidate checksum or backtest record between display and approve. The approve call is refused, and the current version is unchanged.

**SC-042 — Restore preconditions**
- **Status:** ACTIVE
- **Rule:** Restore MUST make a version current only if the version is in the Previous state, was approved by the owner before, and has a stored FR-3.5 gate result that passed. Before the change, Model Lifecycle MUST recompute the model-file SHA-256 and compare it with the stored checksum (SC-021, SC-044). It MUST refuse Rejected versions, never-approved candidates, failed-gate versions, and checksum mismatches. The restore request MUST carry the same identifiers as SC-041.
- **Enforcement:** Model Lifecycle; restore route schema.
- **Evidence:** T-023, T-003; FR-3.5, FR-5.6, FR-5.9; DR-7; ARCHITECTURE "Version states". THREAT_MODEL TR-03.
- **Verification:** Restore of a Rejected version, a never-approved candidate, and a version with a changed model file each fail, and the current version is unchanged.

**SC-043 — Version-transition authority and proposal dispatch**
- **Status:** ACTIVE
- **Rule:** Requirements:
  - The Model Lifecycle operation that sets the current version MUST require a persisted owner approval record (SC-041) or owner restore record (SC-042) bound to that exact version. It MUST NOT be callable from Scheduler jobs or event handlers (DR-11).
  - Scheduler jobs and the 13F hand-off MUST only create candidates and proposals.
  - The system MUST create at most one proposal for each 13F accession number. A retried or duplicated hand-off MUST NOT create a second proposal (FR-5.10).
- **Enforcement:** Model Lifecycle, Adaptation, Scheduler.
- **Evidence:** T-005; FR-5.2, FR-5.4, FR-5.10; DR-6, DR-7, DR-11. THREAT_MODEL TR-05.
- **Verification:** A test replays the 13F hand-off twice and runs the quarterly job during an owner proposal request. The current version is unchanged, and there is exactly one proposal for the accession.

**SC-044 — Server checksum authority for model files**
- **Status:** ACTIVE
- **Rule:** The worker MUST write model files only to a staging directory. Model Lifecycle MUST compute the SHA-256 from the bytes it reads after the worker finishes, move the file to the promoted directory, and record that checksum in Postgres. A checksum that the worker reports MUST NOT be stored or used for SC-021. The worker MUST have read-only access to the promoted directory.
- **Enforcement:** Model Lifecycle; ML Worker adapter; Docker volume mounts.
- **Evidence:** T-006, T-003; ARCHITECTURE "Persistence", "ML worker". THREAT_MODEL TR-06.
- **Verification:**
  - A write from the worker identity to the promoted directory fails.
  - A worker reply that contains a checksum field is rejected by the strict reply schema (SC-020).
  - A promoted file with one changed byte is refused at load (SC-021).

**SC-045 — Server-side backtest gate computation**
- **Status:** ACTIVE
- **Rule:** The worker backtest reply MUST contain per-test-year returns, not only a combined total. Model Lifecycle MUST compute the combined return itself, and MUST compare it with the SPY total return that Market Data owns for the same test years. If the reply has a missing year, a non-finite value, or a total that does not match the per-year values, the gate MUST fail closed and results MUST stay hidden (FR-3.5.1).
- **Enforcement:** Model Lifecycle; ML Worker adapter reply schema (SC-020).
- **Evidence:** T-025; FR-3.4.1, FR-3.5, FR-3.5.1, FR-5.3. THREAT_MODEL TR-07.
- **Verification:** A worker reply with an inflated total that does not match its per-year returns leaves results hidden.

**SC-046 — No persistent plaintext of owner trades in the worker**
- **Status:** ACTIVE
- **Rule:** The worker MUST keep owner trade rows from proposal-training messages in memory only. It MUST NOT write them to temporary files, the model store, or logs. Worker logs MUST NOT include training rows or feature rows derived from owner trades.
- **Enforcement:** ML Worker (Python) training code and logging.
- **Evidence:** T-007; NFR-5, NFR-6; SC-020, SC-022. THREAT_MODEL TR-08.
- **Verification:** A proposal training on synthetic trades is followed by a search of the worker filesystem and logs. The synthetic values are not found.

**SC-047 — Protection of trade-derived artifacts and backups**
- **Status:** BLOCKED (OD-021, AQ-13)
- **Rule:** Candidate model files trained with owner trades, and backups or disk snapshots of the model store and Postgres, MUST get the protection that OD-021 selects. The key for any model-file encryption depends on AQ-13.
- **Enforcement:** Model Lifecycle; backup configuration (not yet specified).
- **Evidence:** T-007; NFR-5; SC-015. THREAT_MODEL TR-08.
- **Verification:** A search of a test backup and the model store finds no plaintext synthetic trade value outside the protection that OD-021 selects.

**SC-048 — No network egress from the ML worker**
- **Status:** ACTIVE
- **Rule:** The worker container MUST NOT be attached to any Docker network. Its only channel is the Unix socket (SC-019). It MUST NOT download packages or models at runtime.
- **Enforcement:** Docker run configuration for the worker.
- **Evidence:** T-008, T-009; TB-4; NFR-6; SC-016, SC-019. THREAT_MODEL TR-09.
- **Verification:**
  - `docker inspect` shows the worker with network mode `none`.
  - From inside the worker, an outbound connection to a test endpoint fails.

**SC-049 — Managed-identity scope and IMDS reach**
- **Status:** BLOCKED (OD-022, AQ-13, AQ-14)
- **Rule:** Each Azure permission (secret read, key wrap and unwrap, DNS TXT write) MUST be granted with the narrowest data-plane role that its purpose needs. Only the component that OD-022 names MUST be able to get managed-identity tokens. The Postgres container MUST NOT reach the instance metadata endpoint. The worker has no network (SC-048).
- **Enforcement:** Azure role assignments; host and container network rules.
- **Evidence:** T-009; TB-5, TB-6; ARCHITECTURE "Azure Key Vault", "TLS and certificates". THREAT_MODEL TR-10.
- **Verification:** A review of the effective role assignments matches the recorded grants. A token request from the Postgres container fails.

**SC-050 — Certificate renewal path and expiry monitoring**
- **Status:** BLOCKED (AQ-14, OD-023)
- **Rule:** Certificate renewal MUST follow the AQ-14 placement and the OD-023 resolution of the outbound-allowlist conflict. It MUST NOT add a plain HTTP listener or an HTTP-01 fallback (SC-030). The system MUST warn the owner before the certificate expires. The warning lead time is part of OD-023.
- **Enforcement:** The renewal component (AQ-14).
- **Evidence:** T-010; TB-1, TB-7; SC-016, SC-017, SC-030. THREAT_MODEL TR-11, CF-002.
- **Verification:** The observed renewal placement matches the decision. No port 80 listener exists. A test certificate near expiry causes the warning.

**SC-051 — Associated data binding for encrypted trade fields**
- **Status:** BLOCKED (AQ-13)
- **Rule:** Each AES-256-GCM encryption under SC-015 MUST include the table name, record identifier, and field name as authenticated associated data. A ciphertext moved to a different row or field MUST fail decryption. This rule adds to SC-015 and does not replace it.
- **Enforcement:** The encryption component (AQ-13).
- **Evidence:** T-011; NFR-5; SC-015. THREAT_MODEL TR-12.
- **Verification:** With a test database, a ciphertext swapped between two rows, and between two fields, fails decryption.

**SC-052 — Decrypt caller allowlist**
- **Status:** BLOCKED (AQ-13)
- **Rule:** Only User Trades, Trade Evaluation, and Adaptation MUST be able to call trade-data decryption (DR-10). The encryption component MUST have no outbound path and MUST NOT log plaintext. A dependency check MUST fail the build when another module imports the decrypt interface.
- **Enforcement:** The encryption component (AQ-13); build checks.
- **Evidence:** T-002; NFR-5, NFR-6; SC-015, SC-016, SC-022. THREAT_MODEL TR-02.
- **Verification:** A decrypt call from a non-allowlisted test module fails the build. Captured outbound requests and logs from trade evaluation contain no trade field.

**SC-053 — Login limit policy and persisted counters**
- **Status:** BLOCKED (OD-024)
- **Rule:** The per-account login limit MUST NOT let a remote party deny the owner login for an unbounded time. Login-limit counters MUST survive a server restart. The policy (progressive delay, lockout with host unlock, network allowlist, or second factor) is open: OD-024. SC-005 stays ACTIVE as written until OD-024 is decided.
- **Enforcement:** Auth module, login route.
- **Evidence:** T-013, T-012; FR-6.2; SC-005. THREAT_MODEL TR-13.
- **Verification:** Failed logins from rotating test IPs, then an owner login, behave as OD-024 states. Counters keep their values across a restart.

**SC-054 — Login request origin**
- **Status:** BLOCKED (OD-025)
- **Rule:** The login route MUST refuse cross-site login submissions by the method that OD-025 selects.
- **Enforcement:** Auth module, login route.
- **Evidence:** T-012; SC-006 (login is not covered). THREAT_MODEL TR-13.
- **Verification:** A cross-origin login submission is refused.

**SC-055 — Key Vault outage behavior**
- **Status:** BLOCKED (OD-026)
- **Rule:** Startup, runtime, and restart behavior during a Key Vault or managed-identity outage MUST follow one rule, which OD-026 selects. In all alternatives, secrets MUST NOT be cached on disk (SC-028, SC-033). SC-028 stays ACTIVE as written until OD-026 is decided.
- **Enforcement:** Fastify server bootstrap; Scheduler.
- **Evidence:** T-014; NFR-2, NFR-4; SC-028; ARCHITECTURE "Scheduling". THREAT_MODEL TR-14, CF-001.
- **Verification:** With Key Vault blocked at runtime and during a restart, the observed behavior matches OD-026, and no secret is written to disk.

**SC-056 — Provider data plausibility checks and quarantine**
- **Status:** BLOCKED (OD-027)
- **Rule:** After SC-018 schema checks, Market Data, Buffett History, and Fundamentals MUST run plausibility checks before data reaches labels, features, the FR-3.5 gate, or scores. Records that fail MUST be quarantined, MUST NOT be used, and MUST be shown to the owner (FR-1.6). The checks and their thresholds are open: OD-027. 13F/A handling: OQ-15.
- **Enforcement:** Market Data, Buffett History, Fundamentals.
- **Evidence:** T-015; TB-2; FR-1.6, FR-2.2, FR-2.4, FR-3.5. THREAT_MODEL TR-15.
- **Verification:** A fixture with a schema-valid price jump with no split record is quarantined and does not reach features.

**SC-057 — One training job at a time**
- **Status:** ACTIVE
- **Rule:** At most one training or proposal job MUST run at a time. A proposal request or CSV import that arrives while the same kind of job is running MUST be refused with a clear cause and MUST NOT be queued silently (FR-5.10). The daily update and scoring job MUST have priority over proposal training.
- **Enforcement:** Adaptation, User Trades, Scheduler.
- **Evidence:** T-017, T-016; FR-5.2, NFR-2. THREAT_MODEL TR-17.
- **Verification:** Two proposal requests sent together start one training job. The second request gets the refusal and cause.

**SC-058 — Worker call deadlines and parsing resource limits**
- **Status:** BLOCKED (OD-028)
- **Rule:** Every server call to the worker MUST have a deadline, and the server MUST end the call when the deadline passes. EDGAR XBRL and 13F processing MUST have a concurrency limit. The deadline values, the concurrency limit, and the worker container resource limits are open: OD-028.
- **Enforcement:** ML Worker adapter; Fundamentals; Buffett History; Docker run configuration.
- **Evidence:** T-016, T-017; NFR-2; TB-4; SC-017, SC-038. THREAT_MODEL TR-17.
- **Verification:** A stalled worker call ends at its deadline. Concurrent large provider responses stay inside the configured limit.

**SC-059 — Provider API key kept out of stored records and error bodies**
- **Status:** ACTIVE
- **Rule:** Job-step failure records, error bodies, and stack traces that the system stores or returns MUST NOT contain the Alpha Vantage API key. A request URL MUST have the key removed before it is stored. This rule extends the SC-022 log redaction to these stores.
- **Enforcement:** Provider Adapters; Scheduler job-state writes; Fastify error handler.
- **Evidence:** T-018; SC-022. THREAT_MODEL TR-18.
- **Verification:** A forced Alpha Vantage failure with a test key leaves the key in no job-step record, error body, or log.

**SC-060 — Host password-reset command**
- **Status:** ACTIVE
- **Rule:** The host password-reset command (FR-6.6) MUST apply SC-004, enforce the SC-036 minimum length, and end all sessions. It MUST NOT be reachable over HTTP. Account recovery MUST NOT use direct database edits.
- **Enforcement:** Auth module; account CLI.
- **Evidence:** T-019, T-013; FR-6.3, FR-6.6; SC-004, SC-036. THREAT_MODEL TR-19.
- **Verification:** In a test environment, the reset stores an Argon2id hash at or above SC-004 minimums, and a session from before the reset returns 401.

**SC-061 — Host and Azure operator access**
- **Status:** BLOCKED (OD-029)
- **Rule:** The host access method, the persons who hold it, who can run the account CLI and `docker exec`, and the Azure subscription roles that can reach the VM, disks, and Key Vault MUST be recorded and limited as OD-029 decides.
- **Enforcement:** Azure role assignments; VM access configuration.
- **Evidence:** T-019, T-009; TB-9. THREAT_MODEL TR-19.
- **Verification:** A review of host access and Azure role assignments matches the recorded list.

**SC-062 — Supply-chain integrity**
- **Status:** BLOCKED (AQ-15, OD-030, AQ-17)
- **Rule:** Each runtime MUST use one package manager with a committed lockfile and integrity checks (AQ-15). CI workflows MUST have the minimum permissions they need. Images MUST be deployed by digest only. Registry and deployment path: AQ-17. Image signing or provenance: OD-030. SC-027, SC-032, and SC-034 stay ACTIVE as written.
- **Enforcement:** CI (GitHub Actions); Dockerfiles; deployment.
- **Evidence:** T-020, T-008; TB-8. THREAT_MODEL TR-20, CF-003.
- **Verification:** A review of CI workflow permissions, lockfiles, and the deploy step finds no unpinned install, no broad permission, and no tag-based deploy.

**SC-063 — Hashed session identifiers at rest**
- **Status:** ACTIVE
- **Rule:** The Postgres session store MUST store only a cryptographic hash of each session identifier, never the identifier itself. A stored value MUST NOT work as a session cookie.
- **Enforcement:** Auth module (session store).
- **Evidence:** T-022; SC-002, SC-003. THREAT_MODEL TR-22.
- **Verification:** A value read directly from the session table, sent as the cookie, returns 401.

**SC-064 — Step-up for version changes**
- **Status:** BLOCKED (OD-032)
- **Rule:** Approve and restore MUST require the protection that OD-032 selects (current-password re-entry, or confirmation only). In all alternatives, the confirmation step in `DESIGN.md` (Version change confirmation) applies.
- **Enforcement:** Auth module; approve and restore routes.
- **Evidence:** T-004; FR-5.4, FR-5.6; SC-001, SC-006. THREAT_MODEL TR-04.
- **Verification:** An approve or restore call that does not meet the OD-032 requirement is refused and leaves state unchanged.

**SC-065 — CSP and HSTS values**
- **Status:** BLOCKED (OD-033)
- **Rule:** SC-023 MUST be configured with a `frame-ancestors` directive that blocks foreign framing, and with the CSP directive and HSTS `max-age` values that OD-033 sets. SC-023 stays ACTIVE as written.
- **Enforcement:** Fastify root-scope header hook.
- **Evidence:** T-004; SC-023. THREAT_MODEL TR-04.
- **Verification:** The app does not render inside a cross-origin frame. Header values match OD-033.

**SC-066 — Security event log**
- **Status:** BLOCKED (OD-034)
- **Rule:** The system MUST record these events with time, actor, and version identifiers: login success and failure, rate-limit hits, password change and reset, proposal creation, approve, reject, restore, and current-version change. Application modules MUST NOT be able to change or delete entries. Entries MUST NOT contain the SC-022 redacted values. Storage location and retention: OD-034.
- **Enforcement:** Auth, Adaptation, Model Lifecycle; event-log storage (OD-034).
- **Evidence:** T-001, T-003, T-004, T-013, T-023; SC-022. THREAT_MODEL TR-23.
- **Verification:** Each listed action in a test environment produces an entry. An attempt to alter an entry from an application module fails.

**SC-067 — Required human review of specification and control changes**
- **Status:** BLOCKED (OD-031)
- **Rule:** A change to `REQUIREMENTS.md`, `ARCHITECTURE.md`, `SECURITY.md`, or `DESIGN.md`, or to code that implements an `SC-*` rule, MUST get the human review that OD-031 sets before merge.
- **Enforcement:** Repository settings (`AGENTS.md` Workflow, "Required human review before merge").
- **Evidence:** T-021, T-020. THREAT_MODEL TR-21.
- **Verification:** A sample spec-changing pull request cannot merge without the required review.

## Open Security Decisions

**Open:** the open decisions below are TO BE DECIDED. Each `BLOCKED` rule cites the item that blocks it. Source: `THREAT_MODEL.md` (2026-10-30).

- **OD-020** Database roles for version tables (T-001, T-022; SC-040). Alternatives: (a) separate Postgres roles per owning module; (b) a dedicated schema for version state, writable only by a Model Lifecycle role; (c) build check only (SC-039) with detection by SC-066. Also to decide: the migration runner identity (`AGENTS.md` Workflow, "Database migration process").
- **OD-021** Scope of NFR-5 for trade-derived artifacts and backups (T-007; SC-047). Alternatives: (a) encrypt candidate model files trained with owner trades at rest; (b) store such files only after approval and delete them on rejection; (c) exclude owner trades from training and use them only for evaluation (changes FR-5.1); (d) accept tree-embedded leakage and protect only scratch and backups. Also to decide: the backup and snapshot design for Postgres, the model store, and the wrapped data key.
- **OD-022** Managed-identity scope (T-009; SC-049). Alternatives: (a) one identity with minimal combined grants, and metadata-endpoint access blocked for non-server containers; (b) one user-assigned identity per purpose; (c) move DNS renewal off the VM (AQ-14).
- **OD-023** Outbound allowlist versus ACME renewal (CF-002, T-010; SC-050). Side 1: ARCHITECTURE "TLS and certificates" (renewal by ACME DNS-01 against Azure DNS with the VM managed identity). Side 2: SC-016 and SC-017 (only Provider Adapters to SEC EDGAR and Alpha Vantage, and the Key Vault adapter, may call out). Alternatives: (a) run ACME outside the Fastify process with its own identity and destinations, so SC-016 and SC-017 stay unchanged; (b) run ACME in the server and widen SC-016 and SC-017; (c) managed certificate service outside the VM; (d) manual renewal with expiry warning. Also to decide: the expiry warning lead time. Linked: AQ-14.
- **OD-024** Per-account login limit policy and recovery (T-013, T-012; SC-053). Alternatives: (a) progressive delay instead of hard lockout; (b) keep the hard limit and add a host unlock command; (c) restrict login to an allowlisted source network; (d) add a second factor. User-visible behavior: OQ-16, DQ-8.
- **OD-025** Login CSRF protection (T-012; SC-054). Alternatives: (a) Origin header check plus JSON-only content type; (b) pre-session CSRF token; (c) accept the risk, because one account removes classic login-CSRF impact.
- **OD-026** Key Vault outage behavior (CF-001, T-014; SC-055). Side 1: ARCHITECTURE "Scheduling" (if Key Vault cannot be reached, jobs that need trade data fail and retry; other jobs continue). Side 2: SC-028 (startup exits non-zero before listening if Key Vault is unreachable). Alternatives: (a) startup requires Key Vault, a running server keeps non-trade jobs and the UI during an outage, and a restart waits and retries; (b) startup in a degraded mode without trade-data capability; (c) cache wrapped secrets on disk under a host key (conflicts with SC-028 and SC-033).
- **OD-027** Provider data plausibility checks (T-015; SC-056). To decide: the check set (price jump without a split record, SPY series continuity, CUSIP-to-symbol mapping review, XBRL restatements) and thresholds. A cross-check with a second provider needs an ARCHITECTURE decision.
- **OD-028** Resource limit values (T-016, T-017; SC-058). To decide: worker call deadlines for score, train, and backtest; EDGAR processing concurrency; worker container CPU and memory limits.
- **OD-029** Host and Azure operator access (T-019; SC-061). To decide: host access method, network rules, who may run the account CLI and `docker exec`, and Azure subscription roles.
- **OD-030** Image signing or provenance (T-020; SC-062). Alternatives: (a) sign images and verify at deploy; (b) build provenance attestations; (c) defer and rely on digest pinning (SC-032) and Trivy (SC-034).
- **OD-031** Required human review before merge (T-021; SC-067). Holds `AGENTS.md` Workflow "Required human review before merge: TO BE DECIDED". Alternatives: (a) required review for all changes to the four specification files and to `SC-*` implementation code; (b) code-owner rule on specification files only; (c) specification change log with owner sign-off.
- **OD-032** Step-up for approve and restore (T-004; SC-064, DQ-7). Alternatives: (a) current-password re-entry for approve and restore; (b) confirmation only; (c) re-entry for restore only. If re-entry is chosen, it needs the login limits (SC-005, SC-053).
- **OD-033** CSP directive and HSTS `max-age` values (T-004; SC-065).
- **OD-034** Security event log storage and retention (T-001, T-003, T-004, T-013, T-023; SC-066). Alternatives: (a) append-only Postgres table owned by Auth and Model Lifecycle; (b) an Azure log service (adds a data flow that ARCHITECTURE does not list; needs NFR-6 review). Also to decide: retention period.
- **OD-035** Risk rating rubric (all threats, `THREAT_MODEL.md` DEC-04-rubric). Proposed: a qualitative scale (low, medium, high) for impact on trade-data confidentiality, credential and session security, approval-chain and model integrity, and daily-pipeline availability, and for likelihood. No rating is set until the owner approves a rubric.

These open architecture items affect security rules and share their status (TO BE DECIDED):

- AQ-13 (Key Vault component and trust boundary): SC-015, SC-016, SC-028, SC-047, SC-049, SC-051, SC-052.
- AQ-14 (ACME client placement): SC-016, SC-030, SC-049, SC-050.
- AQ-15 (package manager and lockfile policy; CF-003: `AGENTS.md` Workflow leaves the package manager open, while SC-032 and SC-027 name npm): SC-027, SC-032, SC-062.
- AQ-17 (image registry and deployment path): SC-062.

**Resolved on 2026-10-30.** The owner reviewed and confirmed every value the assistant first proposed at the owner's request (OD-017).

- **OD-001** Technology recorded: ARCHITECTURE "Required Architecture Inputs".
- **OD-002** TypeScript 7: ARCHITECTURE; SC-035.
- **OD-003** Newest React 19.x: SC-027.
- **OD-004** Fastify TLS, separate containers, public cert plus private CA, Azure VM: ARCHITECTURE; SC-030, SC-031.
- **OD-005** Envelope encryption design: ARCHITECTURE "Persistence"; SC-015.
- **OD-006** Session and CSRF: SC-002, SC-003, SC-006.
- **OD-007** Rate limits: SC-005.
- **OD-008** Size limits: SC-010, SC-017.
- **OD-009** Account lifecycle: SC-036.
- **OD-010** SHA-256 checked by the worker: SC-021.
- **OD-011** Secrets in Key Vault: SC-028, SC-033.
- **OD-012** Python 3.13 pack: "Selected Manicode Prompts"; SC-037.
- **OD-013** Zod validator compiler: ARCHITECTURE "API"; SC-007.
- **OD-014** Official `@azure` SDK, no evaluated pack: ARCHITECTURE "Azure Key Vault"; "Material gaps".
- **OD-015** GitHub Actions with Trivy: ARCHITECTURE "CI"; SC-034.
- **OD-016** Timeout and size values: SC-017, SC-020, SC-029.
- **OD-017** Owner confirmed all delegated values; DNS-01 via Azure DNS chosen for renewal: ARCHITECTURE "TLS and certificates"; SC-030.
- **OD-018** `fast-xml-parser`, entities and DTD off, depth 32: SC-038.
- **OD-019** LightGBM text model format: ARCHITECTURE "Trade model"; SC-021.

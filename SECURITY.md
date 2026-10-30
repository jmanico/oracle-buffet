# SECURITY

Owns secure-coding implementation rules. Behavior: `REQUIREMENTS.md`. Interaction and browser behavior: `DESIGN.md`. Components, interfaces, data flows, and trust boundaries: `ARCHITECTURE.md`.

## Secure-Coding Inputs

**Documents read**

- `REQUIREMENTS.md`: read. Updated 2026-10-30 with FR-6.3–6.5 and TLS on every hop in NFR-5.
- `DESIGN.md`: read. Updated 2026-10-30 with the Change Password view and the Log out action.
- `ARCHITECTURE.md`: read. Updated 2026-10-30 with the stack versions, topology, and "Security-relevant decisions".
- Existing `SECURITY.md`: read. All `SC-###` and `OD-###` IDs are kept.
- Security notes in the task: one user at a time, no admins, no social features.

**Confirmed project facts used by these rules**

- **Users and access:**
  - One owner account, password login, one role. No sign-up, admin, or social features (FR-6.1–6.3; security notes).
  - The account is created by a host CLI command that refuses to run if an account exists (FR-6.3; ARCHITECTURE "Account lifecycle").
  - Password change requires the current password and ends other sessions. Logout destroys the session. Minimum password length is 15 (FR-6.4, FR-6.5; ARCHITECTURE).
- **Stack** (ARCHITECTURE inputs table):
  - Fastify 5.12 on Node.js 26, in TypeScript 7.
  - Zod 4 for validation and Prisma 7.10 for data access.
  - React 19 client on the newest 19.x, in TypeScript 7.
  - Python 3.13 ML worker.
- **Deployment:** one Azure VM with three Docker containers (server, worker, Postgres) on one internal Docker network (ARCHITECTURE).
- **Validation:** Fastify validates every request part through a Zod 4 validator compiler. Zod schemas are the single source of request contracts (ARCHITECTURE "API").
- **Sessions:**
  - `@fastify/cookie` and `@fastify/session` with a Postgres store.
  - Cookie is `__Host-` prefixed, `Secure`, `HttpOnly`, `SameSite=Strict`.
  - 30-minute idle timeout and 8-hour maximum.
  - `@fastify/csrf-protection` in session mode with a header token (ARCHITECTURE).
- **Login rate limit:** `@fastify/rate-limit` with 5 attempts per 15 minutes per IP and 20 failed attempts per hour per account. In-memory store, one instance (ARCHITECTURE).
- **Input limits:** JSON body 64 KiB. CSV as a `text/csv` body of at most 1 MiB and 10,000 data rows. Provider responses at most 50 MiB (ARCHITECTURE).
- **TLS:** on every network hop with no plain HTTP listener (NFR-5).
  - Fastify terminates browser TLS with a public CA certificate. Renewal uses DNS-01 or TLS-ALPN-01.
  - Postgres TLS uses a private CA with `sslmode=verify-full` and `sslrootcert`.
  - Postgres is not published outside the Docker network (ARCHITECTURE; TB-1, TB-3).
- **Trade-data encryption:**
  - Envelope encryption: an AES-256-GCM data key in the server, wrapped by an Azure Key Vault key.
  - The server authenticates to Key Vault with the VM managed identity.
  - Encrypted fields: all trade fields and trade success results.
  - The wrapping key rotates yearly, and the data key is rewrapped.
  - Jobs that need trade data fail and retry when Key Vault cannot be reached (NFR-5; ARCHITECTURE).
- **Secrets:** the DB password, Alpha Vantage key, session secret, and CSRF key come from Key Vault at startup. No secrets in images, env files, or the repository (ARCHITECTURE).
- **Outbound:** HTTPS to SEC EDGAR and Alpha Vantage only. Responses are validated before storage. No trade data leaves the host (TB-2; NFR-6; FR-7.2).
- **ML worker:**
  - Reached through a Unix socket on a shared volume. The socket file is mode 0660 and owned by a group shared by the server and worker only.
  - Messages are length-prefixed JSON, validated on both sides.
  - The worker receives only the data each job needs (TB-4; ARCHITECTURE).
- **Model files:** a non-executable format (no pickle) with a SHA-256 checksum in Postgres. The worker verifies the checksum before loading (ARCHITECTURE).
- **Containers:** multi-stage builds, digest-pinned base images, non-root numeric users, read-only root filesystems, and an image scan that fails on high or critical findings (ARCHITECTURE).
- **State-changing owner actions:** trade entry and import, proposal approve, reject, and request, version restore, password change, logout (FR-4.2, FR-5.2, FR-5.4, FR-5.6, FR-6.4, FR-6.5).
- **Static client and fonts:** the static SPA is served same origin by Fastify, and no external fonts are loaded (ARCHITECTURE; NFR-8).

**Material gaps and conflicts**

- Several decisions in ARCHITECTURE are marked (DELEGATED): the assistant chose them at the owner's request. They are authoritative now, but the owner has not reviewed each value (OD-017).
- No evaluated pack covers the Azure Key Vault SDK or managed-identity client (OD-014).
- The CI system that runs image scans and grep checks is UNKNOWN (OD-015).
- Server timeout and outbound deadline values are UNKNOWN (OD-016).
- 13F information tables are XML. The Node XML parser is not chosen, and no evaluated pack covers it (OD-018).
- The model file format and library are not named beyond "non-executable, no pickle" (OD-019).

## Selected Manicode Prompts

**Corpus root:** `/Users/jmanico/Dropbox/github/aegis/secure-coding-knowledge`

Resolved with `python3 scripts/resolve_prompt.py js/nodejs/26 js/react/19 js/typescript/7 js/fastify/5.12 js/zod/4 js/prisma/7.10 infra/dockerfile/1 python/python/3.13 --json`.

**Direct selection evidence** (all from ARCHITECTURE "Required Architecture Inputs")

- `js/nodejs/26`: **SELECTED**. "Fastify 5.12 on Node.js 26".
- `js/react/19`: **SELECTED**. "React 19 (newest 19.x minor and patch)". Candidate input `js/react/` resolved to 19.
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
- **Evidence:** ARCHITECTURE "Authentication and session model" and "Session and CSRF". Fastify 5.12 prompt, "Browser state and OAuth".
- **Verification:**
  - The `Set-Cookie` header on login matches the attributes above.
  - A session idle for more than 30 minutes is rejected.
  - A continuously active session older than 8 hours is rejected.

**SC-003 — Session regeneration at login and privilege change**
- **Status:** ACTIVE
- **Rule:** The server MUST await `request.session.regenerate()` after a successful login and after a password change. A session ID from before login MUST NOT stay valid.
- **Enforcement:** Auth module.
- **Evidence:** FR-6.2, FR-6.4; ARCHITECTURE "Session and CSRF". Fastify 5.12 prompt, "Browser state and OAuth".
- **Verification:** A session cookie that existed before login returns 401 after login. The cookie value changes after a password change.

**SC-004 — Argon2id password storage**
- **Status:** ACTIVE
- **Rule:** The owner password MUST be stored only as an Argon2id hash with a unique random salt of at least 16 bytes. Parameters MUST be at least `m=19456` KiB, `t=2`, `p=1`, stored with the hash. A fast digest or reversible encryption MUST NOT be used.
- **Enforcement:** Auth module and the account-creation CLI.
- **Evidence:** NFR-5; ARCHITECTURE session model. ES2026 prompt, "Secrets and crypto".
- **Verification:** The stored verifier shows Argon2id with parameters at or above these minimums. Two hashes of the same password differ.

**SC-005 — Login rate limiting**
- **Status:** ACTIVE
- **Rule:** The login route MUST enforce two limits:
  - An early per-IP limit of 5 attempts per 15 minutes with `@fastify/rate-limit`, keyed by the default canonical IP key generator.
  - A separate account limit of 20 failed attempts per hour, created with `fastify.createRateLimit()` and called from `preHandler`.
  - The two limits MUST use distinct key prefixes. `skipOnError` MUST stay `false`. Keys MUST NOT come from forwarding headers.
- **Enforcement:** Auth module, login route.
- **Evidence:** ARCHITECTURE "Login rate limit". Fastify 5.12 prompt, "Availability and operations".
- **Verification:** Test cases:
  - The 6th attempt from one IP within 15 minutes gets 429.
  - The 21st failed attempt within an hour from rotating IPs gets 429.
  - Each budget is tested on its own.

**SC-006 — CSRF protection on state-changing routes**
- **Status:** ACTIVE
- **Rule:** `@fastify/csrf-protection` MUST be registered after the session plugin with `sessionPlugin: '@fastify/session'`. `fastify.csrfProtection` MUST run in `onRequest` with a header-only `getToken` on every state-changing route: trade entry, CSV import, approve, reject, restore, proposal request, password change, logout.
- **Enforcement:** API (Fastify routes).
- **Evidence:** ARCHITECTURE "Session and CSRF"; FR-4.2, FR-5.2, FR-5.4, FR-5.6, FR-6.4, FR-6.5. Fastify 5.12 prompt, "Browser state and OAuth". ES2026 prompt, "Storage and messages".
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
- **Evidence:** ARCHITECTURE "API"; TB-1. Fastify 5.12 prompt, "Request schemas and validation". Zod 4 prompt, "Object and collection schemas" and "Parsing, coercion, and defaults".
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
  - Require exactly the header `symbol, side, date, quantity, price`.
  - Validate every row with a strict schema: `side` is `buy` or `sell`, `date` is `YYYY-MM-DD`, quantity and price are finite and in range.
  - Write nothing if any row fails, inside one transaction.
  - Return the row number and cause without echoing the payload.
- **Enforcement:** User Trades module (CSV import).
- **Evidence:** FR-4.2, FR-4.2.1; ARCHITECTURE "CSV import". Zod 4 prompt, "Output and disclosure". ES2026 prompt, "Text and numbers". Prisma 7.10 prompt, "Transactions and concurrency".
- **Verification:** A file with one bad middle row stores zero rows and names that row number. A wrong header is rejected.

**SC-010 — Request and CSV size limits**
- **Status:** ACTIVE
- **Rule:** Size limits:
  - The server `bodyLimit` MUST be 64 KiB.
  - The CSV route MUST accept only `text/csv`, with a route `bodyLimit` of 1 MiB and a cap of 10,000 data rows. The parser MUST be registered with `parseAs: 'string'`.
  - Other media types MUST be rejected.
- **Enforcement:** Fastify server configuration; User Trades CSV route.
- **Evidence:** ARCHITECTURE "Input limits". Fastify 5.12 prompt, "Parsing and uploads".
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
  - The data key MUST be stored only wrapped by the Azure Key Vault key, and MUST NOT be logged or written unwrapped to disk.
- **Enforcement:** User Trades and Trade Evaluation modules (encryption adapter, DR-4).
- **Evidence:** NFR-5; TB-3; ARCHITECTURE "Trade data encryption". Node.js 26 prompt, "Cryptography and secrets". ES2026 prompt, "Secrets and crypto". The Key Vault client itself: OD-014.
- **Verification:**
  - A direct SQL `SELECT` on the trade and result tables shows no plaintext field values.
  - A tampered ciphertext fails decryption, and nothing is returned.
  - Two encryptions of the same value produce different ciphertexts.

**SC-016 — No user trade data leaves the host**
- **Status:** ACTIVE
- **Rule:** User trade records and trade success results MUST NOT be included in any request to SEC EDGAR, Alpha Vantage, Azure, or any other external endpoint. Only Provider Adapters and the Key Vault adapter may make outbound calls. The Key Vault adapter sends only the data key for wrapping and unwrapping.
- **Enforcement:** Provider Adapters and the Key Vault adapter (DR-4); User Trades, Trade Evaluation, and Adaptation modules.
- **Evidence:** NFR-6; TB-2; DR-4; ARCHITECTURE "Trade data encryption" (envelope). ES2026 prompt, "Secrets and crypto".
- **Verification:**
  - A test with outbound HTTP intercepted runs trade evaluation and adaptation and sees no external request carrying trade fields.
  - A dependency check confirms only adapter modules import HTTP or Azure clients.

**SC-017 — Fixed outbound destinations with TLS verification, deadlines, and byte caps**
- **Status:** ACTIVE
- **Rule:** Provider Adapters MUST:
  - Call only the configured SEC EDGAR and Alpha Vantage HTTPS origins.
  - Set `redirect: 'error'` or check each redirect target.
  - Attach an `AbortSignal` deadline to every request.
  - Cap each response at 50 MiB while streaming.
  - Never set `rejectUnauthorized: false` or `NODE_TLS_REJECT_UNAUTHORIZED=0`.
- **Enforcement:** Provider Adapters.
- **Evidence:** TB-2; FR-7.2; ARCHITECTURE "Input limits". Node.js 26 prompt, "Outbound requests, fetch, and TLS" and "Streams, buffers, and limits". ES2026 prompt, "URLs and SSRF". Deadline values: OD-016.
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
- **Evidence:** TB-4; ARCHITECTURE "ML worker protocol". Node.js 26 prompt, "Filesystem and paths". Python 3.13 prompt, "File access and cleanup".
- **Verification:**
  - Inside the worker container, `ss -ltn` shows no listeners.
  - `stat` on the socket shows mode 0660 and the shared group.
  - `docker inspect` shows no published ports for the worker.

**SC-020 — Minimal, validated worker messages**
- **Status:** ACTIVE
- **Rule:** Each worker job message MUST contain only the fields that job needs. Account, session, and credential data MUST NOT be sent. Owner trades MUST be sent only to proposal-training jobs. Both sides MUST validate the length prefix against a size cap before reading, and MUST check the JSON against a strict schema. The worker MUST reject duplicate keys and non-finite numbers.
- **Enforcement:** ML Worker adapter (server) and worker message handler (Python).
- **Evidence:** TB-4; ARCHITECTURE "Proposals" and "ML worker protocol". Zod 4 prompt, "Object and collection schemas". Python 3.13 prompt, "Data formats and deserialization". Message size cap value: OD-016.
- **Verification:**
  - Contract tests reject extra fields, duplicate keys, and `NaN`.
  - A scoring payload contains no trade records.
  - An oversized length prefix is rejected before any read.

**SC-021 — Model file integrity and safe loading**
- **Status:** ACTIVE
- **Rule:** The worker MUST compute the model file's SHA-256 and compare it with the checksum in the job message, taken from Postgres, before loading. On mismatch it MUST refuse to load. Model files MUST NOT be loaded with `pickle`, `joblib`, or any API that reconstructs Python objects.
- **Enforcement:** ML Worker (Python) model loader; Model Lifecycle (records the checksum).
- **Evidence:** ARCHITECTURE "Persistence" and "Model files". Python 3.13 prompt, "Data formats and deserialization". Format and library: OD-019.
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
- **Evidence:** DR-1; FR-6.5; ARCHITECTURE HTTP-only cookie. React 19 prompt, "State and browser trust". ES2026 prompt, "Storage and messages".
- **Verification:**
  - A built-bundle scan finds no key or DB string.
  - Browser storage holds no session value after login.
  - After logout, client state contains no trade data.

**SC-027 — React Server Components off; React kept current**
- **Status:** ACTIVE
- **Rule:** The client build MUST NOT include `react-server-dom-*` packages or `'use server'` functions. `react` and `react-dom` MUST be the same version, on the newest 19.x minor and patch.
- **Enforcement:** Web Client build.
- **Evidence:** ARCHITECTURE ("React 19 (newest 19.x minor and patch)"; static SPA). React 19 prompt, "Refs, errors, and dependencies".
- **Verification:** `npm ls react react-dom react-server-dom-webpack react-server-dom-turbopack react-server-dom-parcel` shows matching current versions and no RSC packages.

**SC-028 — Startup configuration and secrets validated and frozen**
- **Status:** ACTIVE
- **Rule:** At startup, the server MUST read its secrets from Azure Key Vault with the managed identity, validate all configuration with Zod, and freeze it. If any value is missing or invalid, it MUST exit non-zero before listening. Secrets MUST NOT be passed in command-line arguments, env files, or image layers.
- **Enforcement:** Fastify server bootstrap.
- **Evidence:** NFR-4; ARCHITECTURE "Secrets". Node.js 26 prompt, "Startup configuration and environment". Zod 4 prompt (`z.stringbool`). Key Vault client: OD-014.
- **Verification:** Startup with Key Vault unreachable, or with a missing secret, exits non-zero before listening.

**SC-029 — HTTP server timeouts, strict parsing, no proxy trust**
- **Status:** ACTIVE
- **Rule:** The server MUST set nonzero `requestTimeout`, `connectionTimeout`, and `handlerTimeout`. It MUST NOT enable `insecureHTTPParser`. It MUST keep `trustProxy: false`, because Fastify terminates TLS with no proxy.
- **Enforcement:** Fastify server configuration.
- **Evidence:** ARCHITECTURE "TLS everywhere" (Fastify serves HTTPS directly). Fastify 5.12 prompt, "Availability and operations" and "Proxy trust and outbound requests". Node.js 26 prompt, "HTTP server hardening". Values: OD-016.
- **Verification:** A config test asserts all three timeouts are greater than 0, `trustProxy === false`, and there is no insecure-parser flag.

**SC-030 — HTTPS only, no plain HTTP listener**
- **Status:** ACTIVE
- **Rule:** Fastify MUST listen only with HTTPS, using the public CA certificate. No plain HTTP listener MUST exist on any container, including for redirects or certificate challenges.
- **Enforcement:** Fastify server startup; Docker port configuration; certificate renewal (DNS-01 or TLS-ALPN-01).
- **Evidence:** NFR-5; TB-1; ARCHITECTURE "TLS everywhere". Node.js 26 prompt, "Outbound requests, fetch, and TLS" (keep TLS defaults). Fastify 5.12 prompt, "Proxy trust and outbound requests".
- **Verification:**
  - A plain HTTP connection to every published VM port fails.
  - `docker ps` shows only the HTTPS port published.
  - Each container's listeners show no HTTP-only socket.

**SC-031 — Verified TLS to Postgres; Postgres not published**
- **Status:** ACTIVE
- **Rule:** The `PrismaPg` connection MUST use TLS with `sslmode=verify-full` and the private CA in `sslrootcert`. It MUST NOT use `ssl: { rejectUnauthorized: false }`, `no-verify`, `require`, or `prefer`. The Postgres container MUST NOT publish ports outside the internal Docker network. Postgres MUST reject non-TLS connections.
- **Enforcement:** Prisma datasource configuration; Postgres server configuration (`hostssl` only); Docker network configuration.
- **Evidence:** NFR-5; TB-3; ARCHITECTURE "TLS everywhere". Prisma 7.10 prompt, "Connections and tenant boundaries".
- **Verification:**
  - A non-TLS `psql` connection is refused.
  - A connection with the wrong hostname or CA fails.
  - `docker inspect` shows no published Postgres port.

**SC-032 — Non-root, digest-pinned, multi-stage, read-only images**
- **Status:** ACTIVE
- **Rule:** Each Dockerfile MUST:
  - Pin base images by digest on a supported release line (Node.js 26, Python 3.13).
  - Use multi-stage builds, with `npm ci --omit=dev` for the server.
  - Run as a numeric non-root `USER`.
  - Use exec-form `ENTRYPOINT` or `CMD`.
  - Run with a read-only root filesystem.
- **Enforcement:** Server and worker Dockerfiles; container run configuration.
- **Evidence:** ARCHITECTURE "Containers". Dockerfile 1 prompt, "Syntax and base images", "Build stages", "Users and filesystem", and "Runtime command". Node.js 26 prompt, "Deployment and operations". Python 3.13 prompt, "Supported builds and deployment".
- **Verification:**
  - `docker image inspect` shows a non-root numeric user.
  - Each `FROM` line has `@sha256:`.
  - Running containers report `ReadonlyRootfs: true`.

**SC-033 — No secrets in images or build args**
- **Status:** ACTIVE
- **Rule:** Credentials (DB, Alpha Vantage, session, CSRF, Azure) MUST NOT appear in `ARG`, `ENV`, `COPY`ed files, labels, or image layers. Build-time credentials MUST use `--mount=type=secret`. `.dockerignore` MUST exclude env files and keys.
- **Enforcement:** Dockerfiles and `.dockerignore`.
- **Evidence:** ARCHITECTURE "Secrets". Dockerfile 1 prompt, "Build context", "Build secrets and SSH", and "Arguments and environment".
- **Verification:** `docker history --no-trunc` and a layer secret scan find no credentials.

**SC-034 — Image vulnerability scanning**
- **Status:** ACTIVE
- **Rule:** Every built image MUST be scanned with Docker Scout, Trivy, or Grype. A high or critical finding MUST fail the build.
- **Enforcement:** Image build pipeline (CI system: OD-015).
- **Evidence:** ARCHITECTURE "Containers". Dockerfile 1 prompt, "Verification and deployment".
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
- **Evidence:** FR-6.3–6.5; ARCHITECTURE "Account lifecycle". Fastify 5.12 prompt, "Browser state and OAuth".
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
- **Status:** BLOCKED
- **Dependency:** The XML parser library for 13F information tables is not chosen, and no evaluated pack covers it (OD-018).
- **Evidence:** FR-2.1–2.3; TB-2. ES2026 prompt, "Structured data and queries" (bounded parsing).

## Open Security Decisions

**Resolved on 2026-10-30 and recorded in REQUIREMENTS, DESIGN, or ARCHITECTURE:** OD-001 (technology recorded), OD-002 (TypeScript 7), OD-003 (newest React 19.x), OD-004 (Fastify TLS, separate containers, public cert plus private CA, Azure VM), OD-005 (envelope encryption design), OD-006 (session and CSRF), OD-007 (rate limits), OD-008 (size limits), OD-009 (account lifecycle), OD-010 (SHA-256 checked by the worker), OD-011 (secrets in Key Vault), OD-012 (Python 3.13 pack), OD-013 (Zod validator compiler).

**OD-014 — Azure SDK coverage**
- **Gap:** The Azure Key Vault and managed-identity client libraries and versions are not named, and no evaluated pack covers them.
- **Resolution evidence:** Library and version in ARCHITECTURE, plus a matching pack if the corpus publishes one.
- **Affected:** SC-015, SC-028.
- **Owner:** UNKNOWN

**OD-015 — CI system**
- **Gap:** The CI system that runs image scans, grep checks, and type checks is UNKNOWN.
- **Resolution evidence:** CI named in ARCHITECTURE.
- **Affected:** SC-013, SC-034, SC-035.
- **Owner:** UNKNOWN

**OD-016 — Timeout and message-size values**
- **Gap:** These values are UNKNOWN: Fastify `requestTimeout`, `connectionTimeout`, `handlerTimeout`; provider fetch deadlines; the worker message size cap. Each rule requires a nonzero value.
- **Resolution evidence:** Documented values.
- **Affected:** SC-017, SC-020, SC-029.
- **Owner:** UNKNOWN

**OD-017 — Owner review of delegated decisions**
- **Gap:** The ARCHITECTURE items marked (DELEGATED) were chosen by the assistant at the owner's request: rate limits, size limits, AES-256-GCM, yearly rotation, the password minimum, the CSRF and session plugins, the worker protocol, the renewal challenge, and SHA-256.
- **Resolution evidence:** The owner confirms or changes each value, and the (DELEGATED) markers are removed.
- **Affected:** SC-002, SC-005, SC-006, SC-010, SC-015, SC-020, SC-021, SC-030, SC-036.
- **Owner:** UNKNOWN

**OD-018 — 13F XML parser**
- **Gap:** The Node XML parser for 13F information tables, and its entity, depth, and size settings, are not chosen.
- **Resolution evidence:** Parser and version in ARCHITECTURE, plus a pack if available.
- **Affected:** SC-038.
- **Owner:** UNKNOWN

**OD-019 — Model file format and ML library**
- **Gap:** The gradient-boosting library and the model serialization format are not named beyond "non-executable, no pickle".
- **Resolution evidence:** Library and format in ARCHITECTURE.
- **Affected:** SC-021, SC-037.
- **Candidate packs:** none found for tabular ML libraries.
- **Owner:** UNKNOWN

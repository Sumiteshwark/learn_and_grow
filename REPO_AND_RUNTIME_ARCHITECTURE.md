**Last Updated:** March 2026 
**Author:** SUMITESHWAR KUMAR

---

# 🏗️ Repo Layout, Runtime Topology & Language — A Mental Model

Three terms people constantly conflate: **monorepo/polyrepo**, **monolith/microservices**, and **single-language/polyglot**. They answer three *different* questions and don't constrain each other. This guide separates them, then maps Covlant onto all three.

## Table of Contents

1. [Three independent axes](#1-three-independent-axes) — the one idea this whole doc rests on
2. [Building blocks: repo, package, project](#2-building-blocks-repo-package-project) — the core vocabulary
3. [Package roles: library, deployable, app, service](#3-package-roles-library-deployable-app-service) — how packages relate
4. [Source layout (Axis 1)](#4-source-layout-axis-1) — single-project / monorepo / polyrepo
5. [Runtime topology (Axis 2)](#5-runtime-topology-axis-2) — monolith / microservices
6. [Language (Axis 3)](#6-language-axis-3) — single-language / polyglot
7. [Covlant, fully mapped](#7-covlant-fully-mapped) — the worked example: tree, inventory, request flow, rationale
8. [Cheat sheet](#8-cheat-sheet) — condensed reference

---

## 1. Three independent axes

You can describe any codebase along three separate axes. **Picking a value on one axis does not pick a value on the others.**

| Axis | Question it answers | Choices | Section |
|---|---|---|---|
| **Source layout** | How is the *code* organized? | Single-project · Monorepo · Polyrepo | [§4](#4-source-layout-axis-1) |
| **Runtime topology** | How does the *running system* look? | Monolith · Microservices | [§5](#5-runtime-topology-axis-2) |
| **Language** | How many *primary languages*? | Single-language · Polyglot | [§6](#6-language-axis-3) |

A monorepo can ship a monolith *or* microservices. A microservices system can live in one repo *or* many. Any repo shape can be one language *or* several. Covlant ([§7](#7-covlant-fully-mapped)) deliberately mixes all three.

---

## 2. Building blocks: repo, package, project

Most confusion downstream comes from blurring these three words.

| | Repo | Package | Project |
|---|---|---|---|
| **It is a unit of** | version control | build & dependency | nothing formal — a human label |
| **Defined by** | a `.git/` folder | a manifest (`package.json`, `Cargo.toml`, `go.mod`, …) | whatever the speaker means |
| **Identity** | a git URL | a name + version | a goal / scope / buildable thing |
| **Tracks** | commits, branches, PRs | deps, scripts, build output, exported API | — |

One-line model: **Repo = "what git sees." Package = "what the build tool sees." Project = colloquial** (a product, a buildable unit, or whatever folder your IDE opened — always ask what someone means if scope matters).

### A repo contains 0, 1, or many packages

| Packages in repo | Name | Example |
|---|---|---|
| 0 (just docs/config) | docs/config repo | — |
| 1 | **single-project repo** | `orchestrator/`, `codeparser/` |
| 2+ | **monorepo** | `sentinel/` (24 packages) |

A package lives *inside* a repo; a repo *contains* packages. They are not the same level of thing — and the package count is exactly what distinguishes the source-layout shapes in [§4](#4-source-layout-axis-1).

---

## 3. Package roles: library, deployable, app, service

"Package" is the structural noun ([§2](#2-building-blocks-repo-package-project)). **Library**, **deployable**, **app**, and **service** are *roles* a package plays — distinguished by how it's used, not by anything structural. All have a manifest; all install the same way.

```
                         package  (has a manifest)
                        /                          \
              LIBRARY                            DEPLOYABLE
        (imported; never runs)              (runs as a process / ships an artifact)
                                              /                      \
                                          APP                     SERVICE
                                   (frontend deployable)    (backend deployable)
```

| Role | Runs? | Used by | Covlant example |
|---|---|---|---|
| **Library** | no | other packages, via `import` | `@sentinel/core/common` |
| **Deployable** | yes | end users / network | (umbrella for app + service) |
| **App** | yes (in a browser) | the user | `@sentinel/apps/web` |
| **Service** | yes (server process) | other services / the app | `@sentinel/services/api` |

So: **all apps and services are deployables; a library is the opposite of a deployable.** A package can technically be both (exports *and* boots a process) but that's rare.

### The import-direction rule

Roles imply who may depend on whom. In a monorepo this is enforced by convention:

```
   apps/*  ─────┐
   services/* ──┼──►  core/*  (libraries)      imports always point DOWN
                │
   library  ──► other library      OK (with care)
   service  ──► other service      FORBIDDEN — peers, no coupling
   library  ──► deployable         FORBIDDEN — no upward deps
```

Why services may not import each other:

- **Independent deployment** — coupling collapses the independence that makes them separate services.
- **Peers, not a hierarchy** — mutual imports create build-graph cycles.
- **Replaceability** — one service may be rewritten in another language tomorrow.

**Consequence:** shared code between services must move *down* into a `core/*` library. (Real Covlant case: `recordUsage` started in `services/api`; once `executor` and `healer` needed it too, it was lifted into `@sentinel/core/usage-metering` — the "core if ≥ 2 consumers" rule. The same pattern recurs with `OrchestratorClient` in [§7](#7-covlant-fully-mapped).)

---

## 4. Source layout (Axis 1)

How is the code organized across git repos and packages?

| Shape | Repos | Packages | Use when |
|---|---|---|---|
| **Single-project repo** | 1 | 1 | Default. Most apps, CLIs, libraries. |
| **Monorepo** | 1 | many | ≥ 2 packages need to share code atomically. |
| **Polyrepo** | many | many (≈ 1/repo) | Separate teams, languages, or release cadences. |

Note the often-skipped third case: a repo with **one** package is neither mono nor poly — it's just a single-project repo. "Monorepo" specifically means *many packages deliberately co-located in one repo*.

### What makes a monorepo work

- **Workspace manager** (pnpm/Yarn/Cargo/Go workspaces) — when package A imports package B, it links to the **local folder on disk**, not a published version. No publish step, no internal version bumps.
- **Build orchestrator** (Turborepo, Nx, Bazel) — runs `build`/`typecheck`/`lint` across packages in dependency order, with caching. `turbo … --filter=<pkg>` = "this package plus everything it depends on."

### Why monorepo vs polyrepo

**Pro (monorepo):** a change spanning a shared library and N consumers is **one atomic commit / one PR / one CI run**.
> e.g. a metering change in a `core/*` library + call sites in 3 services → monorepo: 1 PR, CI checks the whole graph. Polyrepo: 4 PRs, a version bump, a release dance, consumers drifting out of sync.

**Con (monorepo) / why split into a polyrepo:**
- Packages with no shared symbols (e.g. across languages) gain little from co-location.
- Separate teams, release cadences, and CI configs are cleaner apart.
- When two components only talk over a wire contract (gRPC `.proto`, HTTP spec), that boundary exists either way — so co-location buys nothing.

### How layouts evolve

```
single-project   →   monorepo        →   polyrepo (or polyrepo of monorepos)
1 repo / 1 pkg        1 repo / N pkgs     N repos
   (start)         (need to share)      (separate teams / languages)
```

Most codebases start single-project and grow rightward as sharing needs and team boundaries appear. Going *backward* (collapsing a polyrepo, or breaking a monolith into services) is also common when coordination cost dominates.

---

## 5. Runtime topology (Axis 2)

At runtime, how many processes run, and how do they talk?

| Shape | Processes | Internal comms | Trade-off |
|---|---|---|---|
| **Monolith** | 1 | function calls | Simple to deploy/debug; scales as a unit; one crash takes everything. |
| **Microservices** | many | network (HTTP/gRPC/queues) | Independent scaling/deploys, fault isolation, per-service languages; but the network and distributed-systems problems (retries, timeouts, idempotency, tracing) become yours. |

("Modular monolith" and "service-oriented architecture" sit between the extremes.)

### A real-life example: an online store

Picture an e-commerce app with four concerns — **Catalog, Cart, Payments, Shipping** — built each way:

```
MONOLITH                              MICROSERVICES
one process (store-server)            four processes, one per concern
┌────────────────────────┐           ┌─────────┐  ┌─────────┐
│ Catalog                │           │ Catalog │  │  Cart   │
│ Cart      (in-memory   │           └────┬────┘  └────┬────┘
│ Payments   function    │                │  network   │
│ Shipping   calls)      │           ┌────┴────┐  ┌────┴────┐
└────────────────────────┘           │Payments │  │Shipping │
                                     └─────────┘  └─────────┘
```

What the difference *means* in practice:

| Situation | Monolith | Microservices |
|---|---|---|
| **Black Friday — Cart is hammered** | scale the *whole* app (Catalog, Payments, Shipping too) just to handle Cart load | scale only the **Cart** process — pay for what you need |
| **Payments has a memory leak and crashes** | the whole store goes down — no browsing, no checkout | only **Payments** is down; customers can still browse Catalog and fill Carts |
| **Ship a one-line fix to Shipping** | rebuild + redeploy the entire app | redeploy only **Shipping**, others untouched |
| **Team writes Payments in a different language** | can't — one codebase, one runtime | **Payments** can be its own language/stack |
| **A checkout reads the cart** | a function call, instant, in-process | a network call — needs timeouts, retries, can fail |

That last row is the catch: microservices buy independence by turning function calls into network calls, and the network is where latency and failure live. Most products are a **monolith first** (simpler, faster to build) and split out services only when scale, fault-isolation, or team boundaries force it — Amazon, Netflix, and Twitter all famously started as monoliths and broke them up as they grew.

**The scope twist (important):** the *system* on the right is microservices — but each box, **in isolation, is its own monolith.** The Payments process is one process running all its payment endpoints via in-memory function calls; zoom into it and it looks exactly like the left side. "Microservice" is the role a box plays *in the system*; "monolith" is what that same box is *on its own*. Same box, two labels, two scopes — which is precisely the next point.

### Scope matters — this label is system-level, not process-level

Generalize what the store just showed: **monolith/microservices is a property of the *whole system*, not of any single process.** So how do you actually classify one?

**5-second test:** when the app is up, how many separate processes (`docker ps`) collaborate to serve users? **1 → monolith. 2+ talking to each other → microservices.**

- A service with many endpoints (login, register, profile…) in one process is *internally* a monolith — but if it's one of several collaborating processes, the **system** is microservices.
- "One endpoint per service" is **not** the goal — that's the *nano-services* anti-pattern. A service owns a **bounded responsibility** that can hold many endpoints. The split is between *processes*, not *endpoints*.

### Does a single-project repo always mean monolith?

Almost — with one nuance and one edge case:

- **Shape:** 1 manifest → 1 buildable thing → typically 1 process → monolithic *in shape*.
- **Scope nuance:** that one process can be *one node* of a larger microservices system (e.g. `orchestrator/` and `codeparser/` are each single-project repos, yet each is one of Covlant's microservices).
- **Edge case:** a single manifest *can* emit multiple binaries (Cargo `[[bin]]`, Go `cmd/foo`, `cmd/bar`, npm `bin` entries). Deploy those as separate collaborating processes and you get microservices *from a single-project repo* — unusual but legal.

> Cleanest formulation: a single-project repo contributes **one process** to the runtime. Whether the *system* is a monolith depends on what else runs alongside it.

---

## 6. Language (Axis 3)

Repo shape says nothing about language count. Any shape can be single-language or **polyglot** (multiple *primary application* languages).

First, a distinction that kills most confusion: **auxiliary file types ≠ polyglot.** Every repo has SQL, YAML, Dockerfiles, shell, `.proto`. Those don't count. The question is whether *application logic* is written in more than one language.

### Polyglot single-project repo — rare

A single manifest implies one primary language. True multi-language-in-one-package happens only via FFI / native extensions / codegen:

- Python + Cython/C extension (numpy, pandas) · Node + C++ via node-gyp (`better-sqlite3`, `sharp`) · Rust compiling C via `build.rs` + `cc` · Python + Rust via PyO3/maturin.

…and even these usually hide a *secondary manifest* (a `Cargo.toml` inside a maturin-built Python package) — at which point it's really a 2-package monorepo in disguise. (Covlant's `orchestrator/` is **not** polyglot — it's Rust + auxiliary `.proto`/SQL/YAML, i.e. "multi-file-type." See [§7](#7-covlant-fully-mapped).)

### Polyglot monorepo — where polyglot lives cleanly

A monorepo is just "many packages in one repo," and packages needn't share a language — each owns its own manifest and toolchain. This is the norm at scale (Google: C++/Java/Python/Go/JS; Meta: Hack/Python/C++/Rust/JS; Nx shops: TS/Python/Go).

What changes vs single-language:

- **Workspace tools are single-language** — pnpm reads `package.json`, Cargo reads `Cargo.toml`, Go reads `go.mod`. They *coexist* (each ignores the others' folders), but no single ecosystem tool sees everything.
- **One graph across languages** needs a polyglot build system — **Bazel / Nx / Pants / Buck2**.
- **No cross-language types** — bridge via generated code from a shared schema (Protobuf, OpenAPI, Prisma).

### Should you go polyglot? (it's never "can you")

| Question | Answer |
|---|---|
| Single-project, multiple languages? | Strictly rare — needs FFI/codegen; often becomes a 2-package monorepo. |
| Monorepo, multiple languages? | Yes, common at scale. This is where polyglot lives cleanly. |
| What restricts language choice? | Not repo shape — only tooling appetite (maintaining N toolchains, CI matrix, wire contracts, context-switching cost). |

---

## 7. Covlant, fully mapped

All three axes at once:

| Axis | Covlant's value |
|---|---|
| Source layout | **Polyrepo** of 3 git repos, one of which (`sentinel/`) is a **monorepo** |
| Runtime topology | **Microservices** — 7 collaborating processes |
| Language | **Polyglot** at the org level (TS + Rust + Go); `sentinel/` itself is polyglot (TS + Go) |

### Source layout

```
Covlant GitHub org                                ← POLYREPO (org level)
│
├── sentinel/      (TS + Go) ── POLYGLOT MONOREPO ── pnpm workspaces + Turborepo
│   └── packages/@sentinel/
│       ├── core/*                              16 libraries
│       ├── services/api                        backend (TS, Fastify HTTP)
│       ├── services/executor                   backend (TS, BullMQ worker, Playwright)
│       ├── services/healer                     backend (TS, BullMQ worker)
│       ├── services/recorder-webrtc-sidecar    backend (Go, WebRTC via Pion)
│       ├── apps/web                            frontend (React SPA)
│       ├── integrations                        library (third-party glue)
│       └── config/{eslint,typescript}          dev-tooling libraries
│
├── orchestrator/  (Rust)    ── SINGLE-PROJECT REPO ── 1 Cargo crate → gRPC service
│
└── codeparser/    (Go)      ── SINGLE-PROJECT REPO ── 1 Go module  → gRPC service ("Graph Service")
```

`orchestrator/` and `codeparser/` are **single-project repos, not monorepos** — being siblings of a monorepo doesn't make them one.

### Where each shape lands (Axis 1 × Axis 2 matrix)

```
             SINGLE-PROJECT        MONOREPO              POLYREPO
             (1 repo, 1 pkg)       (1 repo, N pkgs)      (N repos)
        ┌───────────────────────────────────────────────────────────────────────┐
MONO-   │  most CRUD apps,     │  Rails app w/        │  rare / painful         │
LITH    │  most CLIs           │  engines             │  (cross-repo build)     │
        ├───────────────────────────────────────────────────────────────────────┤
MICRO-  │  orchestrator/,      │  sentinel/ (4 svcs   │  Covlant org as a whole │
SVCS    │  codeparser/         │  + web + libs)       │  (sentinel+orch+parser) │
        └───────────────────────────────────────────────────────────────────────┘
```

### Runtime topology — 7 processes

```
   USER ──HTTPS──► apps/web (SPA) ──HTTPS──► services/api [FRONTEND ↑ / BACKEND ↓]
                                                  │
                          ┌───────────────────────┼────────────────────────┐
                          │ BullMQ/Redis          │            BullMQ/Redis│
                          ▼                       │                        ▼
                  services/executor          (Postgres,              services/healer
                          │                   Redis,                       │
                          │ gRPC              FalkorDB shared)             │ gRPC
                          └───────────────────────┬────────────────────────┘
                                                  ▼
                                          orchestrator (Rust)
                                                  │ gRPC
                                                  ▼
                                          codeparser (Go) ──► FalkorDB

(recorder-webrtc-sidecar runs alongside executor for WebRTC capture)
```

Each box is independently deployable, with its own lifecycle, scaling, and resource profile.

### Package inventory (24 packages → 5 processes)

| Group | Package | Role | Notes |
|---|---|---|---|
| `apps/` | `web` | frontend deployable | React SPA |
| `services/` | `api` | service (TS) | Fastify HTTP server |
| `services/` | `executor` | service (TS) | BullMQ worker — drives Playwright |
| `services/` | `healer` | service (TS) | BullMQ worker — fixes failing tests |
| `services/` | `recorder-webrtc-sidecar` | service (**Go**) | WebRTC capture, uses Pion |
| `core/` | `ai-orchestrator` | library | gRPC client to orchestrator |
| `core/` | `api-test-executor` | library | API-test execution helpers |
| `core/` | `auth` | library | Auth0 / RBAC / sessions |
| `core/` | `browser-engine` | library | Playwright wrapper, locator strategy |
| `core/` | `common` | library | logger, branded IDs, shared types |
| `core/` | `database` | library | Prisma client, query helpers |
| `core/` | `e2e-test-executor` | library | E2E-test execution helpers |
| `core/` | `enterprise` | library | enterprise-tier features |
| `core/` | `feature-flags` | library | feature-flag wrapper |
| `core/` | `graph-service` | library | gRPC client to codeparser |
| `core/` | `limits` | library | quota / rate-limit helpers |
| `core/` | `recording` | library | recording domain logic |
| `core/` | `replay` | library | recording replay logic |
| `core/` | `telemetry` | library | OpenTelemetry / metrics |
| `core/` | `test-generation` | library | test-generation orchestration |
| `core/` | `usage-metering` | library | `recordUsage`, `extractProvider` |
| `integrations/` | `integrations` | library | third-party glue (flat package) |
| `config/` | `eslint` | dev-tooling library | shared ESLint config |
| `config/` | `typescript` | dev-tooling library | shared `tsconfig` base |

| Category | Count |
|---|---|
| Deployables — apps | 1 (`web`) |
| Deployables — services | 4 (`api`, `executor`, `healer`, `recorder-webrtc-sidecar`) |
| Libraries — core | 16 |
| Libraries — integrations | 1 |
| Libraries — config | 2 |
| **Total packages** | **24** |

**Package ≠ process** ([§2](#2-building-blocks-repo-package-project) meets [§5](#5-runtime-topology-axis-2)): 24 packages produce only 5 processes (1 app + 4 services). The 19 libraries/config packages run nowhere — they're imported. And the mapping isn't 1:1 either direction: one package can run as *many* processes (a scaled `services/api`), and one process pulls in *many* packages at build time.

### Request walkthrough — "run tests" (all concepts at once)

1. **`apps/web`** (frontend) fires an HTTPS POST to the API.
2. **`services/api`** authenticates, writes a `TestRun` to Postgres, enqueues a BullMQ job. To do so it `import`s `@sentinel/core/common` + `@sentinel/core/limits` — resolved to local folders by pnpm, no publish step. **← the monorepo win.**
3. **`services/executor`** picks up the job, drives Playwright, and calls **`orchestrator/`** over gRPC for LLM-driven page understanding. **← a polyrepo boundary:** different repo, different language, contract is the `.proto` *duplicated* into both repos.
4. **`orchestrator`** runs its workflow, then calls **`codeparser/`** over gRPC for the code graph. **← another polyrepo hop** (Go, FalkorDB-backed).
5. On failure, **`services/healer`** picks up a heal job and calls `orchestrator` over the same gRPC contract — a different consumer, same monorepo.

What it illustrates:
- **`OrchestratorClient` lives in `@sentinel/core/ai-orchestrator`** because 3 services need it → a shared library, one atomic change ([§3](#3-package-roles-library-deployable-app-service)).
- **The `.proto` is duplicated** across the polyrepo boundary — no workspace links the repos, so a contract change is a coordinated 2-PR release.
- **`codeparser` is also called "Graph Service"** — same Go process, two callers (Sentinel + Orchestrator).

### Design rationale

- **Why microservices?** Rust (orchestrator) and Go (codeparser) want different ecosystems than TS; healer crashing shouldn't take down the API; each service has a different resource profile (executor = CPU + browser RAM; api = I/O-bound; orchestrator = outbound LLM HTTP).
- **Why a monorepo inside Sentinel?** Its TS services share types, logger, DB helpers, RBAC, and the orchestrator client. A schema change touches `core/*` + N services in one PR. Polyrepo here would force internal npm publishing or vendoring.
- **Why a polyrepo across the org?** TS/Rust/Go share no symbols; the only boundary is the `.proto`, which you pay either way. Separate repos give clean per-language toolchains, CI, and cadences.
- **Why single-project for orchestrator & codeparser?** Neither has a second internal package — adding workspace tooling now is cost for no benefit.
- **Why is the Go `recorder-webrtc-sidecar` *inside* Sentinel** (not split out like codeparser)? Server-side WebRTC's only production-grade non-C++ stack is **Pion** (Go), so the language choice is forced. But it's a *sidecar* to `executor` — shared recording schema, shipped/versioned together — so a polyrepo seam isn't justified. **Rule of thumb: tight coupling → same repo (regardless of language); independent evolution → its own repo.**

---

## 8. Cheat sheet

### Vocabulary (one line each)

| Term | Meaning |
|---|---|
| **Repo** | Unit of version control — a `.git/` folder. |
| **Package** | Unit of build & dependency — a manifest (`package.json`/`Cargo.toml`/`go.mod`). |
| **Project** | Loose human label — a goal/scope/buildable thing. No formal boundary. |
| **Library** | A package that gets **imported**; never runs alone. |
| **Deployable** | A package that **runs** as a process / ships an artifact. |
| **App** | A *frontend* deployable (`apps/*`, runs in a browser). |
| **Service** | A *backend* deployable (`services/*`, a server process). |
| **Process** | A running instance of a deployable. One package → many processes possible. |
| **Polyglot** | Repo using multiple *primary* languages. Lives cleanly only in a **monorepo**; auxiliary files (SQL/YAML/Dockerfile) don't count. |

### Five identity rules ("X ≠ Y")

- **Repo ≠ Package** — git's unit vs the build tool's unit; a repo holds 0/1/N packages.
- **Package ≠ Process** — libraries never run; a scaled service is one package as many processes.
- **App ⊂ Deployable, Service ⊂ Deployable** — both are *kinds* of deployable; `apps/web` is a deployable but not a service.
- **Language ≠ Repo shape** — any shape can be polyglot (Sentinel: TS + Go).
- **Source layout ≠ Runtime topology** — "how it's organized" vs "how it runs" are independent axes.

### The three axes at a glance

| Axis | Choices | Test |
|---|---|---|
| Source layout | single-project / monorepo / polyrepo | how many repos? how many packages each? |
| Runtime topology | monolith / microservices | how many collaborating processes (`docker ps`)? |
| Language | single / polyglot | how many *primary* languages (ignore SQL/YAML/etc.)? |

> **Scope reminder:** runtime topology is a *system* label. Each process inside a microservices system is internally monolithic. A service with 50 endpoints in one process is still one process.

### Covlant's shape

| Layer | Value | Detail |
|---|---|---|
| Runtime | Microservices | 7 collaborating processes |
| Org source | Polyrepo | `sentinel/` + `orchestrator/` + `codeparser/` |
| `sentinel/` | Polyglot monorepo (TS+Go) | 24 packages: 1 app + 4 services + 16 core + 1 integrations + 2 config |
| `orchestrator/` | Single-project repo (Rust) | 1 crate → gRPC service |
| `codeparser/` | Single-project repo (Go) | 1 module → gRPC service ("Graph Service") |

### Where boundaries hurt

- **Across the polyrepo boundary** (Sentinel ↔ Orchestrator ↔ Codeparser) → duplicated `.proto`, coordinated PRs, no shared types.
- **Across the monorepo boundary** (between Sentinel packages) → nothing; pnpm links the folder.
- **Inside a single-project repo** → no boundary; one buildable unit.
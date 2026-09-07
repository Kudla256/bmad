---
title: 'Story 1.1 — Project Foundation & Data Layer'
type: 'feature'
created: '2026-09-07'
status: 'ready-for-dev'
route: 'dispatch'
review_loop_iteration: 0
context:
  - '{project-root}/_bmad-output/implementation-artifacts/epic-1-context.md'
  - '{project-root}/_bmad-output/planning-artifacts/architecture/architecture-Securer-2026-02-13/ARCHITECTURE-SPINE.md'
  - '{project-root}/_bmad-output/planning-artifacts/ux-designs/ux-Securer-2026-02-13/DESIGN.md'
---

<frozen-after-approval reason="human-owned intent — do not modify unless human renegotiates">

## Intent

**Problem:** The repository holds only planning documents — there is no code. Every story in all four epics is blocked on a solution that builds, an orchestrated Cosmos DB, a persistence model for the one entity this product has, and a styling pipeline that can express the design tokens.

**Approach:** Scaffold the Aspire starter as `Securer/`, reshape it to the spine's layout, strip the template's demo content, add the `SecretDocument` + `SecretsDbContext` layer over a single Cosmos container, and wire Tailwind v4 with self-hosted variable fonts. No product behaviour — no endpoints, services, crypto, or UI.

## Boundaries & Constraints

**Always:**
- One-way dependencies (AD-13): AppHost → Web/ApiService, Web → ApiService, both → ServiceDefaults. `Securer.Tests` is the only test project (AD-22).
- One container `secrets`, one document type, partition key `/id`, point-read by id only (AD-10).
- `ExpiresAt` is the authoritative liveness field checked by the request path; Cosmos TTL is a second mechanism, not the only one (AD-8, AD-9).
- Web holds no Cosmos reference, connection string, or `DbContext` (AD-12); it reaches the API only via Aspire service discovery (AD-14).
- Blazor InteractiveServer only (AD-21). Tailwind utilities only, no custom CSS vocabulary. `ILogger<T>`, never `Console.WriteLine` (AD-16).

**Never:**
- No endpoints, `CryptoService`, `SecretService`, middleware, or UI — stories 1.2–1.4 own those.
- No third-party crypto package (AD-2). No Bootstrap or component library. No JS (AD-15). No sweep job for expiry (AD-8). No auth or session state (AD-18).
- No `azure.yaml`, pipeline, or Dockerfile — Epic 4 owns deployment.

**Recorded decisions** (settled at planning, do not re-litigate):
- **Attempt budget: 3.** Settles PRD open question 1 and the spine's deferred item. Configuration; consumed by 1.2.
- **Maximum secret size: 25 KB.** Settles PRD open question 2. Configuration; bounds request body and encryption buffers in 1.2.
- **`RunAsPreviewEmulator()`, not `RunAsEmulator()`.** The `stable` tag behind `RunAsEmulator` publishes a linux/amd64 manifest only, and the dev machine is arm64 (Apple M1 Max); `vnext-latest` has a native arm64 manifest. Costs one `ASPIRECOSMOSDB001` suppression and is gateway-mode only. A knowing deviation from the spine's deployment diagram.

## I/O & Edge-Case Matrix

| Scenario | Input / State | Expected Output / Behavior | Error Handling |
|---|---|---|---|
| Provisioning | AppHost run on a fresh emulator | Database + container `secrets` exist, partition key `/id`, per-item TTL enabled | N/A |
| Point-read, present | Document written, then read by id | Same document; all seven fields round-trip, byte arrays intact | N/A |
| Point-read, absent | Id never written | `null` — absence is a normal result | No throw-on-missing |
| TTL derivation | `ExpiresAt` = now + 1h at write time | Persisted `ttl` ≈ 3600 relative seconds, alongside absolute `ExpiresAt` | Never write a zero or negative `ttl` |
| Datastore down | AppHost started with Docker stopped | ApiService health check reports unhealthy; logged without secret material | No crash loop, no secret-bearing log line |

</frozen-after-approval>

## Code Map

Greenfield — nothing to reuse. Below is what `dotnet new aspire-starter` emits (verified against `Aspire.ProjectTemplates` 13.5.3) and what must change.

- `Securer/Securer.AppHost/AppHost.cs` -- named `AppHost.cs`, **not** `Program.cs`; classic `DistributedApplication.CreateBuilder`. Add the Cosmos resource; keep the existing `apiservice`/`webfrontend` wiring, health checks and `WaitFor`.
- `Securer/Securer.AppHost/Securer.AppHost.csproj` -- `Aspire.AppHost.Sdk/13.5.3`, zero PackageReferences; the Cosmos hosting package is the first added.
- `Securer/Securer.ServiceDefaults/Extensions.cs` -- already provides OpenTelemetry, health checks, service discovery, resilience. **Do not rewrite.** `MapDefaultEndpoints` maps `/health` in Development only.
- `Securer/Securer.ApiService/Program.cs` -- has a `/weatherforecast` demo. Strip it; keep `AddServiceDefaults()` + `MapDefaultEndpoints()`, add the DbContext registration.
- `Securer/Securer.Web/Program.cs` -- render mode is already correct; the service-discovery `HttpClient` (`https+http://apiservice`) is the pattern to keep — delete only its `WeatherApiClient`.
- `Securer/Securer.Web/` demo content to delete -- `WeatherApiClient.cs`, `Components/Pages/{Counter,Weather}.razor`, `Components/Layout/NavMenu.*`, `wwwroot/lib/bootstrap/**` (44 files), `wwwroot/app.css`. `App.razor` links `bootstrap.min.css` and must be rewired.

## Tasks & Acceptance

**Execution:**
- [ ] `Securer/` -- `dotnet new aspire-starter --output Securer --test-framework xUnit.net --xunit-version v3` (templates installed globally at 13.5.3) -- yields the five projects the spine requires. Fall back to default `v2` and record it if v3 does not build.
- [ ] `.gitignore` -- `dotnet new gitignore` at repo root, then append `node_modules/` and the generated `wwwroot/css/app.css` -- none exists today; `bin/obj` would be committed.
- [ ] `Securer/Securer.ApiService/Models/SecretDocument.cs` -- the seven fields `Id`, `Ciphertext`, `Salt`, `IV`, `ExpiresAt`, `RemainingAttempts`, `CreatedAt` -- the product's single entity (AD-10, AD-11).
- [ ] `Securer/Securer.ApiService/Data/SecretsDbContext.cs` -- container `secrets`, partition key `/id`, per-item TTL enabled -- the only persistence surface. Verify the real EF Core 10 Cosmos TTL API; do not assume a method name.
- [ ] `Securer/Securer.ApiService/Program.cs` -- drop the weather demo; register `SecretsDbContext` via the Aspire EF Cosmos client integration -- DI only (AD-19).
- [ ] `Securer/Securer.AppHost/AppHost.cs` -- `AddAzureCosmosDB("cosmos").RunAsPreviewEmulator(...)`, a database, and the `secrets` container on `/id`; reference from `apiservice` and `WaitFor` it -- AC 2.
- [ ] `Securer/Securer.ApiService/appsettings.json` -- attempt budget (3) and max secret size (25 KB) as environment-overridable configuration -- FR-24; consumed by 1.2.
- [ ] `Securer/Securer.Web/` -- delete the demo content listed in the Code Map; create the spine's empty `Components/Shared/`, `Services/`, `Interop/`, `Styles/`.
- [ ] `Securer/Securer.Web/Styles/tailwind.css` + `package.json` -- Tailwind v4 (`@tailwindcss/cli` 4.3.3) with DESIGN.md's colour, type, radius and spacing tokens in `@theme`; self-host fonts via `@fontsource-variable/inter` and `@fontsource-variable/jetbrains-mono` (5.3.0) -- AC 3.
- [ ] `Securer/Securer.Web/Securer.Web.csproj` -- MSBuild target running the Tailwind CLI before build into `wwwroot/css/app.css`; rewire `App.razor` to it -- AC 3's build-target requirement.
- [ ] `Securer/Securer.Tests/Data/SecretsDbContextTests.cs` -- cover the I/O matrix rows: provisioning, present/absent point-read, TTL derivation -- mirrors source structure (AD-22).

**Acceptance Criteria:**
- Given the generated solution, when `dotnet build` runs, then it succeeds with no warnings and produces all five assemblies.
- Given Docker is available, when the AppHost runs, then the dashboard shows Web, ApiService and the Cosmos emulator all healthy.
- Given the solution, when the dependency graph is inspected, then no arrow exists outside AD-13 — in particular Web references neither EF Core nor any Cosmos package.
- Given the served page, when its assets are inspected, then no Bootstrap asset is requested and both fonts load from the app's own origin, never a third-party CDN.

## Implementation Notes

## Spec Change Log

## Review Triage Log

## Design Notes

**Why fonts are self-hosted.** DESIGN.md asks only for variable fonts with `font-display: swap`. A Google Fonts CDN would send every sender's and recipient's IP to a third party on a product whose claim is that nothing correlates a request to a person (AD-18), and would break the self-hosted instance FR-23 promises.

**Why npm is acceptable.** AD-15 forbids an npm *runtime* dependency. Tailwind and Fontsource are build-time devDependencies producing a static stylesheet; no JS ships. Forward dependency: Epic 4's container build stage will need node.

**Cosmos TTL is not a serialised datetime.** Cosmos expires items on a relative `ttl` in seconds, so `ExpiresAt` cannot map onto it directly. The container needs TTL enabled-but-not-defaulted and each document a derived `ttl`. Both fields must exist — `ExpiresAt` is what AD-9's request path checks, because TTL deletion is only eventually consistent.

**Scope the suppression.** `ASPIRECOSMOSDB001` goes on the AppHost's Cosmos lines with a comment naming the arm64 reason — never a project-wide `NoWarn`.

## Verification

**Commands:**
- `dotnet build Securer/Securer.sln` -- expected: 0 warnings, 0 errors
- `dotnet test Securer/Securer.sln` -- expected: `SecretsDbContext` tests pass (needs the emulator)
- `npx @tailwindcss/cli -i Styles/tailwind.css -o wwwroot/css/app.css` from `Securer.Web` -- expected: `app.css` contains DESIGN.md's token values
- `grep -ri "bootstrap\|weather\|counter" Securer --include=*.cs --include=*.razor --include=*.csproj` -- expected: no matches

**Manual checks:**
- The Docker daemon is **down** on this machine. AC 2 and the DbContext tests cannot be proven until it is started — verify build, dependency direction and assets first, then run the AppHost once Docker is up and confirm the emulator reaches healthy.

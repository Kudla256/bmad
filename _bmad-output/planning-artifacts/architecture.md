---
stepsCompleted: [1, 2, 3, 4, 5, 6, 7, 8]
workflowType: 'architecture'
lastStep: 8
status: 'complete'
completedAt: '2026-02-13'
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-Securer-2026-02-12.md
  - _bmad-output/planning-artifacts/prd.md
  - _bmad-output/planning-artifacts/ux-design-specification.md
workflowType: 'architecture'
project_name: 'Securer'
user_name: 'Nick'
date: '2026-02-13'
---

# Architecture Decision Document

_This document builds collaboratively through step-by-step discovery. Sections are appended as we work through each architectural decision together._

## Project Context Analysis

### Requirements Overview

**Functional Requirements:**

24 functional requirements across 6 categories. Architecturally, they decompose into three core operations:

1. **Secret Creation (FR1-FR7):** Accept plaintext + password + options → encrypt server-side → store ciphertext → return unique link. The critical path is: receive request → encrypt in memory → persist ciphertext → discard plaintext and password → respond with link. No step can be reordered or skipped.

2. **Secret Retrieval (FR8-FR12):** Accept link + password → validate attempts → decrypt in memory → serve plaintext → destroy record. This is a destructive read — the retrieval operation itself is a delete. The confirmation step (FR10) means the backend must support a two-phase retrieval: validate password first, then serve content on confirmation.

3. **Secret Lifecycle (FR13-FR16):** Three destruction triggers — viewed (immediate), expired (scheduled), burned by failed attempts (conditional). All three must guarantee the ciphertext is irrecoverably removed. Password attempt tracking (FR16) is the only mutable field on an otherwise write-once record.

**Non-Functional Requirements:**

- **Performance:** Sub-500ms server-side for both encrypt and decrypt operations. Sub-1s page loads. Minimal payload size. .NET's built-in crypto libraries (System.Security.Cryptography) are highly optimized — these targets are easily achievable.
- **Security:** TLS-only. Zero logging of secrets, passwords, or request bodies on sensitive endpoints. URL fragments never sent to server. Failed attempts reveal nothing about secret existence.
- **Scalability:** Stateless request handling. Simple key-value lookups (O(1) by secret ID). Azure enables elastic scaling. Single Azure App Service or Container App instance handles typical usage; scales horizontally for growth.

**Scale & Complexity:**

- Primary domain: Full-stack web (.NET Aspire + Blazor + API + database)
- Complexity level: Low
- Estimated architectural components: ~6-8 (Blazor frontend, ASP.NET Core API, crypto service, database, secret cleanup background service, Aspire orchestration, Azure infrastructure)

### Technology Constraints

**Mandated Stack:**
- **.NET with Aspire** — Aspire provides service defaults, orchestration, and telemetry out of the box. Defines the AppHost + service project structure.
- **Blazor frontend** — Replaces the React + Tailwind + Shadcn/ui assumption from the UX spec. Options are Blazor Server (SignalR connection), Blazor WebAssembly (client-side), or Blazor Web App (hybrid with per-page render mode). This is an architectural decision for a later step.
- **Azure hosting** — Target deployment platform. Azure Container Apps, Azure App Service, or Azure Kubernetes Service are the primary options. Aspire has native Azure provisioning support via `azd` (Azure Developer CLI).
- **Docker** — Still relevant as the container format, but deployment target is Azure rather than a self-managed VPS.

**UX Spec Implications:**
- The UX spec's design system choice (Tailwind + Shadcn/ui + React) does not apply. Blazor component libraries (e.g., MudBlazor, Radzen, FluentUI Blazor, or custom CSS) will be needed instead.
- The visual design direction (split layout, glassmorphism, color system, typography) remains fully valid — it's the implementation tooling that changes.
- CSS approach: Tailwind CSS can still be used with Blazor, or standard CSS / CSS isolation. This is an implementation decision.

### Technical Constraints & Dependencies

- **Azure-native services available** — Azure Key Vault (for app secrets, not user secrets), Azure SQL / Cosmos DB / Table Storage for ciphertext storage, Azure Monitor for telemetry (with sensitive endpoint exclusions), Azure Container Apps for deployment.
- **.NET Aspire structure** — AppHost project orchestrates services, ServiceDefaults project provides shared configuration (telemetry, health checks, resilience). This is the project skeleton.
- **No user accounts** — No authentication system, no sessions, no user state. Every request is anonymous.
- **Server-side encryption only** — .NET's `System.Security.Cryptography` namespace provides AES-256, PBKDF2/Argon2 for key derivation. Well-established, FIPS-compliant.
- **Password-in-link via URL fragment** — Fragment (`#`) is never sent to the server by the browser. Client-side code must extract and forward it in the request body. With Blazor WASM this is straightforward; with Blazor Server it requires JS interop.
- **Solo developer** — Architecture must be simple enough for one person to build, debug, and maintain.
- **Open source** — Code must be auditable. No security through obscurity.

### Cross-Cutting Concerns Identified

- **Cryptographic operations** — Encryption/decryption via `System.Security.Cryptography`. Key derivation from passwords (PBKDF2 or Argon2id). Must be a single, well-tested service. Memory hygiene: use `Span<byte>`, clear buffers after use, avoid string allocations for sensitive data where possible.
- **Secret destruction** — Three triggers (view, expiry, brute-force burn) must all guarantee irrecoverable deletion. Single destruction path regardless of trigger.
- **Information leakage prevention** — API responses for all error states must be identical. ASP.NET middleware must suppress detailed error pages and server headers. Logging middleware must exclude sensitive endpoints. Timing attacks on password validation should be mitigated with constant-time comparison.
- **Expiration cleanup** — .NET `BackgroundService` (hosted service) for periodic purge of expired secrets. Aspire orchestrates this as part of the API project.
- **Azure deployment** — Aspire's `azd` integration for provisioning. Infrastructure-as-code via Bicep generated from Aspire's app model. CI/CD via GitHub Actions or Azure DevOps.

## Starter Template Evaluation

### Primary Technology Domain

Full-stack .NET web application targeting .NET 10 LTS with Aspire 13 for orchestration and Azure for hosting.

### Starter Options Considered

1. **`aspire-starter`** — Official Aspire Starter App (AppHost + ServiceDefaults + Blazor Web + API). Maps directly to Securer's architecture.
2. **`aspire`** — Empty Aspire (AppHost + ServiceDefaults only). More manual setup, unnecessary for this project.
3. **Custom from scratch** — Maximum control, maximum effort. Not justified for low-complexity project.

### Selected Starter: `aspire-starter`

**Rationale:** Provides the exact project structure Securer needs out of the box — a Blazor frontend, a Minimal API backend, and Aspire orchestration. Removes boilerplate decisions about project wiring, service discovery, and telemetry configuration. Well-maintained by Microsoft, aligned with .NET 10 LTS.

**Initialization Command:**

```bash
dotnet new aspire-starter --output Securer
```

### Architectural Decisions Provided by Starter

**Language & Runtime:** C# on .NET 10 LTS. Latest language features (C# 14).

**Project Structure:** Four projects — AppHost (orchestration), ServiceDefaults (shared config), Web (Blazor frontend), ApiService (Minimal API backend).

**Blazor Render Mode:** InteractiveServer — simplest model for a low-interaction app. No WASM download overhead. SignalR connection is acceptable given modest concurrency requirements. URL fragment handling via JS interop.

**Styling Solution:** Tailwind CSS v4 — integrated via Tailwind CLI with build target automation. Full visual control to implement the UX spec's custom design (glassmorphism, split layout, indigo accent system). No component library overhead.

**Build Tooling:** .NET SDK build pipeline. Tailwind CSS CLI as a pre-build step. Docker containerization via Aspire-generated Dockerfile.

**Development Experience:** Aspire Dashboard for local development (service discovery, telemetry, logs). Hot reload via `dotnet watch`. Aspire CLI for orchestration.

**Note:** Project initialization using this command should be the first implementation story.

## Core Architectural Decisions

### Decision Priority Analysis

**Critical Decisions (Block Implementation):**
- Database: Azure Cosmos DB (NoSQL API)
- Encryption: AES-256-GCM with PBKDF2 key derivation
- API: Two-endpoint Minimal API with single atomic reveal
- Hosting: Azure Container Apps

**Important Decisions (Shape Architecture):**
- Blazor InteractiveServer render mode
- Tailwind CSS v4 for styling
- EF Core Cosmos provider for data access
- Local component state only (no global state management)

**Deferred Decisions (Post-MVP):**
- Rate limiting strategy (Phase 2)
- CDN / edge caching for static assets
- Multi-region deployment

### Data Architecture

- **Database:** Azure Cosmos DB (NoSQL API) — point-read by secret ID as partition key delivers sub-10ms lookups. Native TTL on documents handles auto-expiration of unclaimed secrets without a background job.
- **Data Access:** EF Core with Cosmos DB provider — familiar .NET data access pattern, minimal boilerplate.
- **Local Development:** Cosmos DB Emulator via Aspire `.RunAsEmulator()` — runs as a Docker container, zero setup.
- **Document Model:** Single container, single document type. Fields: Id (partition key), Ciphertext, Salt, IV, ExpiresAt (maps to Cosmos TTL), RemainingAttempts, CreatedAt.

### Security & Encryption

- **Encryption:** AES-256-GCM via `System.Security.Cryptography` — authenticated encryption (confidentiality + integrity). No external dependencies.
- **Key Derivation:** PBKDF2 via `Rfc2898DeriveBytes` — derives a 256-bit AES key from the user-provided password + random salt. Built into .NET, FIPS-compliant.
- **Memory Hygiene:** Use `Span<byte>` and `CryptographicOperations.ZeroMemory()` to clear sensitive buffers after use. Avoid `string` allocations for passwords/plaintext where possible.
- **Password Attempts:** `RemainingAttempts` field on Cosmos document. Decremented on each failed attempt. Document deleted when counter hits 0.
- **Timing Attack Mitigation:** Constant-time comparison via `CryptographicOperations.FixedTimeEquals()` for password-derived key validation.
- **Information Leakage Prevention:** All error states return identical `404` response. ASP.NET middleware suppresses server headers and detailed error pages. Logging middleware excludes `/api/secrets` request bodies.

### API & Communication Patterns

- **Framework:** ASP.NET Core Minimal APIs
- **Endpoints:**
  - `POST /api/secrets` — Create secret (accept plaintext + password + options, return secret ID)
  - `POST /api/secrets/{id}/reveal` — Reveal secret (verify password → decrypt → delete → return plaintext; or decrement attempts + return error)
- **All POST** — No GET for secrets. No secret data in URLs or access logs.
- **Single Atomic Reveal** — Password verification, decryption, and deletion happen in one request. Confirmation dialog is a client-side UI gate before the API call.
- **Uniform Error Response** — Every failure state returns `404 { "message": "This secret is not available" }`. No `401`, `403`, or `409`.

### Frontend Architecture

- **Render Mode:** Blazor InteractiveServer — SignalR connection, no WASM download, simplest model for low-interaction app.
- **Routing:** Two routes via `@page` — `/` (sender form), `/s/{id}` (recipient flow).
- **State Management:** Local component state only. No global state library. Two pages with no shared state.
- **Styling:** Tailwind CSS v4 via CLI with build target automation. Custom Razor components matching UX spec design direction.
- **JS Interop:** Two use cases only — Clipboard API (`navigator.clipboard.writeText`) and URL fragment reading (`window.location.hash`) for password-in-link mode.
- **Components:** `SplitLayoutShell`, `GlassCard`, `PasswordRow`, `SecretDisplay`, `CountdownTimer`, `SuccessPanel`, `DeadEndScreen` — all as `.razor` files with Tailwind classes.

### Infrastructure & Deployment

- **Hosting:** Azure Container Apps — native Aspire integration, scale-to-zero, `azd up` deployment.
- **CI/CD:** Azure DevOps Pipelines — familiar tooling, free tier (1 parallel job, 1,800 min/month), `azd` integration for automated deployments.
- **Monitoring:** Azure Monitor via Aspire's OpenTelemetry (ServiceDefaults). Log filtering configured to exclude sensitive endpoint request bodies.
- **App Configuration:** Azure Key Vault for production secrets (connection strings). Aspire handles environment-specific configuration.
- **Local Development:** Aspire AppHost orchestrates Cosmos DB Emulator container + API + Blazor. Single `dotnet run` starts everything.

### Decision Impact Analysis

**Implementation Sequence:**
1. Scaffold project with `dotnet new aspire-starter`
2. Add Cosmos DB integration to AppHost with emulator
3. Define secret document model + EF Core Cosmos context
4. Implement crypto service (AES-256-GCM + PBKDF2)
5. Build two Minimal API endpoints
6. Build Blazor sender page with Tailwind
7. Build Blazor recipient page
8. Configure security middleware (headers, logging exclusions, error handling)
9. Azure deployment via `azd` + Azure DevOps pipeline

**Cross-Component Dependencies:**
- Crypto service is shared between both API endpoints
- Cosmos DB document model drives both API and TTL configuration
- JS interop module is shared between sender (clipboard) and recipient (URL fragment) pages
- Security middleware applies globally but with endpoint-specific filtering

## Implementation Patterns & Consistency Rules

### Naming Patterns

**Cosmos DB Naming:**
- Container name: `secrets` (lowercase plural)
- Document properties: PascalCase (`RemainingAttempts`, `ExpiresAt`) — standard C# property convention, EF Core maps naturally
- Partition key path: `/id`

**API Naming:**
- Endpoints: lowercase, plural nouns — `/api/secrets`, `/api/secrets/{id}/reveal`
- Route parameters: `{id}` (lowercase)
- JSON request/response fields: camelCase (`secretId`, `expiresAt`) — ASP.NET's default `System.Text.Json` serialization handles PascalCase → camelCase automatically

**Code Naming (C# conventions):**
- Classes/Records: PascalCase — `SecretDocument`, `CryptoService`, `CreateSecretRequest`
- Interfaces: `I` prefix — `ICryptoService`, `ISecretRepository`
- Methods: PascalCase — `CreateSecretAsync`, `RevealSecretAsync`
- Private fields: `_camelCase` — `_cryptoService`, `_dbContext`
- Local variables: camelCase — `secretId`, `encryptedBytes`
- Constants: PascalCase — `MaxPasswordAttempts`, `DefaultExpiration`
- Async methods: always suffixed with `Async`

**Blazor Component Naming:**
- Components: PascalCase `.razor` files — `GlassCard.razor`, `PasswordRow.razor`
- Component parameters: PascalCase `[Parameter]` — `OnSecretCreated`, `SecretId`
- CSS classes: Tailwind utilities only — no custom CSS class naming needed

### Structure Patterns

**Project Organization:**
```
Securer/
├── Securer.AppHost/              # Aspire orchestration
├── Securer.ServiceDefaults/      # Shared config (telemetry, resilience)
├── Securer.ApiService/           # Minimal API backend
│   ├── Endpoints/                # Endpoint definitions (SecretsEndpoints.cs)
│   ├── Services/                 # Business logic (CryptoService.cs, SecretService.cs)
│   ├── Models/                   # DTOs and documents (SecretDocument.cs, CreateSecretRequest.cs)
│   ├── Data/                     # EF Core context (SecretsDbContext.cs)
│   └── Middleware/               # Security middleware (SecurityHeadersMiddleware.cs)
├── Securer.Web/                  # Blazor frontend
│   ├── Components/
│   │   ├── Layout/               # SplitLayoutShell, GlassCard
│   │   ├── Shared/               # Reusable components (PasswordRow, CountdownTimer)
│   │   └── Pages/                # Route pages (Home.razor, RevealSecret.razor)
│   ├── Services/                 # API client service (SecretApiClient.cs)
│   ├── wwwroot/                  # Static assets, generated Tailwind CSS
│   └── Interop/                  # JS interop (clipboard.js, fragment.js)
└── Securer.Tests/                # Separate test project
    ├── ApiService/               # API tests mirroring ApiService structure
    └── Web/                      # Blazor component tests
```

**Key rules:**
- Tests in a separate project (`Securer.Tests`), NOT co-located — standard .NET convention
- One endpoint class per resource (`SecretsEndpoints.cs`) using Minimal API route groups
- One service per concern (`CryptoService`, `SecretService`) — not one mega-service
- JS interop files in `Interop/` folder, not scattered in `wwwroot`

### Format Patterns

**API Response Formats:**

Success (create):
```json
{ "secretId": "abc-123", "expiresAt": "2026-02-14T10:00:00Z" }
```

Success (reveal):
```json
{ "content": "the decrypted secret text" }
```

Error (wrong password):
```json
{ "message": "Incorrect password", "remainingAttempts": 2 }
```

Error (all failure states — expired, burned, nonexistent, already viewed):
```json
{ "message": "This secret is not available" }
```
HTTP `404` for all of these. No other status codes for secret operations.

- **Dates:** ISO 8601 strings in UTC (`2026-02-14T10:00:00Z`) — `System.Text.Json` default
- **Nulls:** Omit null fields from JSON responses (`JsonIgnoreCondition.WhenWritingNull`)
- **No response wrapper** — direct DTOs, no `{ data: ..., meta: ... }` envelope. Two endpoints don't need it.

### Process Patterns

**Error Handling:**
- API: Global exception handler middleware returns `500 { "message": "An error occurred" }` — no stack traces, no details
- Blazor: Try-catch in component event handlers, display inline error messages per UX spec
- Never throw exceptions for expected flow (wrong password is not exceptional — it's a normal code path returning a result)

**Loading States:**
- Blazor: `bool isLoading` local field per component. Button shows spinner when `true`, form disabled.
- No global loading state. Each page manages its own.

**Validation:**
- API: Validate on endpoint entry using FluentValidation or manual checks. Return `400` with field-specific errors for create endpoint only.
- Blazor: Validate on submit, not on blur. Required fields: secret text + password.

**Logging:**
- Use `ILogger<T>` everywhere — Aspire's ServiceDefaults configures OpenTelemetry sinks automatically.
- **NEVER log:** secret content, passwords, ciphertext, request bodies on `/api/secrets` endpoints.
- **DO log:** secret ID (non-sensitive), operation type, success/failure, attempt count, expiration time.

### Enforcement Guidelines

**All AI Agents MUST:**
1. Follow C# naming conventions exactly (PascalCase methods, `_camelCase` fields, `IInterface` prefix, `Async` suffix)
2. Place files in the prescribed folder structure — no new top-level folders without discussion
3. Return identical `404` responses for ALL secret failure states — no information leakage
4. Use `ILogger<T>` for logging, never `Console.WriteLine`
5. Never log sensitive data (content, passwords, ciphertext)
6. Use `async/await` for all I/O operations
7. Register services via dependency injection — no `new Service()` instantiation in endpoints or components

### Pattern Examples

**Good:**
```csharp
public async Task<SecretDocument?> GetSecretAsync(string id)
{
    return await _dbContext.Secrets.FindAsync(id);
}
```

**Anti-Pattern:**
```csharp
// Wrong: sync, no interface, logs sensitive data
public SecretDocument GetSecret(string id)
{
    Console.WriteLine($"Getting secret: {id} with content: {doc.Content}");
    return _dbContext.Secrets.Find(id);
}
```

## Project Structure & Boundaries

### Complete Project Directory Structure

```
Securer/
├── Securer.sln
├── README.md
├── .gitignore
├── .editorconfig
├── azure-pipelines.yml                    # Azure DevOps CI/CD pipeline
├── azure.yaml                             # azd deployment manifest
│
├── Securer.AppHost/                       # Aspire orchestration
│   ├── Securer.AppHost.csproj
│   ├── Program.cs                         # Service wiring: Cosmos emulator, API, Web
│   └── appsettings.json
│
├── Securer.ServiceDefaults/               # Shared Aspire configuration
│   ├── Securer.ServiceDefaults.csproj
│   └── Extensions.cs                      # OpenTelemetry, health checks, resilience
│
├── Securer.ApiService/                    # ASP.NET Core Minimal API backend
│   ├── Securer.ApiService.csproj
│   ├── Program.cs                         # App builder, middleware, endpoint registration
│   ├── appsettings.json
│   ├── Endpoints/
│   │   └── SecretsEndpoints.cs            # POST /api/secrets, POST /api/secrets/{id}/reveal
│   ├── Services/
│   │   ├── ICryptoService.cs              # Encryption/decryption interface
│   │   ├── CryptoService.cs               # AES-256-GCM + PBKDF2 implementation
│   │   ├── ISecretService.cs              # Business logic interface
│   │   └── SecretService.cs               # Create, reveal, destroy orchestration
│   ├── Models/
│   │   ├── SecretDocument.cs              # Cosmos DB document entity
│   │   ├── CreateSecretRequest.cs         # Inbound DTO for secret creation
│   │   ├── CreateSecretResponse.cs        # Outbound DTO (secretId, expiresAt)
│   │   ├── RevealSecretRequest.cs         # Inbound DTO (password)
│   │   ├── RevealSecretResponse.cs        # Outbound DTO (content)
│   │   └── ErrorResponse.cs              # Uniform error DTO (message, remainingAttempts?)
│   ├── Data/
│   │   └── SecretsDbContext.cs            # EF Core Cosmos DB context
│   └── Middleware/
│       ├── SecurityHeadersMiddleware.cs   # Remove server headers, add security headers
│       └── ExceptionHandlerMiddleware.cs  # Global exception → generic 500 response
│
├── Securer.Web/                           # Blazor InteractiveServer frontend
│   ├── Securer.Web.csproj
│   ├── Program.cs                         # Blazor app builder, service registration
│   ├── appsettings.json
│   ├── Components/
│   │   ├── App.razor                      # Root component
│   │   ├── Routes.razor                   # Router
│   │   ├── Layout/
│   │   │   ├── SplitLayoutShell.razor     # Gradient panel + form panel (sender)
│   │   │   └── MinimalLayout.razor        # Centered single-column (recipient, dead-end)
│   │   ├── Pages/
│   │   │   ├── Home.razor                 # Sender form — route: /
│   │   │   └── RevealSecret.razor         # Recipient flow — route: /s/{id}
│   │   └── Shared/
│   │       ├── GlassCard.razor            # Frosted glass form container
│   │       ├── PasswordRow.razor          # Password input + random generator button
│   │       ├── SecretDisplay.razor        # Revealed secret content + copy button
│   │       ├── CountdownTimer.razor       # Auto-clear countdown (cosmetic)
│   │       ├── SuccessPanel.razor         # Link generated confirmation
│   │       └── DeadEndScreen.razor        # Generic "not available" terminal state
│   ├── Services/
│   │   ├── ISecretApiClient.cs            # API client interface
│   │   └── SecretApiClient.cs             # HttpClient wrapper for API calls
│   ├── Interop/
│   │   ├── clipboard.js                   # navigator.clipboard.writeText wrapper
│   │   └── fragment.js                    # window.location.hash reader
│   ├── Styles/
│   │   └── tailwind.css                   # Tailwind v4 input file (@import "tailwindcss")
│   └── wwwroot/
│       ├── css/
│       │   └── app.css                    # Generated Tailwind output
│       └── favicon.ico
│
└── Securer.Tests/                         # All tests
    ├── Securer.Tests.csproj
    ├── ApiService/
    │   ├── Endpoints/
    │   │   └── SecretsEndpointsTests.cs   # Integration tests for API endpoints
    │   ├── Services/
    │   │   ├── CryptoServiceTests.cs      # Unit tests for encryption/decryption
    │   │   └── SecretServiceTests.cs      # Unit tests for business logic
    │   └── Middleware/
    │       └── SecurityHeadersTests.cs    # Verify headers and error responses
    └── Web/
        └── Components/
            └── Pages/
                ├── HomeTests.cs           # Sender form component tests
                └── RevealSecretTests.cs   # Recipient flow component tests
```

### Architectural Boundaries

**API Boundary (Securer.ApiService):**
- The ONLY entry point for data operations. Blazor frontend NEVER accesses Cosmos DB directly.
- Two endpoints, both POST. All secret logic lives behind `ISecretService`.
- `CryptoService` is internal to the API — frontend never handles encryption.

**Frontend Boundary (Securer.Web):**
- Communicates with ApiService exclusively via `SecretApiClient` (typed `HttpClient`).
- Aspire's service discovery resolves the API URL — no hardcoded URLs.
- JS interop is limited to two files for clipboard and URL fragment — no other JS.

**Data Boundary (Cosmos DB):**
- Accessed only through `SecretsDbContext` (EF Core) inside ApiService.
- Single container (`secrets`), single document type (`SecretDocument`).
- Partition key: `/id`. TTL mapped from `ExpiresAt`.

**Service Boundary (between projects):**
- AppHost → orchestrates ApiService + Web + Cosmos emulator
- ServiceDefaults → shared by ApiService and Web (telemetry, health checks)
- ApiService → depends on Cosmos DB (via Aspire resource reference)
- Web → depends on ApiService (via Aspire service discovery)

### Requirements to Structure Mapping

**Secret Creation (FR1-FR7):**
- UI: `Securer.Web/Components/Pages/Home.razor` + `PasswordRow.razor` + `GlassCard.razor`
- API: `Securer.ApiService/Endpoints/SecretsEndpoints.cs` → `POST /api/secrets`
- Logic: `SecretService.CreateSecretAsync()` → `CryptoService.EncryptAsync()`
- Data: `SecretsDbContext` → Cosmos DB document insert

**Secret Retrieval (FR8-FR12):**
- UI: `Securer.Web/Components/Pages/RevealSecret.razor` + `SecretDisplay.razor` + `CountdownTimer.razor`
- API: `Securer.ApiService/Endpoints/SecretsEndpoints.cs` → `POST /api/secrets/{id}/reveal`
- Logic: `SecretService.RevealSecretAsync()` → `CryptoService.DecryptAsync()` → delete document
- JS: `fragment.js` (password-in-link extraction), `clipboard.js` (copy revealed content)

**Secret Lifecycle (FR13-FR16):**
- Auto-expiration: Cosmos DB native TTL — no application code needed
- View destruction: `SecretService.RevealSecretAsync()` deletes after decryption
- Attempt exhaustion: `SecretService.RevealSecretAsync()` decrements counter, deletes at 0

**Error Handling (FR21-FR22):**
- API: `ExceptionHandlerMiddleware.cs` + uniform `ErrorResponse` DTO
- UI: `DeadEndScreen.razor` for all terminal states

**Security (FR17-FR20):**
- Encryption: `CryptoService.cs` (AES-256-GCM + PBKDF2)
- Headers: `SecurityHeadersMiddleware.cs`
- Logging exclusions: configured in `Program.cs` middleware pipeline

**Deployment (FR23-FR24):**
- Pipeline: `azure-pipelines.yml`
- Infra: `azure.yaml` + Aspire-generated Bicep templates
- Config: `appsettings.json` per project + Azure Key Vault in production

### Data Flow

```
Sender:
  Browser → Home.razor → SecretApiClient → POST /api/secrets
    → SecretsEndpoints → SecretService.CreateSecretAsync()
      → CryptoService.EncryptAsync(plaintext, password)
        → PBKDF2(password, salt) → AES-256-GCM(key, plaintext)
      → SecretsDbContext.SaveAsync(document)
    ← { secretId, expiresAt }
  ← SuccessPanel (link copied to clipboard)

Recipient:
  Browser → RevealSecret.razor → (fragment.js extracts password if in URL)
    → Confirmation dialog (client-side gate)
    → SecretApiClient → POST /api/secrets/{id}/reveal
      → SecretsEndpoints → SecretService.RevealSecretAsync()
        → Lookup document by ID
        → CryptoService.DecryptAsync(ciphertext, password, salt, iv)
        → Delete document from Cosmos DB
      ← { content } or 404
  ← SecretDisplay (with copy + countdown) or DeadEndScreen
```

## Architecture Validation Results

### Coherence Validation ✅

**Decision Compatibility:** All technology choices are first-party Microsoft and fully compatible. .NET 10 + Aspire 13 + Blazor InteractiveServer + EF Core Cosmos + Azure Container Apps — no version conflicts. Tailwind CSS v4 integrates via CLI build step with no .NET toolchain conflicts. AES-256-GCM + PBKDF2 both in `System.Security.Cryptography` — zero external dependencies.

**Pattern Consistency:** C# naming conventions, camelCase JSON serialization, folder-by-concern structure — all standard .NET, consistent across all projects.

**Structure Alignment:** Four-project Aspire structure with clean boundaries (Web → ApiService → Cosmos DB). No circular dependencies.

### Requirements Coverage Validation ✅

**Functional Requirements:** All 24 FRs (FR1-FR24) have explicit architectural support with specific files and components mapped.

**Non-Functional Requirements:** Performance (sub-500ms via .NET crypto + Cosmos point-reads), security (TLS via Azure, logging exclusions, memory hygiene), and scalability (stateless API, Cosmos DB, Container Apps auto-scaling) all architecturally addressed.

### Implementation Readiness Validation ✅

**Decision Completeness:** All technology choices have specific versions. Encryption, API contracts, and deployment strategy fully specified.

**Structure Completeness:** Every file has a defined location. Every FR maps to specific files. Data flow documented end-to-end.

**Pattern Completeness:** Naming, structure, format, process, and enforcement patterns all defined with examples.

### Gap Analysis Results

**Critical Gaps:** None.

**Resolved Minor Gaps:**
- Testing framework: **xUnit** — most common in .NET ecosystem, used in Aspire samples
- CORS: **Not needed** — Blazor InteractiveServer calls API from the server side, not the browser. No cross-origin requests.
- Health checks: **Aspire ServiceDefaults** provides health check infrastructure automatically, including Cosmos DB connectivity checks. No custom implementation needed.

### Architecture Completeness Checklist

**✅ Requirements Analysis**
- [x] Project context thoroughly analyzed
- [x] Scale and complexity assessed
- [x] Technical constraints identified
- [x] Cross-cutting concerns mapped

**✅ Architectural Decisions**
- [x] Critical decisions documented with versions
- [x] Technology stack fully specified
- [x] Integration patterns defined
- [x] Performance considerations addressed

**✅ Implementation Patterns**
- [x] Naming conventions established
- [x] Structure patterns defined
- [x] Communication patterns specified
- [x] Process patterns documented

**✅ Project Structure**
- [x] Complete directory structure defined
- [x] Component boundaries established
- [x] Integration points mapped
- [x] Requirements to structure mapping complete

### Architecture Readiness Assessment

**Overall Status:** READY FOR IMPLEMENTATION

**Confidence Level:** High

**Key Strengths:**
- Extremely simple architecture — two endpoints, one database, one document type
- Zero external dependencies for crypto — all built into .NET
- Cosmos DB native TTL eliminates background job complexity
- Single atomic reveal simplifies the most critical code path
- Aspire handles orchestration, telemetry, and deployment plumbing
- Clean boundaries make each component independently testable

**Areas for Future Enhancement:**
- Rate limiting (Phase 2)
- Public hosted instance with abuse prevention (Phase 2)
- Multi-region Cosmos DB replication if worldwide scale demands it (Phase 3)

### Implementation Handoff

**AI Agent Guidelines:**
- Follow all architectural decisions exactly as documented
- Use implementation patterns consistently across all components
- Respect project structure and boundaries
- Refer to this document for all architectural questions

**First Implementation Priority:**
```bash
dotnet new aspire-starter --output Securer
```

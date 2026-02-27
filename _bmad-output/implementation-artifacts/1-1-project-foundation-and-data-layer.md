# Story 1.1: Project Foundation & Data Layer

Status: ready-for-dev

<!-- Note: Validation is optional. Run validate-create-story for quality check before dev-story. -->

## Story

As a developer,
I want the Aspire starter project scaffolded with Cosmos DB and Tailwind CSS configured,
So that I have a working foundation to build features on.

## Acceptance Criteria

**Given** the project is initialized with `dotnet new aspire-starter --output Securer`
**When** I run the AppHost project
**Then** the Aspire dashboard launches with the Web and ApiService projects running
**And** the Cosmos DB emulator container starts via Aspire's `.RunAsEmulator()`

**Given** the ApiService project
**When** I inspect the data layer
**Then** a `SecretDocument` model exists with fields: Id, Ciphertext, Salt, IV, ExpiresAt, RemainingAttempts, CreatedAt
**And** a `SecretsDbContext` (EF Core Cosmos provider) is configured with a `secrets` container and `/id` partition key

**Given** the Web project
**When** I run the application
**Then** Tailwind CSS v4 is integrated via CLI with a build target that generates `wwwroot/css/app.css`
**And** the Inter and JetBrains Mono fonts are loaded

**Given** the project structure
**When** I inspect the solution
**Then** it follows the architecture document's folder structure (AppHost, ServiceDefaults, ApiService, Web, Tests)
**And** ServiceDefaults provides OpenTelemetry and health check configuration

## Tasks / Subtasks

- [ ] Initialize Aspire Starter Project (AC: 1)
  - [ ] Run `dotnet new aspire-starter --output Securer` command
  - [ ] Verify solution structure: AppHost, ServiceDefaults, ApiService, Web projects created
  - [ ] Open solution and build to verify all projects compile
  - [ ] Run AppHost and verify Aspire dashboard launches

- [ ] Configure Cosmos DB with Emulator (AC: 1, 2)
  - [ ] Add Cosmos DB emulator to AppHost via `.AddAzureCosmosDB().RunAsEmulator()`
  - [ ] Configure ApiService to reference Cosmos DB resource via Aspire service discovery
  - [ ] Add `Microsoft.EntityFrameworkCore.Cosmos` NuGet package to ApiService
  - [ ] Verify emulator container starts when running AppHost

- [ ] Implement Data Layer (AC: 2)
  - [ ] Create `SecretDocument` model class in `ApiService/Models/` with required fields:
    - Id (string, partition key)
    - Ciphertext (byte array)
    - Salt (byte array)
    - IV (byte array)
    - ExpiresAt (DateTimeOffset, maps to Cosmos TTL)
    - RemainingAttempts (int, default 3-5)
    - CreatedAt (DateTimeOffset)
  - [ ] Create `SecretsDbContext` in `ApiService/Data/` extending DbContext
  - [ ] Configure DbContext for Cosmos DB with container name "secrets" and partition key "/id"
  - [ ] Configure TTL mapping for ExpiresAt field
  - [ ] Register DbContext in ApiService Program.cs with DI
  - [ ] Add database initialization/ensure created logic

- [ ] Integrate Tailwind CSS v4 (AC: 3)
  - [ ] Install Tailwind CSS CLI (via npm or standalone binary)
  - [ ] Create `Styles/tailwind.css` input file in Web project with `@import "tailwindcss"`
  - [ ] Configure build target to run Tailwind CLI and generate `wwwroot/css/app.css`
  - [ ] Add Tailwind output CSS reference to App.razor or index layout
  - [ ] Verify CSS generates correctly and applies to a test component

- [ ] Configure Typography (AC: 3)
  - [ ] Add Inter font via Google Fonts CDN or local files
  - [ ] Add JetBrains Mono font via Google Fonts CDN or local files
  - [ ] Configure Tailwind theme to use Inter as default sans-serif
  - [ ] Configure JetBrains Mono as monospace font family
  - [ ] Test font loading in browser dev tools

- [ ] Verify Complete Project Structure (AC: 4)
  - [ ] Confirm folder structure matches architecture doc:
    - ApiService: Endpoints/, Services/, Models/, Data/, Middleware/ folders
    - Web: Components/Layout/, Components/Pages/, Components/Shared/, Services/, Interop/, wwwroot/
  - [ ] Create placeholder README.md files in key folders
  - [ ] Verify ServiceDefaults configures OpenTelemetry and health checks
  - [ ] Run full solution and verify all services start correctly

## Dev Notes

### 🎯 CRITICAL DEVELOPER CONTEXT — READ BEFORE IMPLEMENTATION

This is **THE FOUNDATION STORY** — everything else depends on this being done correctly. This isn't just "scaffold a project" — you're establishing the architectural skeleton, naming conventions, folder structure, and technology integrations that EVERY subsequent story will build upon.

**DO NOT RUSH THIS. DO NOT TAKE SHORTCUTS. DO NOT DEVIATE FROM THE ARCHITECTURE DOCUMENT.**

### Architecture Alignment — NON-NEGOTIABLE DECISIONS

**Technology Stack (from Architecture Doc):**
- **.NET 10 LTS** — Latest long-term support, C# 14 features available
- **Aspire 13** — Service orchestration, telemetry, and deployment infrastructure
- **Blazor InteractiveServer** — SignalR-based rendering, no WASM overhead
- **Azure Cosmos DB (NoSQL API)** — Sub-10ms point reads, native TTL for auto-expiration
- **EF Core Cosmos Provider** — Standard .NET data access pattern
- **Tailwind CSS v4** — Utility-first CSS via CLI, no framework "look"

**Project Initialization Command (EXACT COMMAND FROM ARCHITECTURE):**
```bash
dotnet new aspire-starter --output Securer
```

**Why this starter?** Provides exactly what Securer needs: AppHost orchestration, ServiceDefaults for telemetry/health checks, a Blazor Web frontend, and a Minimal API backend. This is the official Microsoft template aligned with .NET 10 LTS best practices.

**Expected Project Structure After Initialization:**
```
Securer/
├── Securer.sln
├── Securer.AppHost/              # Orchestration project
├── Securer.ServiceDefaults/      # Shared configuration
├── Securer.ApiService/           # Backend API
└── Securer.Web/                  # Blazor frontend
```

You will also need to create a `Securer.Tests/` project later (not in this story, but keep it in mind).

### Cosmos DB Configuration — CRITICAL DATA LAYER DECISIONS

**Container Configuration:**
- **Container name:** `secrets` (lowercase plural)
- **Partition key:** `/id` (forward slash required in Cosmos DB path syntax)
- **TTL configuration:** Must map `SecretDocument.ExpiresAt` to Cosmos DB's native TTL property
- **Why Cosmos DB?** Point-read lookups by secret ID are sub-10ms. Native TTL eliminates the need for background cleanup jobs. Stateless API can scale horizontally with no session affinity concerns.

**SecretDocument Model — EXACT SCHEMA:**

```csharp
public class SecretDocument
{
    public string Id { get; set; } = string.Empty;  // Partition key, unique secret ID
    public byte[] Ciphertext { get; set; } = Array.Empty<byte>();  // Encrypted secret content
    public byte[] Salt { get; set; } = Array.Empty<byte>();  // PBKDF2 salt for key derivation
    public byte[] IV { get; set; } = Array.Empty<byte>();  // AES-256-GCM initialization vector
    public DateTimeOffset ExpiresAt { get; set; }  // Maps to Cosmos TTL
    public int RemainingAttempts { get; set; } = 3;  // Brute-force protection counter
    public DateTimeOffset CreatedAt { get; set; }  // Audit timestamp
}
```

**Why these fields?**
- **Ciphertext, Salt, IV:** Required for AES-256-GCM encryption with PBKDF2 key derivation (Story 1.2 will implement the crypto service)
- **ExpiresAt:** Cosmos DB's native TTL automatically deletes documents when this timestamp passes (FR14)
- **RemainingAttempts:** Brute-force protection — decremented on wrong password, secret deleted when 0 (FR15, FR16)
- **CreatedAt:** Audit trail for debugging and analytics

**EF Core Cosmos Configuration:**

You need to:
1. Install `Microsoft.EntityFrameworkCore.Cosmos` NuGet package
2. Create `SecretsDbContext : DbContext` in `ApiService/Data/`
3. Configure Cosmos DB connection via Aspire service discovery
4. Map `SecretDocument` to container "secrets"
5. Configure partition key as `/id`
6. Map `ExpiresAt` to Cosmos TTL property (use `.UsePropertyAccessMode(PropertyAccessMode.Property)` and custom TTL configuration)

**Important:** Aspire's `.AddAzureCosmosDB().RunAsEmulator()` starts the Cosmos DB emulator as a Docker container for local development. You don't need to manually install or start the emulator — Aspire handles it.

### Tailwind CSS v4 Integration — EXACT SETUP

**Why Tailwind CSS?**
From Architecture Doc: "Tailwind CSS v4 via CLI with build target automation. Full visual control to implement the UX spec's custom design (glassmorphism, split layout, indigo accent system). No component library overhead."

**Integration Steps:**
1. Install Tailwind CSS CLI (standalone binary or via npm)
2. Create `Web/Styles/tailwind.css` with content:
   ```css
   @import "tailwindcss";
   ```
3. Add a build target to `Web.csproj` that runs Tailwind CLI **before** build:
   ```xml
   <Target Name="BuildTailwind" BeforeTargets="Build">
     <Exec Command="npx tailwindcss -i ./Styles/tailwind.css -o ./wwwroot/css/app.css" />
   </Target>
   ```
4. Reference generated CSS in `App.razor` or layout:
   ```html
   <link rel="stylesheet" href="css/app.css" />
   ```

**Typography Setup:**
- **Inter font:** Primary font for all text (from UX spec)
- **JetBrains Mono:** Monospace for secret content and generated links (from UX spec)

Add to Tailwind config:
```javascript
// tailwind.config.js
module.exports = {
  theme: {
    extend: {
      fontFamily: {
        sans: ['Inter', 'system-ui', 'sans-serif'],
        mono: ['JetBrains Mono', 'monospace']
      }
    }
  }
}
```

Load fonts via Google Fonts in the layout:
```html
<link rel="preconnect" href="https://fonts.googleapis.com">
<link rel="preconnect" href="https://fonts.gstatic.com" crossorigin>
<link href="https://fonts.googleapis.com/css2?family=Inter:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap" rel="stylesheet">
```

### Folder Structure — MANDATORY CONVENTIONS

**From Architecture Doc — Section: Project Structure & Boundaries**

Your mission is to establish this structure NOW so future stories can follow it blindly:

```
Securer.ApiService/
├── Endpoints/                # Minimal API endpoint definitions
├── Services/                 # Business logic services
├── Models/                   # DTOs and document models
├── Data/                     # EF Core DbContext
└── Middleware/               # Custom middleware (future stories)

Securer.Web/
├── Components/
│   ├── Layout/               # Page shells and layouts
│   ├── Pages/                # Route pages (@page directives)
│   └── Shared/               # Reusable components
├── Services/                 # API client services
├── Interop/                  # JS interop files
├── Styles/                   # Tailwind input CSS
└── wwwroot/                  # Static assets, generated CSS
```

**Create empty folders with placeholder README.md files** if they won't have content yet. This prevents future developers from creating inconsistent structures.

### Naming Conventions — C# STANDARDS (FROM ARCHITECTURE DOC)

**CRITICAL: ALL CODE MUST FOLLOW THESE EXACTLY**

- **Classes/Records:** PascalCase — `SecretDocument`, `SecretsDbContext`
- **Interfaces:** `I` prefix — `ISecretService`, `ICryptoService`
- **Methods:** PascalCase — `CreateSecretAsync`, `GetSecretByIdAsync`
- **Private fields:** `_camelCase` — `_dbContext`, `_logger`
- **Properties:** PascalCase — `Id`, `Ciphertext`, `ExpiresAt`
- **Local variables:** camelCase — `secretId`, `encryptedData`
- **Async methods:** ALWAYS suffix with `Async` — `SaveChangesAsync`, not `SaveChanges`

**Cosmos DB Naming:**
- Container name: `secrets` (lowercase plural)
- Document properties: PascalCase (C# convention, EF Core maps naturally)
- Partition key path: `/id` (lowercase)

**API Naming (for future reference):**
- Endpoints: lowercase, plural nouns — `/api/secrets`
- JSON fields: camelCase (ASP.NET auto-converts PascalCase → camelCase)

### Logging and Telemetry — ASPIRE SERVICEDEFAULTS

**From Architecture Doc:**
- Use `ILogger<T>` everywhere — ServiceDefaults configures OpenTelemetry sinks automatically
- **NEVER log:** secret content, passwords, ciphertext, request bodies on `/api/secrets` endpoints (you won't implement logging exclusions yet, but be aware)
- **DO log:** secret ID (non-sensitive), operation type, success/failure, timestamps

For this story, just verify that:
1. ServiceDefaults project exists and is referenced by ApiService and Web
2. OpenTelemetry is configured in ServiceDefaults (Aspire starter does this by default)
3. Health checks are configured (Aspire starter does this by default)

You don't need to add custom logging yet — just ensure the foundation is there.

### Testing Setup — FUTURE STORY, BUT PLAN AHEAD

**From Architecture Doc:**
- Tests in a **separate project** (`Securer.Tests`), NOT co-located — standard .NET convention
- xUnit framework (most common in .NET ecosystem)

You won't create tests in this story, but:
1. Note where the `Securer.Tests/` project should be added in future
2. Ensure solution structure doesn't block adding it later

### AppHost Configuration — ASPIRE ORCHESTRATION

**Your tasks:**
1. Add Cosmos DB emulator resource to AppHost `Program.cs`:
   ```csharp
   var cosmosDb = builder.AddAzureCosmosDB("cosmosdb").RunAsEmulator();
   ```
2. Reference Cosmos DB in ApiService:
   ```csharp
   var apiService = builder.AddProject<Projects.Securer_ApiService>("apiservice")
       .WithReference(cosmosDb);
   ```
3. Ensure ApiService and Web are already wired up by the starter template
4. Run AppHost and verify Aspire dashboard shows all services + Cosmos DB emulator

**Why this matters:** Aspire handles service discovery, connection string management, and container orchestration. You don't manually configure connection strings — Aspire injects them at runtime.

### Security and Performance Notes — FORWARD-LOOKING

**From Architecture Doc:**
- **Memory hygiene:** Use `Span<byte>` and `CryptographicOperations.ZeroMemory()` for sensitive buffers (Story 1.2 will implement this for crypto)
- **No plaintext storage:** Only encrypted ciphertext persists (verified in Story 1.2)
- **Stateless API:** Any instance can serve any request (validated by Cosmos DB partition key design)

For this story, you're just setting up the data model that WILL enforce these constraints. The actual encryption logic comes in Story 1.2.

### Common Pitfalls — AVOID THESE MISTAKES

**❌ DO NOT:**
- Install the full Cosmos DB emulator locally — use Aspire's `.RunAsEmulator()` Docker container
- Hard-code connection strings — use Aspire service discovery
- Create a "Models" folder in Web for domain models — keep data models in ApiService
- Use synchronous DbContext methods (`Find`, `SaveChanges`) — always use async (`FindAsync`, `SaveChangesAsync`)
- Log sensitive data or connection strings
- Deviate from the folder structure defined in the architecture doc
- Use different naming conventions from the architecture doc

**✅ DO:**
- Run `dotnet new aspire-starter --output Securer` as the FIRST command
- Add Cosmos DB to AppHost with `.RunAsEmulator()`
- Install `Microsoft.EntityFrameworkCore.Cosmos` NuGet package in ApiService
- Create `SecretDocument` model with exact schema specified above
- Create `SecretsDbContext` with container "secrets" and partition key "/id"
- Integrate Tailwind CSS v4 via CLI with build target
- Load Inter and JetBrains Mono fonts
- Verify Aspire dashboard launches with all services running
- Commit frequently with clear messages

### Validation Checklist — MUST PASS BEFORE MARKING DONE

Before marking this story complete, verify:

1. **Project structure:**
   - [ ] Solution has AppHost, ServiceDefaults, ApiService, Web projects
   - [ ] AppHost runs and launches Aspire dashboard
   - [ ] All projects compile without errors

2. **Cosmos DB:**
   - [ ] Cosmos DB emulator starts as Docker container via Aspire
   - [ ] ApiService references Cosmos DB resource
   - [ ] `SecretsDbContext` is configured and registered in DI
   - [ ] `SecretDocument` model matches exact schema above

3. **Tailwind CSS:**
   - [ ] Tailwind CLI build target runs before build
   - [ ] `wwwroot/css/app.css` is generated
   - [ ] CSS is referenced and applies to test component

4. **Typography:**
   - [ ] Inter font loads (check browser dev tools)
   - [ ] JetBrains Mono font loads (check browser dev tools)

5. **Folder structure:**
   - [ ] ApiService has Endpoints/, Services/, Models/, Data/ folders
   - [ ] Web has Components/Layout/, Components/Pages/, Components/Shared/, Services/, Interop/, Styles/, wwwroot/

6. **AppHost orchestration:**
   - [ ] Running AppHost starts ApiService, Web, and Cosmos DB emulator
   - [ ] Aspire dashboard shows all services healthy

### Project Structure Notes

**Alignment with Unified Project Structure:**

This story establishes the exact structure defined in the Architecture Doc section "Project Structure & Boundaries" (lines 342-433). Every folder created here is referenced by name in the architecture doc, and future stories will assume this structure exists.

**Key Structural Decisions:**
- **ApiService** owns all data models and business logic — Web never accesses Cosmos DB directly
- **Web** communicates with ApiService exclusively via HTTP client (future stories)
- **ServiceDefaults** is shared by both ApiService and Web for telemetry and health checks
- **AppHost** orchestrates all services and dependencies (Cosmos DB emulator)

**No conflicts expected** — this is a greenfield project, and the Aspire starter template aligns perfectly with the architecture doc's structure.

### References

**All technical specifications from:**

- **Architecture Document:** `_bmad-output/planning-artifacts/architecture.md`
  - Starter Template: Lines 79-114
  - Data Architecture: Lines 137-142
  - Project Structure: Lines 342-433
  - Naming Conventions: Lines 201-226
  - Technology Stack: Lines 49-53

- **Epics Document:** `_bmad-output/planning-artifacts/epics.md`
  - Story 1.1: Lines 177-203
  - Acceptance Criteria: Lines 183-203
  - Requirements Coverage: Lines 86-104 (Architecture section), 107-124 (UX section)

- **UX Design Specification:** `_bmad-output/planning-artifacts/ux-design-specification.md`
  - Typography (Inter + JetBrains Mono): Line 412 (fonts section)
  - Color System: Lines 289-300 (to be used in future UI stories)
  - Design System Choice (Tailwind CSS): Lines 173-201

- **PRD:** `_bmad-output/planning-artifacts/prd.md`
  - Referenced via epics for FR17, FR18 (encryption requirements that drive data model design)

### Implementation Sequence Recommendation

**STEP-BY-STEP ORDER (suggested):**

1. **Initialize Aspire Starter** — Run `dotnet new aspire-starter --output Securer`, open solution, build to verify
2. **Add Cosmos DB to AppHost** — Configure emulator, run to verify it starts
3. **Create Data Layer** — `SecretDocument` model, `SecretsDbContext`, register in DI
4. **Integrate Tailwind CSS** — CLI setup, build target, test output generation
5. **Load Fonts** — Add Google Fonts links, configure Tailwind theme
6. **Create Folder Structure** — Add empty folders with README placeholders
7. **Final Verification** — Run AppHost, check Aspire dashboard, verify all services healthy

**Estimated Effort:** 1-2 hours for an experienced .NET developer, 3-4 hours for someone new to Aspire.

### Known Unknowns & Decisions Deferred

**Questions for user if issues arise:**
- Should Tailwind be installed via npm (requires Node.js) or standalone binary?
- Should fonts be CDN-loaded (Google Fonts) or self-hosted for performance?
- Cosmos DB emulator configuration: any custom settings needed (default should work)?

**Decisions explicitly deferred to later stories:**
- API endpoint implementation (Story 1.2)
- Encryption service implementation (Story 1.2)
- UI components (Stories 1.3, 1.4)
- Security middleware (Story 3.2)
- Deployment configuration (Story 4.1)

### Git History Context

**Recent commits:**
- `08fe0e2` — Add copilot agent to bmad (tooling setup)
- `111a3b8` — Create sprint (sprint planning completed)
- `9a5e645` — Perform 'Implementation Readiness Check' (validation passed)
- `f469e42` — Create epics and stories (this story originates from epics.md)
- `021f5ff` — Finish architecture (all architecture decisions finalized)

**What this means for you:**
- Architecture is **LOCKED** — no guessing, all decisions are documented
- Epics and stories are **FINALIZED** — follow acceptance criteria exactly
- This is **FIRST IMPLEMENTATION STORY** — no prior code patterns to follow, you're establishing them
- Next commits should be incremental implementation of this story's tasks

### Success Definition — WHAT "DONE" LOOKS LIKE

**You will know this story is complete when:**

1. A team member can clone the repo, run `dotnet run --project Securer.AppHost`, and see the Aspire dashboard launch with 3 services running (ApiService, Web, Cosmos DB emulator)
2. The `SecretDocument` model exists in `ApiService/Models/` with all 7 required fields
3. The `SecretsDbContext` is configured and registered in DI, ready for Story 1.2 to use
4. Tailwind CSS compiles to `wwwroot/css/app.css` and applies to a test component
5. Inter and JetBrains Mono fonts load (verified in browser dev tools)
6. The folder structure matches the architecture doc EXACTLY
7. All projects compile with zero errors
8. No warnings about missing dependencies or configuration

**The developer working on Story 1.2 should NOT need to:**
- Fix project structure issues
- Reconfigure Cosmos DB
- Install missing NuGet packages for EF Core Cosmos
- Set up Tailwind CSS
- Diagnose why fonts aren't loading

**If Story 1.2 encounters ANY of those issues, Story 1.1 is not done.**

---

## Dev Agent Record

### Agent Model Used

_To be filled by dev agent_

### Debug Log References

_To be filled by dev agent during implementation_

### Completion Notes List

_To be filled by dev agent with lessons learned, gotchas, and any deviations from plan_

### File List

_To be filled by dev agent with all files created or modified_

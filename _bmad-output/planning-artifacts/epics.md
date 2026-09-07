---
stepsCompleted:
  - step-01-validate-prerequisites
  - step-02-design-epics
  - step-03-create-stories
  - step-04-final-validation
inputDocuments:
  - _bmad-output/planning-artifacts/prds/prd-Securer-2026-02-13/prd.md
  - _bmad-output/planning-artifacts/architecture/architecture-Securer-2026-02-13/ARCHITECTURE-SPINE.md
  - _bmad-output/planning-artifacts/ux-designs/ux-Securer-2026-02-13/DESIGN.md
  - _bmad-output/planning-artifacts/ux-designs/ux-Securer-2026-02-13/EXPERIENCE.md
---

# Securer - Epic Breakdown

## Overview

This document provides the complete epic and story breakdown for Securer, decomposing the requirements from the PRD, UX Design if it exists, and Architecture requirements into implementable stories.

Requirement ids (`FR-`, `NFR-`) are owned by the PRD; architecture decision ids (`AD-`) by the architecture spine; design tokens by `DESIGN.md`. This document references them and does not redefine them.

## Requirements Inventory

### Functional Requirements

**Secret Creation (FR-1 to FR-7):**
- FR-1: Sender can input secret text content into a text field
- FR-2: Sender can select an expiration period (1 hour, 24 hours, 7 days)
- FR-3: Sender can provide a password required to access the secret
- FR-4: Sender can generate a random password via the UI instead of typing one manually
- FR-5: Sender can optionally include the password in the generated link (via URL fragment)
- FR-6: Sender can generate a unique, shareable link for the created secret
- FR-7: Sender receives the generated link automatically copied to their clipboard

**Secret Retrieval (FR-8 to FR-12):**
- FR-8: Recipient can open a secret link and be prompted for a password
- FR-9: Recipient with a password-in-link URL can access the secret without a separate password prompt
- FR-10: Recipient can view a confirmation warning before the secret is revealed
- FR-11: Recipient can view the decrypted secret content after providing the correct password and confirming
- FR-12: Recipient can copy the revealed secret content

**Secret Lifecycle (FR-13 to FR-16):**
- FR-13: System destroys the secret immediately after it has been viewed once
- FR-14: System destroys unclaimed secrets automatically when their expiration period elapses
- FR-15: System destroys the secret after the maximum number of failed password attempts is reached
- FR-16: System tracks remaining password attempts per secret

**Security & Encryption (FR-17 to FR-20):**
- FR-17: System encrypts secret content server-side using the sender-provided password before storing
- FR-18: System stores only encrypted ciphertext - plaintext never persists in storage
- FR-19: System decrypts secret content in memory only at the moment of authorized retrieval
- FR-20: System discards plaintext and password from memory immediately after encryption or decryption

**Error Handling (FR-21 to FR-22):**
- FR-21: System displays an identical generic message for all failure states (expired, viewed, burned, nonexistent)
- FR-22: System reveals no information about whether a secret ever existed, was already viewed, or was destroyed

**Deployment (FR-23 to FR-24):**
- FR-23: Administrator can deploy the application using a single `docker-compose up` command
- FR-24: Administrator can configure basic settings (port, database) via environment variables

### NonFunctional Requirements

Cross-cutting NFRs are owned by PRD section 5 and carry stable `NFR-` ids there. Reproduced here as the coverage inventory.

**Performance:**
- NFR-1: Page load time under 1 second on standard broadband connections
- NFR-2: Secret creation (encryption + storage) completes in under 500ms server-side
- NFR-3: Secret retrieval (decryption + delivery) completes in under 500ms server-side
- NFR-4: UI remains responsive during link generation (no blocking operations on the client)
- NFR-5: Minimal JS bundle size

**Security:**
- NFR-6: All client-server communication over TLS (HTTPS only)
- NFR-7: No logging of secret content, passwords, or decrypted data at any point
- NFR-8: No logging of request bodies on secret creation or retrieval endpoints
- NFR-9: Encryption keys (user passwords) never written to disk, logs, or persistent storage
- NFR-10: Failed password attempts do not reveal whether the secret exists
- NFR-11: URL fragments (password-in-link mode) are never sent to the server by the browser

**Reliability:**
- NFR-12: Destruction is authoritative — once a destruction trigger fires, no later request may observe the secret, regardless of storage cleanup timing
- NFR-13: A reveal either delivers the secret and destroys the record, or does neither — a displayed secret must never survive

**Scalability:**
- NFR-14: System supports thousands of concurrent users worldwide
- NFR-15: Stateless request handling - any instance can serve any request
- NFR-16: Database operations are simple key-value lookups (O(1) by secret ID)
- NFR-17: Horizontal scaling possible by adding container instances behind a load balancer
- NFR-18: Storage grows linearly with active (unexpired) secrets only - expired secrets are cleaned up
- NFR-19: Single small VPS (1-2 GB RAM) handles typical usage; scales horizontally for worldwide adoption

**Observability:**
- NFR-20: Health and telemetry surfaces exist for operating an instance and carry no secret-derived data

### Additional Requirements

**From Architecture:**
- Starter template: `dotnet new aspire-starter --output Securer` — project initialization as first implementation step
- .NET 10 LTS with Aspire 13 for orchestration
- Blazor InteractiveServer render mode (SignalR connection, no WASM)
- Azure Cosmos DB (NoSQL API) with native TTL for auto-expiration — eliminates need for background cleanup job
- EF Core with Cosmos DB provider for data access
- AES-256-GCM encryption via System.Security.Cryptography + PBKDF2 key derivation via Rfc2898DeriveBytes
- Memory hygiene: Span<byte> and CryptographicOperations.ZeroMemory() for sensitive buffers
- Constant-time comparison via CryptographicOperations.FixedTimeEquals() for timing attack mitigation
- ASP.NET Core Minimal APIs — two POST endpoints only (POST /api/secrets, POST /api/secrets/{id}/reveal)
- Single atomic reveal — password verification, decryption, and deletion in one request
- Uniform 404 error response for ALL failure states
- Tailwind CSS v4 via CLI with build target automation
- JS interop limited to clipboard API and URL fragment reading
- Azure Container Apps deployment with azd CLI
- Azure DevOps Pipelines for CI/CD
- Azure Monitor via Aspire's OpenTelemetry (with sensitive endpoint exclusions)
- Security headers middleware + global exception handler middleware
- xUnit for testing framework

**From UX Design:**

Visual decisions are owned by `DESIGN.md` and behavioral decisions by `EXPERIENCE.md` in the UX run folder; both win over any mock. Token names below are `DESIGN.md` references.

- Split layout: gradient info panel (`{components.split-panel}`, left) + glass card form (`{components.glass-card}`, right) for the sender page
- Centered minimal layout for recipient and dead-end views — no glass treatment on recipient surfaces
- `{typography.body.fontFamily}` Inter for interface text, `{typography.mono.fontFamily}` JetBrains Mono for all user data (secret input, revealed secret, generated URL)
- Color system: `{colors.accent}` #6366F1 indigo accent, `{colors.success}` emerald confirmation, `{colors.error}` red for incorrect password and validation only, `{colors.warning}` amber reserved for irreversible destruction
- Dead-end screen uses neutral `{colors.text-tertiary}` with no error styling — it is a terminal state, not an error
- WCAG 2.1 Level AA accessibility compliance
- Keyboard navigation: full tab-through, Enter submits, Escape closes dialogs
- Screen reader support: semantic HTML, aria-live regions, proper ARIA roles
- Visible focus indicators (indigo border plus 3px indigo glow at 10% opacity)
- prefers-reduced-motion support - disable animations for users who prefer it
- Desktop-first responsive: info panel hidden on mobile (<768px)
- Glass card effect with backdrop-filter blur behind `@supports`, solid white fallback
- 44px minimum touch targets on mobile
- No information conveyed by color alone
- Auto-copy to clipboard on link generation, with a visible link and manual copy fallback
- Confirmation dialog before secret reveal (role="alertdialog") - the only dialog in the product
- Toast notification for clipboard confirmation (auto-dismiss 3s)
- Countdown timer with auto-clear after reveal (cosmetic)
- Form validation on submit only, not on blur

### FR Coverage Map

| FR | Epic | Description |
|---|---|---|
| FR-1 | Epic 1 | Secret text input |
| FR-2 | Epic 1 | Expiration selection |
| FR-3 | Epic 1 | Password input |
| FR-4 | Epic 1 | Random password generator |
| FR-5 | Epic 1 | Password-in-link option |
| FR-6 | Epic 1 | Unique link generation |
| FR-7 | Epic 1 | Auto-copy to clipboard |
| FR-8 | Epic 2 | Password prompt on link open |
| FR-9 | Epic 2 | Password-in-link bypass |
| FR-10 | Epic 2 | Confirmation before reveal |
| FR-11 | Epic 2 | View decrypted content |
| FR-12 | Epic 2 | Copy revealed secret |
| FR-13 | Epic 2 | Destroy after single view |
| FR-14 | Epic 3 | Auto-expire unclaimed secrets |
| FR-15 | Epic 3 | Destroy on failed attempt exhaustion |
| FR-16 | Epic 3 | Track remaining attempts |
| FR-17 | Epic 1 | Server-side encryption |
| FR-18 | Epic 1 | Store only ciphertext |
| FR-19 | Epic 2 | Decrypt in memory at retrieval |
| FR-20 | Epic 2 | Discard plaintext/password after use |
| FR-21 | Epic 3 | Identical generic error message |
| FR-22 | Epic 3 | No information leakage |
| FR-23 | Epic 4 | Docker single-command deploy |
| FR-24 | Epic 4 | Environment variable config |

## Epic List

### Epic 1: Secret Creation
A sender can paste a secret, set a password (or generate one), choose expiration, optionally include password in link, and get a shareable link auto-copied to clipboard. Includes project scaffolding, database setup, crypto service, create API endpoint, and full sender UI (split layout, glass card, form).
**FRs covered:** FR-1, FR-2, FR-3, FR-4, FR-5, FR-6, FR-7, FR-17, FR-18

### Epic 2: Secret Retrieval
A recipient can open a shared link, enter a password (or bypass via password-in-link), see a confirmation warning, view the decrypted secret, and copy it. The secret is destroyed immediately after viewing. Completes the end-to-end secret sharing loop.
**FRs covered:** FR-8, FR-9, FR-10, FR-11, FR-12, FR-13, FR-19, FR-20

### Epic 3: Security Hardening & Lifecycle
System enforces brute-force protection (limited attempts, secret burned on exhaustion), auto-expires unclaimed secrets via Cosmos DB TTL, and returns identical generic responses for all failure states. Adds security headers middleware and sensitive logging exclusions.
**FRs covered:** FR-14, FR-15, FR-16, FR-21, FR-22

### Epic 4: Deployment & Operations
Administrator can deploy with a single command (Docker/Azure Container Apps), configure via environment variables, and have CI/CD pipeline and monitoring in place.
**FRs covered:** FR-23, FR-24

## Epic 1: Secret Creation

A sender can paste a secret, set a password (or generate one), choose expiration, optionally include password in link, and get a shareable link auto-copied to clipboard. Includes project scaffolding, database setup, crypto service, create API endpoint, and full sender UI.

### Story 1.1: Project Foundation & Data Layer

As a developer,
I want the Aspire starter project scaffolded with Cosmos DB and Tailwind CSS configured,
So that I have a working foundation to build features on.

**Acceptance Criteria:**

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

### Story 1.2: Encryption Service & Secret Creation API

As a sender,
I want to create an encrypted secret via the API,
So that my secret is stored securely with zero-knowledge encryption.

**Acceptance Criteria:**

**Given** the CryptoService implements ICryptoService
**When** `EncryptAsync(plaintext, password)` is called
**Then** it derives a 256-bit key using PBKDF2 (`Rfc2898DeriveBytes`) with a random salt
**And** encrypts using AES-256-GCM via `System.Security.Cryptography`
**And** returns ciphertext, salt, and IV
**And** clears sensitive buffers using `CryptographicOperations.ZeroMemory()`

**Given** a valid `POST /api/secrets` request with `{ content, password, expirationMinutes, includePasswordInLink }`
**When** the endpoint processes the request
**Then** SecretService encrypts the content, stores a SecretDocument in Cosmos DB, and returns `{ secretId, expiresAt }`
**And** plaintext and password are never persisted to storage (FR-18)
**And** the document's TTL is set based on the selected expiration period

**Given** an invalid request (missing content or password)
**When** the endpoint processes the request
**Then** it returns `400` with field-specific validation errors

**Given** the encryption operation
**When** it completes
**Then** `Span<byte>` is used for sensitive data and all buffers are zeroed after use (FR-17, FR-20)

### Story 1.3: Sender Page Layout & Form

As a sender,
I want a clean, professional form to input my secret and protection options,
So that creating a secret feels fast and trustworthy.

**Acceptance Criteria:**

**Given** I navigate to the root URL `/`
**When** the page loads on desktop (1024px+)
**Then** I see a split layout: left gradient panel (indigo spectrum) with product tagline and feature checklist, right panel with a glass card form
**And** the glass card has `backdrop-filter: blur(20px)`, `background: rgba(255,255,255,0.75)`, `border-radius: 24px`

**Given** the sender form
**When** I inspect the form elements
**Then** I see a textarea for secret text (FR-1) with JetBrains Mono font and placeholder "Paste your secret here..."
**And** a PasswordRow with password input and "Random" generator button (FR-3, FR-4)
**And** an expiration dropdown with options: 1 hour, 24 hours (default), 7 days (FR-2)
**And** a "Include password in link" checkbox, unchecked by default (FR-5)
**And** a "Create Secret Link" primary button (solid indigo, full-width)

**Given** I click the "Random" password generator button
**When** a password is generated
**Then** the password field is populated with a strong random password
**And** the password is temporarily visible (toggle visibility)

**Given** I view the page on mobile (<768px)
**When** the page renders
**Then** the left info panel is hidden
**And** the form displays full-width with 16px horizontal padding
**And** all touch targets are minimum 44px

**Given** keyboard navigation
**When** I tab through the form
**Then** focus moves in logical order: textarea → password → random button → expiration → checkbox → submit
**And** visible focus indicators (2px indigo ring) appear on each element
**And** Enter key submits the form

### Story 1.4: Secret Creation Flow & Success

As a sender,
I want to generate a shareable link that's instantly copied to my clipboard,
So that I can share the secret faster than pasting into Slack.

**Acceptance Criteria:**

**Given** I have filled in the sender form with valid content and password
**When** I click "Create Secret Link"
**Then** the button shows a spinner and the form is disabled during the API call
**And** SecretApiClient sends a POST request to `/api/secrets` via Aspire service discovery

**Given** the API returns a successful response with `{ secretId, expiresAt }`
**When** the success state renders
**Then** the form transitions to a SuccessPanel showing: success icon, heading, generated link (in JetBrains Mono), "Copied to clipboard" emerald badge, and expiration metadata
**And** the link is automatically copied to clipboard via JS interop (`navigator.clipboard.writeText`) (FR-7)
**And** a toast notification confirms "Copied to clipboard" (auto-dismiss 3s)

**Given** the password-in-link checkbox was checked (FR-5)
**When** the link is generated
**Then** the password is embedded in the URL fragment (e.g., `/s/{id}#{password}`)
**And** the fragment is included in the copied link

**Given** the SuccessPanel is displayed
**When** I click "Copy Link Again"
**Then** the link is re-copied to clipboard with a brief confirmation flash

**Given** the SuccessPanel is displayed
**When** I click "Create Another Secret"
**Then** the form resets to its initial state with all fields cleared

**Given** the API returns an error
**When** the error state renders
**Then** inline error messages appear below the relevant fields
**And** the form remains editable for retry

## Epic 2: Secret Retrieval

A recipient can open a shared link, enter a password (or bypass via password-in-link), see a confirmation warning, view the decrypted secret, and copy it. The secret is destroyed immediately after viewing. Completes the end-to-end secret sharing loop.

### Story 2.1: Reveal API Endpoint

As a recipient,
I want the system to verify my password, decrypt the secret, and destroy it in one atomic operation,
So that the secret is only ever accessible once and never lingers after viewing.

**Acceptance Criteria:**

**Given** a valid secret exists in Cosmos DB with matching ID
**When** `POST /api/secrets/{id}/reveal` is called with the correct password
**Then** SecretService looks up the document, derives the key using PBKDF2 with the stored salt, decrypts using AES-256-GCM with the stored IV
**And** deletes the document from Cosmos DB immediately after successful decryption (FR-13)
**And** returns `{ content }` with the decrypted plaintext (FR-11)

**Given** a valid secret exists
**When** `POST /api/secrets/{id}/reveal` is called with an incorrect password
**Then** the system uses `CryptographicOperations.FixedTimeEquals()` for constant-time comparison to prevent timing attacks
**And** returns `{ message: "Incorrect password", remainingAttempts: N }` with appropriate status

**Given** decryption completes (success or failure)
**When** the operation finishes
**Then** all sensitive buffers (plaintext, password, derived key) are cleared using `CryptographicOperations.ZeroMemory()` (FR-19, FR-20)
**And** no secret content, password, or ciphertext is written to logs

**Given** a secret ID that does not exist in Cosmos DB
**When** `POST /api/secrets/{id}/reveal` is called
**Then** the system returns `404 { message: "This secret is not available" }`

### Story 2.2: Recipient Password Prompt & Verification

As a recipient,
I want to open a shared link and enter the password to access the secret,
So that I can retrieve the secret securely.

**Acceptance Criteria:**

**Given** I open a valid secret link at `/s/{id}`
**When** the page loads
**Then** I see a minimal, centered layout with a password input field and a "View Secret" primary button (FR-8)
**And** the page uses MinimalLayout (centered single-column, no split layout or glass effects)
**And** the password field has focus on load

**Given** I enter the correct password and click "View Secret"
**When** the API call is in progress
**Then** the button shows a spinner and the form is disabled

**Given** the API returns a successful response
**When** the result is received
**Then** the flow proceeds to the confirmation step (handled by Story 2.3)

**Given** I enter an incorrect password
**When** the API returns an error with remaining attempts
**Then** an inline error message "Incorrect password" appears below the password field in red text
**And** a neutral "X attempts remaining" message is displayed
**And** the password field is cleared and re-focused for retry

**Given** keyboard interaction
**When** I press Enter in the password field
**Then** the form submits (same as clicking "View Secret")

### Story 2.3: Confirmation, Reveal & Password-in-Link

As a recipient,
I want to see a confirmation before the secret is revealed, view and copy the content, and have password-in-link work seamlessly,
So that I don't accidentally burn a secret and can retrieve it with minimal friction.

**Acceptance Criteria:**

**Given** the correct password has been verified (from Story 2.2 or password-in-link)
**When** the confirmation step displays
**Then** I see a dialog with amber warning: "This secret will be permanently deleted after you view it. Continue?" (FR-10)
**And** the dialog has "Continue" (primary button) and "Cancel" (secondary button)
**And** the dialog uses `role="alertdialog"` with `aria-describedby` pointing to the warning text
**And** Escape key or backdrop click cancels

**Given** I click "Continue" on the confirmation dialog
**When** the secret is revealed
**Then** the SecretDisplay component shows the decrypted content in JetBrains Mono font with a copy button (FR-11, FR-12)
**And** a post-reveal amber banner states "This secret has been permanently deleted from the server"
**And** a CountdownTimer displays "This page will auto-clear in Xs" with `aria-live="polite"` and `role="timer"`

**Given** the CountdownTimer reaches zero
**When** auto-clear triggers
**Then** the page transitions to the DeadEndScreen

**Given** I click the copy button on SecretDisplay
**When** the content is copied
**Then** clipboard contains the secret text via JS interop
**And** the button shows a checkmark with "Copied" confirmation

**Given** I open a link with a password in the URL fragment (e.g., `/s/{id}#password`) (FR-9)
**When** the page loads
**Then** JS interop (`fragment.js`) extracts the password from `window.location.hash`
**And** the password prompt is skipped entirely
**And** the flow goes directly to the confirmation dialog

**Given** any secret link resolves to an unavailable secret (expired, viewed, nonexistent)
**When** the page loads or the API returns 404
**Then** the DeadEndScreen displays: neutral gray lock icon, "This secret is not available" heading, and a "Create a new secret" link to the root URL
**And** the message is identical regardless of the reason for unavailability

## Epic 3: Security Hardening & Lifecycle

System enforces brute-force protection (limited attempts, secret burned on exhaustion), auto-expires unclaimed secrets via Cosmos DB TTL, and returns identical generic responses for all failure states. Adds security headers middleware and sensitive logging exclusions.

### Story 3.1: Brute-Force Protection & Attempt Tracking

As a system operator,
I want the system to limit password attempts and destroy secrets after too many failures,
So that intercepted links cannot be brute-forced to reveal secrets.

**Acceptance Criteria:**

**Given** a secret is created with a `RemainingAttempts` field (default 3-5)
**When** an incorrect password is submitted via `POST /api/secrets/{id}/reveal`
**Then** the `RemainingAttempts` counter is decremented on the Cosmos DB document (FR-16)
**And** the response includes `{ message: "Incorrect password", remainingAttempts: N }`

**Given** a secret has 1 remaining attempt
**When** another incorrect password is submitted
**Then** the secret document is permanently deleted from Cosmos DB (FR-15)
**And** all subsequent requests for that ID return `404 { message: "This secret is not available" }`

**Given** a secret has been destroyed by attempt exhaustion
**When** the attacker makes another request
**Then** the response is identical to any other unavailable secret — no indication of burn vs expiry vs nonexistence

**Given** the recipient UI displays attempt feedback
**When** a failed attempt occurs and attempts remain
**Then** the "X attempts remaining" counter updates to reflect the new count
**And** the counter only appears after the first failed attempt (not on initial page load)

### Story 3.2: Auto-Expiration & Uniform Error Handling

As a system operator,
I want unclaimed secrets to auto-expire and all error states to be indistinguishable,
So that secrets have a limited shelf life and attackers gain zero information from error responses.

**Acceptance Criteria:**

**Given** a secret is created with an expiration period (1h, 24h, or 7d)
**When** the expiration time elapses
**Then** Cosmos DB native TTL automatically deletes the document (FR-14)
**And** no background job or application code is required for cleanup

**Given** the Cosmos DB TTL configuration
**When** a SecretDocument is stored
**Then** the `ExpiresAt` field maps to Cosmos DB's TTL property
**And** the TTL is calculated from the selected expiration period at creation time

**Given** any secret request fails for any reason (expired, already viewed, burned by attempts, nonexistent ID, any other error)
**When** the API responds
**Then** it returns `404 { message: "This secret is not available" }` (FR-21)
**And** no variation in status code, message, response time, or headers between different failure reasons (FR-22)

**Given** the SecurityHeadersMiddleware
**When** any response is sent
**Then** server identification headers are removed (Server, X-Powered-By)
**And** security headers are added (X-Content-Type-Options: nosniff, X-Frame-Options: DENY, Referrer-Policy: no-referrer)

**Given** the ExceptionHandlerMiddleware
**When** an unhandled exception occurs
**Then** it returns `500 { message: "An error occurred" }` with no stack traces or details

**Given** the logging configuration
**When** requests are made to `/api/secrets` endpoints
**Then** request bodies are excluded from logs (NFR-7, NFR-8)
**And** secret IDs, operation types, success/failure, and attempt counts ARE logged
**And** secret content, passwords, and ciphertext are NEVER logged

## Epic 4: Deployment & Operations

Administrator can deploy with a single command (Docker/Azure Container Apps), configure via environment variables, and have CI/CD pipeline and monitoring in place.

### Story 4.1: Docker & Azure Deployment

As an administrator,
I want to deploy the application with a single command and configure it via environment variables,
So that I can get the app running in production without manual setup steps.

**Acceptance Criteria:**

**Given** the project repository
**When** I run `docker-compose up` in the project root
**Then** the application starts with all services (API, Web, database) running in containers (FR-23)
**And** the app is accessible on the configured port

**Given** environment variables for configuration
**When** I set `PORT`, database connection string, and other operational settings
**Then** the application uses those values without code changes (FR-24)
**And** default values work for a zero-config first deployment

**Given** the `azure.yaml` manifest and Aspire app model
**When** I run `azd up`
**Then** Azure Container Apps, Cosmos DB, and supporting infrastructure are provisioned via generated Bicep templates
**And** the application deploys and is publicly accessible

**Given** the `azure-pipelines.yml` pipeline definition
**When** code is pushed to the main branch
**Then** the CI/CD pipeline builds, tests, and deploys to Azure Container Apps
**And** the pipeline uses `azd` for deployment

**Given** Azure Monitor integration via Aspire's OpenTelemetry
**When** the application runs in production
**Then** telemetry (logs, traces, metrics) is collected by Azure Monitor
**And** log filtering excludes request bodies on `/api/secrets` endpoints
**And** Azure Key Vault is used for production connection strings and app secrets

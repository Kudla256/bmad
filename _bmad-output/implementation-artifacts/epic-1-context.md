# Epic 1 Context: Secret Creation

<!-- Compiled from planning artifacts. Edit freely. Regenerate with compile-epic-context if planning docs change. -->

## Goal

Deliver the sender half of the product: paste a secret, set or generate a password, pick an expiration, optionally embed the password in the link, and get back a unique shareable link already on the clipboard. As the first epic in a greenfield repository it also lays the foundation every later epic builds on — scaffolding, the single-container data layer, the encryption service, the create endpoint, and the styling pipeline. The flow competes with pasting a credential into a chat DM, so a three-interaction happy path is a requirement, not polish.

## Stories

- Story 1.1: Project Foundation & Data Layer
- Story 1.2: Encryption Service & Secret Creation API
- Story 1.3: Sender Page Layout & Form
- Story 1.4: Secret Creation Flow & Success

## Requirements & Constraints

- Text secrets only — multi-line, with newlines, tabs and non-ASCII round-tripping byte-identically. No files or binary content.
- Exactly three expiration options (1h / 24h / 7d), 24h preselected, no free-form durations. Stored expiry is creation time plus the chosen period.
- Secret text and password are the only required fields; everything else has a working default. An empty either is rejected *before* any record is created.
- The random generator draws from a cryptographically secure source, never repeats consecutively, shows the value temporarily so the sender can share it, and leaves it editable.
- Password-in-link is opt-in, off by default — the secure path is the default path. When on, the password appears only after the `#`, never in a path, query string, or referrer.
- Identifiers must be unguessable, collision-free, and reveal nothing about creation time, expiration, or sender. Unguessability is v1's only enumeration defence; rate limiting is deferred.
- The link reaches the clipboard on success with no extra click *and* stays visible with a manual copy control — a browser may refuse a clipboard write outside a user gesture, so the fallback is required, not optional.
- Zero-knowledge is the central claim: the stored record must contain nothing from which the secret is derivable without the password, identical content with an identical password must still yield different ciphertext, and the password never reaches storage, logs, or the response body.
- Encrypt-plus-store under 500 ms server-side, page load under 1 s, no blocking work on the client thread, minimal JS payload.
- Every request is anonymous. No accounts, sessions, or cookies, and nothing correlating two requests to one person.
- Two open decisions must be settled here: the attempt-budget number (a 3–5 range today) and the maximum secret size, which bounds the request body and the encryption buffers.

## Technical Decisions

- Greenfield start from the Aspire starter template: orchestration host, shared service defaults, API service, Blazor web, one test project. Dependencies flow one way — the host wires everything, web talks to the API over HTTP, both depend on service defaults, which depends on nothing in the product. Tests stay in their own project mirroring the source structure, never co-located. Blazor InteractiveServer is the single render mode: no WebAssembly, no per-page divergence.
- Encryption happens only inside the API service, keyed only by the sender-supplied password. The frontend never encrypts, derives keys, holds a database reference, or sees a connection string; it reaches data exclusively through its API client.
- Cryptography comes from the platform library only — AES-256-GCM plus PBKDF2 over the password and a per-secret random salt. No third-party crypto package; recomposing the primitives differently is an architecture change, not an implementation choice.
- Sensitive data moves as byte spans and is explicitly zeroed after use on success *and* failure paths; managed strings are avoided for passwords and plaintext wherever the API allows.
- Persistence is one container, one document type, partition key on the id, point-read by id only — no queries, scans, or secondary indexes. The document is written once, with the remaining-attempts counter as the sole mutable field, and the expiry field maps to the store's native TTL set at creation. No background cleanup job.
- Secret operations are POST only, with no GET for secret data and no secret material in any URL. The create endpoint is the only one permitted to return a 400 with field-level errors.
- Conventions: camelCase JSON, ISO 8601 UTC dates, nulls omitted, no response envelope, lowercase plural routes, errors as a single message field. Services are injected rather than constructed inline and stay one-per-concern; the web client resolves the API through service discovery, with no literal URLs or per-environment host switching in code.
- JS interop is confined to two files with two purposes — clipboard access and fragment reading. This epic needs the clipboard one only. No other JS, no npm runtime dependency.
- Styling is Tailwind v4 through a CLI build target that generates the app stylesheet, with design tokens in the Tailwind theme. Tailwind utilities only — no custom CSS class vocabulary and no component library of any kind.
- Logging is allow-list, not deny-list. Loggable: secret id, operation, outcome, attempt count, expiration. Never loggable: content, passwords, ciphertext, or request bodies on the secret endpoints — including inside exception handlers.

## UX & Interaction Patterns

- The sender page is the only surface with the glass treatment: an indigo-gradient info panel left, a glass card holding the form right, and no header, sidebar, footer or navigation anywhere. Glass sits behind a feature query with a solid-white fallback and an identical layout either way; below 768px the info panel is removed entirely and the form goes full-width in gutters.
- Semantic type split: the interface face for anything the product says, the mono face for the user's own data (secret textarea, generated link). Indigo marks exactly one action per screen; emerald is only completed confirmation, red only validation and wrong passwords, amber only irreversible destruction — so amber never appears here. No state is conveyed by color alone.
- One primary button per surface; it swaps its label for a spinner and disables its form in flight. No page-level loader, skeleton, or overlay.
- Validation fires on submit, not blur. Errors render inline under their field, persist until corrected, move focus there, and always carry text. Microcopy is factual short declaratives with no exclamation marks, no security or strength claims, and no onboarding.
- Success is an in-place state change plus an auto-dismissing toast — never a dialog, and nothing pushes history. The form becomes a success panel with the link in mono, a copied badge, metadata (expiration, password required, one-time view), and two onward actions: copy again, create another. Focus moves there on generation and the outcome announces assertively.
- Tab order is textarea → password → generator → expiration → checkbox → submit; Enter submits from any field. The password field is masked by default with a visibility toggle. Use a native select for expiration precisely because the platform supplies its keyboard and screen-reader behavior — a custom listbox is acceptable only if it reimplements arrow keys, type-ahead, active-descendant semantics and focus return, and is tested against them.
- WCAG 2.1 AA floor, hand-written since there is no primitive library: real HTML with ARIA only where the platform has no equivalent, a real label on every field, visible focus everywhere, a focus-revealed skip link, 44px touch targets, rem sizing, and reduced-motion switching transitions off instantly rather than shortening them.

## Cross-Story Dependencies

- Story 1.1 gates the rest of the epic and every later epic. Once it lands, 1.2 (API) and 1.3 (form) can proceed in parallel; 1.4 needs both.
- The key-derivation parameters and salt/IV handling chosen in 1.2 are consumed unchanged by Epic 2's reveal path — pick them as a contract, not an implementation detail. Story 1.1's document must already carry the remaining-attempts and expiry fields, even though Epic 3 implements the behaviors built on them.
- The clipboard interop file added in 1.4 is shared with Epic 2's copy action; fragment reading belongs to Epic 2, and the password-in-link format 1.4 emits is exactly what Epic 2 must parse.
- Global security middleware and endpoint logging exclusions arrive in Epic 3, but their rules already constrain what 1.2 and 1.4 may log or return. Environment-variable configuration and production secret storage land in Epic 4, so 1.1 should leave port and datastore settings overridable rather than hardcoded.

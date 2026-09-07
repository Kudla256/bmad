---
title: Securer
status: final
created: 2026-02-13
updated: 2026-09-04
---

# PRD: Securer

*Working title "Secret Drop" appears in early artifacts; "Securer" is the name of record.*

## 0. Document Purpose

This PRD is the requirements contract for Securer, written for the solo developer building it and for the downstream BMad workflows that consume it — `bmad-architecture`, `bmad-ux`, and `bmad-create-epics-and-stories`. It is Glossary-anchored: §3 defines every domain noun once, and §2, §4 and §9 use those terms verbatim with no synonyms. Features are grouped in §4 with their functional requirements nested and numbered globally as `FR-1`…`FR-24`, so epics and stories can reference a stable ID even if features are reorganised. User journeys carry global IDs `UJ-1`…`UJ-6` and are referenced inline from the FRs they realise; success metrics carry `SM-` IDs and name the FRs they validate. Inferences made without confirmation are tagged inline as `[ASSUMPTION: …]` and collected in §11.

It builds on two upstream artifacts and does not duplicate them: the product brief at `../../briefs/brief-Securer-2026-02-12/brief.md` (with persona depth, deferral reasoning and the risk register in its `addendum.md`), and the brainstorming session at `../../../brainstorming/brainstorm-secret-drop-2026-02-12/`. UX decisions live downstream in `../../ux-designs/ux-Securer-2026-02-13/` (`DESIGN.md`, `EXPERIENCE.md`) and technical decisions in `../../architecture/architecture-Securer-2026-02-13/ARCHITECTURE-SPINE.md`; where those documents and this one disagree on a requirement, this PRD wins.

## 1. Vision

Securer is a self-hosted, zero-knowledge, one-time secret sharing web application. A Sender pastes a Secret, picks an Expiration, sets a Password, and gets a Link that is already on their clipboard. The server encrypts the Secret in memory using the Sender's Password, persists only the Ciphertext, and discards the plaintext and Password before it responds. A Recipient opens the Link, supplies the Password, confirms, and sees the Secret exactly once — the read itself destroys the record.

It exists because the fast way to share a credential is also the worst way. Slack, email and pastebins persist and index whatever passes through them, so a credential shared once stays findable for months, and every person with channel access becomes an attack surface. Secure alternatives lose to that convenience: they demand accounts, ship dashboards, or ask a team to hand its most sensitive data to a third party.

Securer's bet is that the secure path can be the fast path. The entire product is one form on one page, with no accounts, no navigation, no API and no admin surface, deployed by a single `docker-compose up` and open source so a team can read the code holding its credentials. If sharing a Secret through it beats pasting into a DM, adoption needs no mandate.

## 2. Target User

### 2.1 Jobs To Be Done

- **Move a credential to one specific person without leaving a copy anywhere.** Functional; the core job. Today this is a Slack DM the Sender knows they should not send.
- **Hand credentials to someone who cannot be walked through a security tool.** Functional and social; a team lead onboarding a new hire cannot spend the first morning explaining encryption.
- **Stop feeling responsible for a credential after sharing it.** Emotional; the persistent copy in chat history is a low-grade obligation that never resolves.
- **Receive something sensitive without being asked to sign up for anything.** Functional; the Recipient did not choose this tool and owes it nothing.
- **Run a security tool the team can audit rather than trust.** Social and contextual; self-hosting and open source are the substitutes for institutional trust.
- **Deploy it once and never operate it.** Functional; the Instance Administrator's whole job is that there is no ongoing job.

### 2.2 Non-Users (v1)

- **Organisations wanting managed SaaS.** There is no hosted instance in v1; a team that will not self-host is not served.
- **Teams needing audit trails or compliance evidence.** Securer deliberately records nothing about who shared what, which is incompatible with an audit requirement.
- **Anyone sharing files or binaries.** Text Secrets only.
- **Programmatic consumers.** There is no public API surface in v1; automation is not a supported path.

### 2.3 Key User Journeys

*Named-persona narratives the product enables. FRs reference these by ID inline; the UX spec mirrors the same IDs.*

- **UJ-1. Alex shares a staging API key without leaving it in Slack.**
  - **Persona + context:** Alex, a backend developer, has just rotated the staging API key and his teammate Maria needs it to configure her local environment. He would normally paste it into a DM and feel mildly bad about it.
  - **Entry state:** unauthenticated — there is no account. Desktop browser, team's self-hosted Instance.
  - **Path:** opens the Instance, pastes the key into the form, picks `24h` from the Expiration dropdown because Maria is online today, types a Password he will say out loud on standup, leaves password-in-link unchecked because this is a real credential, clicks Generate.
  - **Climax:** the Link is on his clipboard before he reaches for it; he pastes it into Slack with "password on the call." About ten seconds, start to finish.
  - **Resolution:** the credential is not in Slack — only a Link that is worthless without the Password and dies after one view.

- **UJ-2. Maria retrieves the Secret and converts into a Sender.**
  - **Persona + context:** Maria, the teammate, sees an unfamiliar Link in a DM.
  - **Entry state:** unauthenticated, no prior knowledge of the tool, following a Link from chat.
  - **Path:** clicks the Link, lands on a page with a Password field and a submit button and nothing else, hears the Password on standup, submits it, reads the confirmation warning that viewing is permanent, clicks Continue.
  - **Climax:** the API key appears; she copies it into her `.env`. Refreshing shows the generic dead-end screen — the Secret is genuinely gone.
  - **Resolution:** no signup happened, nothing was installed, and she now knows what to use next time she has to share something sensitive.
  - **Edge case:** if she never got the Password, the Link is inert to her and to anyone else who obtained it.

- **UJ-3. Dana onboards a new hire with three credentials and no second channel.**
  - **Persona + context:** Dana, a team lead, is onboarding Jake on his first day. He needs the database password, the CI/CD token and the VPN credentials, and he is not yet technical enough for a per-credential password handoff to survive.
  - **Entry state:** unauthenticated, desktop, composing a welcome email.
  - **Path:** for each credential she creates a Secret with Expiration `1h` because Jake is online now, generates a random Password rather than inventing one, checks "include password in link" so the Password rides in the URL fragment, and drops the three Links into the email.
  - **Climax:** Jake clicks each Link, is taken straight to the confirmation screen with no Password prompt, views and copies each credential.
  - **Resolution:** three credentials delivered, all three Links dead within minutes, and Jake never had to learn anything about the tool.
  - **Edge case:** if Jake forwards a Link before opening it, whoever opens it first consumes the Secret — password-in-link mode trades the second factor for convenience, and Dana chose that trade knowingly.

- **UJ-4. Sam hits a Link that has already expired.**
  - **Persona + context:** Sam, a project manager, finds a two-day-old Slack message with a Link for a shared account password.
  - **Entry state:** unauthenticated, following a stale Link.
  - **Path:** clicks the Link and gets a single screen: "This secret is not available."
  - **Climax:** there is nothing to interpret — no hint whether it expired, was viewed, was burned, or never existed.
  - **Resolution:** she messages the Sender for a fresh Link. The dead end is unhelpful by design and costs one round trip.

- **UJ-5. An attacker with an intercepted Link burns it instead of opening it.**
  - **Persona + context:** someone who obtained a Link but not the Password.
  - **Entry state:** unauthenticated, holding a valid Link.
  - **Path:** tries `password123`, then `company2024`; an attempts-remaining counter appears and decrements.
  - **Climax:** the attempt budget is exhausted and the Secret is destroyed. The screen shows the same generic dead-end message as every other failure.
  - **Resolution:** the attacker cannot tell whether they burned a live Secret or found an already-dead Link, and the legitimate Recipient learns the Link no longer works — which is the correct signal.

- **UJ-6. Chris deploys the Instance and forgets about it.**
  - **Persona + context:** Chris, a senior developer, decides the team should stop pasting credentials into Slack.
  - **Entry state:** shell on a host with Docker, behind the company reverse proxy.
  - **Path:** clones the repo, sets port and database connection, runs `docker-compose up`.
  - **Climax:** the Instance is live. There is no admin dashboard to configure, no users to provision, no background worker to supervise.
  - **Resolution:** it runs. Operating it is not a task on anyone's list.

## 3. Glossary

*Downstream workflows and readers must use these terms exactly.*

- **Secret** — the plaintext text content a Sender wants to transfer. Text only; never persisted. Exactly one Secret per Link.
- **Ciphertext** — the encrypted form of a Secret, produced server-side from the Secret and the Password. The only representation that reaches storage.
- **Sender** — the person who creates a Secret. Unauthenticated; has no identity in the system.
- **Recipient** — the person who opens a Link to retrieve a Secret. Unauthenticated; not known to the system in advance and not distinguished from anyone else holding the Link.
- **Password** — the sender-supplied string used to encrypt and later decrypt a Secret. Never stored, never logged, and not recoverable. May be typed by the Sender or produced by the random password generator.
- **Link** — the unique shareable URL that addresses one Secret. Carries the Secret's identifier and, in password-in-link mode, the Password in its URL fragment.
- **Password-in-link mode** — the optional mode in which the Password is embedded in the Link's URL fragment so the Recipient is not prompted for it. Trades the second access factor for convenience.
- **Expiration** — the unclaimed shelf life of a Secret, chosen by the Sender from 1 hour, 24 hours or 7 days. Governs only how long an unviewed Secret survives; it is not an alternative to one-time view.
- **Reveal** — the act of decrypting and displaying a Secret to a Recipient. Destructive: a Reveal deletes the Secret's record.
- **Attempt budget** — the finite number of failed Password submissions a Secret tolerates before it is burned. Tracked per Secret; the only mutable field on an otherwise write-once record.
- **Burn** — destruction of a Secret triggered by exhausting the Attempt budget. One of three destruction triggers, alongside Reveal and Expiration.
- **Dead-end screen** — the single generic screen shown for every unavailable-Secret condition. Identical text regardless of cause.
- **Instance** — one self-hosted deployment of Securer. Owns its own storage; Instances share nothing.
- **Instance Administrator** — the person who deploys and maintains an Instance. Has no in-product surface.

## 4. Features

### 4.1 Secret Creation

**Description:** The Sender's entire experience is one form on one page: a text area for the Secret, an Expiration dropdown, a Password field with a generator beside it, a password-in-link checkbox, and a generate button. Submitting encrypts the Secret server-side and returns a Link that is placed on the clipboard without the Sender asking. Realises UJ-1, UJ-3. The design constraint that governs this feature is speed — the whole flow must beat pasting into a chat DM, which is roughly ten seconds.

**Functional Requirements:**

#### FR-1: Enter Secret content

Sender can input Secret text content into a text field. Realises UJ-1.

**Consequences (testable):**
- The field accepts multi-line text including newlines, tabs and non-ASCII characters, and round-trips them byte-identically through Reveal.
- Submitting with an empty Secret field is rejected before any record is created.

**Out of Scope:**
- File or binary content of any kind.

#### FR-2: Select an Expiration

Sender can select an Expiration of 1 hour, 24 hours or 7 days. Realises UJ-1, UJ-3.

**Consequences (testable):**
- Exactly these three options are offered; no free-form duration is accepted.
- The stored expiry instant equals creation time plus the selected period.
- A Secret whose Expiration has not elapsed and which has not been Revealed or Burned is retrievable.

#### FR-3: Set a Password

Sender can provide a Password required to access the Secret. Realises UJ-1.

**Consequences (testable):**
- Submitting with an empty Password field is rejected before any record is created.
- The Password is never written to storage, logs, or the response body.

#### FR-4: Generate a random Password

Sender can generate a random Password through the UI instead of typing one. Realises UJ-3.

**Consequences (testable):**
- Activating the generator populates the Password field with a value drawn from a cryptographically secure random source.
- Two consecutive generations do not produce the same value.
- A generated Password is editable afterwards, exactly like a typed one.

#### FR-5: Include the Password in the Link

Sender can optionally include the Password in the generated Link via the URL fragment. Realises UJ-3.

**Consequences (testable):**
- When the option is checked, the Password appears only after the `#` in the Link and never in the path or query string.
- When it is unchecked, the Link contains no Password material in any component.
- The Recipient flow for such a Link skips the Password prompt (see FR-9).

#### FR-6: Generate a unique Link

Sender can generate a unique, shareable Link for the created Secret. Realises UJ-1, UJ-3.

**Consequences (testable):**
- Each created Secret yields an identifier that is unguessable and does not collide with any other Secret on the Instance.
- The identifier reveals nothing about creation time, Expiration, or Sender.
- The Link resolves to that Secret and no other.

#### FR-7: Auto-copy the Link

Sender receives the generated Link automatically copied to their clipboard. Realises UJ-1.

**Consequences (testable):**
- On successful creation the Link is written to the clipboard without further interaction.
- The Link remains visible on screen with a manual copy affordance, so a clipboard write that is blocked or unavailable does not strand the Sender. `[ASSUMPTION: browsers may refuse the clipboard write outside a user-gesture context, so a visible fallback is required rather than optional.]`

### 4.2 Secret Retrieval

**Description:** The Recipient arrives from a Link with no context and no account. They are asked for the Password, warned that viewing is permanent, and shown the Secret once. Password-in-link mode collapses the prompt but keeps the warning — the confirmation step is the only thing standing between a click and an irreversible Reveal, so it survives every convenience path. Realises UJ-2, UJ-3.

**Functional Requirements:**

#### FR-8: Prompt for the Password

Recipient can open a Link and be prompted for a Password. Realises UJ-2.

**Consequences (testable):**
- The prompt page exposes nothing about the Secret — not its size, its Expiration, nor that it exists at all beyond rendering the prompt.
- Submitting an incorrect Password decrements the Attempt budget (see FR-16) and does not Reveal.

#### FR-9: Skip the prompt in password-in-link mode

Recipient opening a password-in-link Link can access the Secret without a separate Password prompt. Realises UJ-3.

**Consequences (testable):**
- The Password is read from the URL fragment client-side and never appears in any request path, query string, or referrer header.
- The Recipient lands on the confirmation step (FR-10), not directly on the revealed Secret.
- A password-in-link Link carrying a wrong Password consumes Attempt budget exactly as a typed wrong Password does.

#### FR-10: Confirm before Reveal

Recipient can view a confirmation warning before the Secret is revealed. Realises UJ-2, UJ-3.

**Consequences (testable):**
- The warning states that viewing permanently destroys the Secret.
- The Secret is not decrypted for display, and the record is not destroyed, until the Recipient confirms.
- Abandoning the page at the confirmation step leaves the Secret retrievable.

#### FR-11: Reveal the Secret

Recipient can view the decrypted Secret after supplying the correct Password and confirming. Realises UJ-2, UJ-3.

**Consequences (testable):**
- The displayed content is byte-identical to what the Sender entered.
- The Secret's record is destroyed as part of serving the Reveal (see FR-13).
- Reloading the page or re-opening the Link after a Reveal produces the Dead-end screen.

#### FR-12: Copy the revealed Secret

Recipient can copy the revealed Secret content. Realises UJ-2.

**Consequences (testable):**
- A copy affordance places the exact Secret content on the clipboard.
- The content also remains selectable, so a blocked clipboard write does not strand the Recipient.

### 4.3 Secret Lifecycle

**Description:** A Secret has exactly three ways to die — Reveal, Expiration, and Burn — and all three must leave nothing recoverable. Retrieval is a destructive read: serving the Secret and deleting it are the same operation, not two steps that could be interrupted between. The Attempt budget is the only field on the record that ever changes after creation. Realises UJ-2, UJ-4, UJ-5.

**Functional Requirements:**

#### FR-13: Destroy on Reveal

System destroys the Secret immediately after it has been viewed once. Realises UJ-2.

**Consequences (testable):**
- After a successful Reveal, no query by the Secret's identifier returns a record.
- A second concurrent request for the same Secret cannot also receive a Reveal — at most one Reveal succeeds per Secret.

#### FR-14: Destroy on Expiration

System destroys unclaimed Secrets automatically when their Expiration elapses. Realises UJ-4.

**Consequences (testable):**
- A Secret whose Expiration has passed is never Revealed, even with the correct Password.
- Expired records are removed from storage rather than merely hidden, so storage grows only with active Secrets.

#### FR-15: Burn on exhausted Attempt budget

System destroys the Secret after the maximum number of failed Password attempts is reached. Realises UJ-5.

**Consequences (testable):**
- The Secret is destroyed on the failing attempt that exhausts the budget, not on the next request.
- After a Burn, the correct Password no longer Reveals the Secret.

#### FR-16: Track remaining attempts

System tracks the remaining Attempt budget per Secret. Realises UJ-5.

**Consequences (testable):**
- The remaining count decrements by exactly one per failed Password submission and is never incremented.
- The Recipient is shown how many attempts remain.
- Concurrent failed attempts cannot decrement past zero or bypass the Burn.

**Feature-specific NFRs:**
- Destruction must be durable, not deferred: once a destruction trigger fires, no subsequent request may observe the Secret, even if cleanup of the underlying storage is asynchronous.

### 4.4 Encryption and Zero-Knowledge Handling

**Description:** The zero-knowledge property is the product's central claim, and it is a claim about what the server *cannot* do: without the Sender's Password, an Instance operator holding the full database cannot recover any Secret. Plaintext exists only in memory, only during the encrypt and decrypt operations. Realises UJ-1, UJ-2, UJ-6.

**Functional Requirements:**

#### FR-17: Encrypt server-side with the Sender's Password

System encrypts Secret content server-side using the Sender-provided Password before storing. Realises UJ-1.

**Consequences (testable):**
- The stored record contains no representation from which the Secret can be derived without the Password.
- Two Secrets with identical content and identical Passwords produce different Ciphertext.

#### FR-18: Store Ciphertext only

System stores only Ciphertext — plaintext never persists in storage. Realises UJ-1.

**Consequences (testable):**
- No field of the stored record contains Secret plaintext or the Password in any form.
- A dump of the datastore yields nothing readable without per-Secret Passwords.

#### FR-19: Decrypt in memory at retrieval

System decrypts Secret content in memory only, at the moment of authorised retrieval. Realises UJ-2.

**Consequences (testable):**
- Decrypted content is never written to storage, cache, or any file.
- Decryption occurs only after the correct Password is supplied and the Recipient confirms.

#### FR-20: Discard plaintext and Password after use

System discards plaintext and Password from memory immediately after encryption or decryption. Realises UJ-1, UJ-2.

**Consequences (testable):**
- No request-scoped state retains the Password or plaintext after the response is produced.
- Neither value appears in any log, trace, metric, or error report, including on the failure paths.

### 4.5 Uniform Failure Handling

**Description:** Every way a Secret can be unavailable produces one screen with one message. This is deliberately unhelpful: distinguishing "expired" from "already viewed" from "never existed" hands an attacker an oracle, and the cost of the ambiguity is one message to the Sender asking for a fresh Link. Realises UJ-4, UJ-5.

**Functional Requirements:**

#### FR-21: One generic message for all failure states

System displays an identical generic message for all failure states — expired, viewed, burned, nonexistent. Realises UJ-4, UJ-5.

**Consequences (testable):**
- The rendered text, status code and response shape are identical across all four conditions.
- Response timing does not systematically differ by cause in a way that distinguishes them.

#### FR-22: Reveal nothing about Secret history

System reveals no information about whether a Secret ever existed, was already viewed, or was destroyed. Realises UJ-5.

**Consequences (testable):**
- A request for a well-formed but never-issued identifier is indistinguishable from a request for a consumed one.
- No header, redirect, or error body differentiates the cases.

### 4.6 Deployment and Operation

**Description:** The Instance Administrator's success condition is the absence of work. One command brings the Instance up; port and database connection are the only knobs. There is no admin UI, no user management, and no scheduled job an operator must supervise. Realises UJ-6.

**Functional Requirements:**

#### FR-23: Single-command deployment

Instance Administrator can deploy the application using a single `docker-compose up` command. Realises UJ-6.

**Consequences (testable):**
- On a machine with Docker and no prior Securer state, `docker-compose up` yields a serving Instance with no additional manual steps.
- No step requires editing a file inside the image or running a follow-up migration command by hand.

#### FR-24: Environment-variable configuration

Instance Administrator can configure basic settings — port and database — via environment variables. Realises UJ-6.

**Consequences (testable):**
- Changing the port variable changes the listening port with no code or image change.
- Changing the database variable points the Instance at a different datastore with no code or image change.
- Every configuration value has a working default or fails fast at startup with a message naming the missing variable.

## 5. Cross-Cutting NFRs

*System-wide quality attributes not tied to a single feature. Feature-specific NFRs stay nested under their feature in §4.*

**Performance**
- Page load completes in under 1 second on a standard broadband connection.
- Secret creation — encryption plus storage — completes in under 500 ms server-side.
- Secret retrieval — decryption plus delivery — completes in under 500 ms server-side.
- The UI stays responsive during Link generation; no blocking work on the client thread.
- The JavaScript payload stays minimal; the client is a thin form, not an application.

**Security**
- All client-server communication is over TLS. There is no plaintext transport path.
- Nothing logs Secret content, Passwords, or decrypted data at any point, on any path, including error and exception handlers.
- Request bodies on the Secret creation and retrieval endpoints are never logged.
- Passwords are never written to disk, logs, or any persistent store.
- A failed Password submission reveals nothing about whether the Secret exists (see FR-21, FR-22).
- URL fragments used by password-in-link mode are never transmitted to the server by the browser.

**Reliability**
- Destruction is authoritative: once a destruction trigger has fired for a Secret, no later request may observe it, regardless of storage cleanup timing.
- A Reveal either delivers the Secret and destroys the record, or does neither. It must not be possible to display a Secret that survives.

**Scalability**
- Request handling is stateless; any Instance replica can serve any request.
- Storage access for a Secret is a single key lookup by identifier — O(1), no scans.
- Horizontal scaling is achieved by adding replicas behind a load balancer, with no coordination between them.
- Storage grows only with active, unexpired Secrets; expired records are removed rather than accumulated.
- A single small VPS (1-2 GB RAM) handles typical single-team usage.

**Observability**
- Health and telemetry surfaces exist for operating an Instance, and carry no Secret-derived data — no content, no Passwords, no identifiers that would let an operator correlate Secret activity.

## 6. Platform and Constraints

**Application type.** Single-page web application with three surfaces: the Sender form, the Recipient flow (Password prompt → confirmation → Reveal), and the Dead-end screen.

**Browser support.** Modern browsers only — the last two versions of Chrome, Firefox, Safari and Edge. This unlocks the Clipboard API, modern CSS layout and ES2020+ without polyfill work.

**Responsive behaviour.** Desktop-first, but Recipients routinely open Links on phones (UJ-3, UJ-4), so every surface must remain usable at narrow widths.

**Architecture constraint.** Pure request-response. No WebSockets, polling, or server-sent events. No SEO, server-side rendering, or meta-tag requirements. Static assets are served by the same container as the backend.

**Cryptography constraint.** Established platform cryptographic libraries only. Designing a bespoke scheme is prohibited (see §7).

**Data constraint.** Text Secrets only, with a single maximum size to be fixed (see §10, question 2).

## 7. Non-Goals (Explicit)

- **Securer is not a SaaS product.** There is no hosted Instance in v1 and no account system to build one on. A public showcase Instance is a post-v1 possibility, not a v1 hedge.
- **Securer does not know who its users are.** No accounts, no identity, no sessions. Anything that would require knowing who sent or received a Secret is out.
- **Securer is not an audit tool.** It records nothing about Secret activity by design. Audit logs are not a deferred feature; they contradict the product.
- **Securer is not a file-sharing tool.** Text Secrets only.
- **Securer has no programmatic surface.** No public API, no webhooks, no integrations in v1.
- **Securer has no operator surface.** No admin dashboard, no analytics UI, no moderation tooling.
- **Securer does not roll its own cryptography.** It composes established platform primitives; inventing a scheme is prohibited, not merely discouraged.
- **Securer does not become a team collaboration product.** Team and org features arrive only if real demand appears after v1, never speculatively.

## 8. MVP Scope

### 8.1 In Scope

- Single-page Sender form: Secret text input, Expiration dropdown (1h / 24h / 7d), Password field, random Password generator, password-in-link checkbox, generate button.
- Server-side zero-knowledge encryption; plaintext never persisted.
- One-time Reveal, with Expiration governing the unclaimed shelf life.
- Password-required retrieval with a finite Attempt budget; Burn on exhaustion.
- Password-in-link mode via the URL fragment.
- Auto-copy of the generated Link to the clipboard, with a visible fallback.
- Confirmation step before every Reveal.
- Uniform Dead-end screen for every failure state.
- Expiration-based automatic deletion of unclaimed Secrets.
- Docker / docker-compose single-command deployment.

### 8.2 Out of Scope for MVP

- **Public hosted Instance** — self-hosted only; a public Instance would require rate limiting and abuse handling that v1 does not have.
- **User accounts or authentication** — the friction they add is the reason competing tools lose to Slack.
- **API access** — deferred to v3; no v1 use case requires automation.
- **File or attachment sharing** — deferred to v3; text-only keeps the encryption path and the UI simple.
- **Admin dashboard or analytics UI** — contradicts the zero-knowledge posture and the deploy-and-forget promise.
- **Rate limiting and abuse prevention** — deferred to v2, and a prerequisite for any public Instance.
- **Audit logs** — a non-goal, not a deferral (see §7).
- **QR code for the Link** — deferred to v1.5. `[NOTE FOR PM: genuinely useful for desktop-to-phone handoff; first candidate to pull forward if the timeline allows.]`
- **Syntax highlighting on the Reveal view** — deferred to v1.5; polish on a path that already works.
- **Countdown timer with auto-clear on the Reveal view** — deferred to v1.5; cosmetic, since the backend has already destroyed the record.
- **About page explaining the security model** — deferred to v1.5. `[NOTE FOR PM: load-bearing for Recipient trust in UJ-2 and arguably the product's best marketing; deferred only because it is content work.]`
- **"Copy with message" pre-written sharing text** — deferred to v1.5; small and cheap, second candidate to pull forward.
- **Browser extension and CLI** — deferred to v2.
- **Team / org features** — deferred to v3 and gated on demonstrated demand.

## 9. Success Metrics

**Primary**

- **SM-1: Time to share.** A first-time Sender completes Secret creation and gets a Link in under 15 seconds, measured from page load to clipboard. Validates FR-1, FR-2, FR-3, FR-6, FR-7.
- **SM-2: End-to-end completion.** Share of Secrets that reach a Reveal before Expiration or Burn. A low rate means Links are being created but not successfully consumed. Validates FR-8, FR-10, FR-11.
- **SM-3: Zero plaintext exposure.** Count of incidents in which Secret plaintext or a Password was found in storage, logs or telemetry. Target: zero, verifiable by code audit. Validates FR-17, FR-18, FR-19, FR-20.

**Secondary**

- **SM-4: Adoption trend.** Secrets created per week on an Instance, growing organically after deployment without a mandate. Validates FR-1 through FR-7 as a set.
- **SM-5: First-try deployment.** An Instance Administrator reaches a serving Instance on the first `docker-compose up` on a clean machine. Validates FR-23, FR-24.
- **SM-6: Recipient-to-Sender conversion.** A Recipient who retrieved a Secret later creates one. This is the growth mechanism, since there is no marketing surface. Validates FR-8 through FR-12. `[ASSUMPTION: measuring this requires distinguishing Recipients from Senders, which the zero-knowledge posture prevents — it may only ever be observable qualitatively.]`

**Counter-metrics (do not optimise)**

- **SM-C1: Secret longevity.** Average time a Secret survives before destruction. Counterbalances SM-2: pushing completion rates up by lengthening Expirations directly worsens the exposure window the product exists to close.
- **SM-C2: Password-in-link share.** Proportion of Secrets created with password-in-link mode. Counterbalances SM-1: the fastest possible share is the one that drops the second factor, and optimising for speed alone would make the weakest mode the default.
- **SM-C3: Retrieval retry rate.** Failed Password submissions per Secret. Counterbalances SM-3: an Attempt budget so tight that legitimate Recipients Burn Secrets they were entitled to read trades usability for a security gain that is already covered.

## 10. Open Questions

1. What exactly is the Attempt budget — 3, 4 or 5? The brief and journeys say "3-5"; the value must be fixed before FR-15 and FR-16 are implementable, and it is the direct lever on SM-C3.
2. What is the maximum Secret size? The brief carries a 10–50 KB working range; a single number is needed to bound the request body and the encryption path.
3. Should the Sender be able to choose a shorter Expiration than 1 hour for the onboarding case in UJ-3, where Links are expected to live for minutes?
4. Is the Attempt budget per Secret only, or is there any per-Instance protection against enumerating identifiers? Rate limiting is deferred to v2, which leaves identifier unguessability (FR-6) as the sole defence in v1 — is that acceptable for a self-hosted Instance behind a corporate proxy?
5. How does the Instance handle a datastore that is unavailable at Reveal time, given that a partial Reveal must never both display a Secret and fail to destroy it?

## 11. Assumptions Index

*Every `[ASSUMPTION]` in this document, surfaced for explicit confirmation:*

- **§4.1 / FR-7** — Browsers may refuse a clipboard write outside a user-gesture context, so a visible Link with a manual copy affordance is a requirement rather than a nicety.
- **§9 / SM-6** — Recipient-to-Sender conversion may not be measurable at all, because the product deliberately cannot distinguish a Recipient from a Sender; it may remain a qualitative signal only.
- **Inherited from the brief** — Credential leaks through chat history are asserted to be a leading breach vector based on general security knowledge; this was not measured for this audience and is not load-bearing for any FR.

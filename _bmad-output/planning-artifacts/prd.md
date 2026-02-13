---
stepsCompleted:
  - step-01-init
  - step-02-discovery
  - step-03-success
  - step-04-journeys
  - step-05-domain
  - step-06-innovation
  - step-07-project-type
  - step-08-scoping
  - step-09-functional
  - step-10-nonfunctional
  - step-11-polish
inputDocuments:
  - _bmad-output/planning-artifacts/product-brief-Securer-2026-02-12.md
  - _bmad-output/brainstorming/brainstorming-session-2026-02-12.md
documentCounts:
  briefs: 1
  research: 0
  brainstorming: 1
  projectDocs: 0
classification:
  projectType: web_app
  domain: general
  complexity: low
  projectContext: greenfield
workflowType: 'prd'
---

# Product Requirements Document - Securer (Secret Drop)

**Author:** Nick
**Date:** 2026-02-13

## Executive Summary

Secret Drop is a self-hosted, zero-knowledge, one-time secret sharing web application for developers. Users paste a secret, set an expiration, provide a password, and get a link. The server encrypts using the user-provided password, stores only the ciphertext, and discards plaintext immediately. Secrets are destroyed after a single view or upon expiration.

**Target Users:** Developers sharing API keys and credentials, team leads onboarding new hires, and non-technical users receiving sensitive information.

**Key Differentiators:**
- Zero-knowledge architecture - server cannot decrypt without the user-provided password
- Always one-time view - every secret self-destructs after first access
- Two-factor access - link + password required; link alone is worthless
- Radical simplicity - one page, no accounts, no API, no navigation
- Self-hosted and open source - Docker deployment, full audit transparency

**Project Context:** Greenfield web app (SPA), solo developer, worldwide audience.

## Success Criteria

### User Success

- Users create and share a secret in under 15 seconds on first use, with zero prior knowledge of the tool
- Recipients retrieve secrets without signing up - just a link and a password
- The tool becomes the default method for sharing sensitive info ("just Secret Drop it")

### Business Success

- **Adoption:** Secrets created per week grows organically after deployment
- **End-to-end usage:** Percentage of secrets viewed before expiry
- **Security:** Zero plaintext exposure incidents

### Technical Success

- Deployable with a single `docker-compose up` - no manual config steps
- Minimal resource footprint - lightweight container, small DB, no background workers
- Scales to many concurrent users without expensive infrastructure
- Zero plaintext ever stored at rest - verifiable by code audit

### Measurable Outcomes

- End-to-end secret create-share-retrieve flow completes in under 30 seconds
- Docker deployment works first try on a fresh machine
- App runs comfortably on a single small VPS (1-2 GB RAM)

## Product Scope

### MVP Strategy

**Approach:** Problem-solving MVP - the smallest thing that proves developers will use a simpler, more secure way to share secrets instead of pasting into Slack.

**Resource:** Solo developer. Every feature must justify its existence against "could I ship sooner without it?"

**Core Principle:** The MVP must be faster than pasting into Slack. Under 10 seconds end-to-end or adoption won't happen.

### MVP (Phase 1)

- Single-page sender form (text input, expiration dropdown, password field, random password generator, password-in-link checkbox, generate button)
- Server-side zero-knowledge encryption (AES-256, plaintext never touches storage)
- One-time view with expiration shelf life (1h / 24h / 7d)
- Password-required retrieval with limited attempts (3-5), secret destroyed on exhaustion
- Password-in-link mode (password embedded in URL fragment)
- Auto-copy generated link to clipboard
- Confirmation before reveal
- Generic dead-end screen for all error/failure states
- Docker / docker-compose single-command deployment
- Expiration-based auto-deletion of unclaimed secrets

### Phase 1.5 (Quick Wins)

- QR code representation of links
- Syntax highlighting for code/JSON/YAML on recipient view
- View countdown timer with auto-clear (cosmetic - backend already deletes)
- About page explaining the security model
- "Copy with message" option (pre-written sharing text)

### Phase 2 (Growth)

- Rate limiting and abuse prevention
- Public hosted instance as showcase
- Browser extension for quick secret creation
- CLI tool for developer workflows
- Mobile-optimized UI refinements

### Phase 3 (Expansion)

- Team/org features if demand emerges
- API access
- File/attachment sharing

### Risk Mitigation

**Technical:** Use well-established crypto libraries (no rolling own crypto). Keep architecture dead simple for solo developer maintainability.

**Market:** Make the tool faster than the insecure alternative. Organic spread via recipients becoming senders.

**Resource:** Aggressive Phase 1.5 deferral. Absolute minimum: create secret + retrieve secret + Docker deploy.

## User Journeys

### Journey 1: Alex the Developer - Sharing a Secret (Sender Happy Path)

Alex just generated a new API key for the team's staging environment. His teammate Maria needs it to configure her local setup. Normally he'd paste it into a Slack DM - quick, easy, and he knows it's not great. But his team recently deployed a Secret Drop instance.

Alex opens the Secret Drop page - a single form, nothing else. He pastes the API key, picks "24h" from the expiration dropdown (Maria's online today, she'll grab it), types a password he'll share over a quick call, and leaves "include password in link" unchecked since this is a real credential. He clicks Generate.

The link is instantly copied to his clipboard. He pastes it into Slack: "Hey Maria, staging API key is here: [link]. I'll tell you the password on our standup call." Done in about 10 seconds. He doesn't worry about the key sitting in Slack history because the link is worthless without the password, and it self-destructs after one view anyway.

### Journey 2: Maria the Recipient - Retrieving a Secret (Recipient Happy Path)

Maria sees Alex's Slack message with an unfamiliar link. She clicks it and lands on a minimal page - just a password field and a submit button. On the standup call, Alex tells her the password. She types it in, hits submit.

A warning appears: "This secret will be permanently deleted after you view it. Continue?" She clicks Continue. The API key appears. She copies the key into her `.env` file. If she hits refresh or revisits the link - nothing. Generic "This secret is not available" message. She realizes how clean this is - no signup, no account, just worked.

**The trust moment:** Maria notices a small "How does this work?" link. She skims it - zero-knowledge encryption, one-time view, open source. She feels confident this isn't some sketchy link. Next time she needs to share a password herself, she uses Secret Drop without being asked.

### Journey 3: Dana the Team Lead - Onboarding a New Hire (Convenience Mode)

Dana is onboarding Jake, a new junior developer starting today. She needs to send him the database password, the CI/CD token, and the VPN credentials. Jake isn't technical yet - asking him to receive a password through a separate channel for each secret would be confusing.

For each credential, Dana opens Secret Drop, pastes the value, sets expiration to "1h" (Jake's online right now), enters a password, and checks the "include password in link" box. The generated link has the password embedded in the URL fragment. She drops three links into a welcome email: "Here are your credentials. Each link works once - click, copy, done."

Jake clicks each link. No password prompt - he goes straight to the confirmation screen, views the secret, copies it. Three credentials shared securely without Jake needing to know anything about encryption or Secret Drop. The links are dead within minutes.

### Journey 4: Sam Receives an Expired Secret (Failure Path)

Sam, a project manager, gets a Slack message from two days ago with a Secret Drop link for a shared account password. She clicks it. The page shows: "This secret is not available." That's it - no explanation of whether it expired, was already viewed, or never existed. Sam messages the sender: "Hey, can you re-share that password? The link didn't work." The sender creates a new secret and shares a fresh link. Simple, no confusion, no information leaked.

### Journey 5: Brute Force Attempt (Security Failure Path)

Someone intercepts a Secret Drop link but doesn't have the password. They try "password123" - wrong. "company2024" - wrong. A subtle "X attempts remaining" counter appears. After 3-5 failed attempts, the secret is permanently destroyed. The page shows: "This secret is not available" - the same generic message as every other failure state. The attacker can't tell if they burned the secret or if it was already gone.

### Journey 6: Instance Admin - Deploy and Forget

Chris, a senior developer, decides the team needs Secret Drop. He clones the repo, runs `docker-compose up`, and the app is live behind the company's reverse proxy. No environment variables to configure beyond the basics (port, database). No admin dashboard, no user management, no ongoing maintenance. The app just runs.

### Journey Requirements Summary

| Journey | Key Capabilities Revealed |
|---|---|
| Alex (Sender) | Single-page form, expiration dropdown, password field, auto-copy to clipboard, instant link generation |
| Maria (Recipient) | Password prompt page, confirmation before reveal, auto-clear, trust-building About page |
| Dana (Convenience) | Password-in-link checkbox, URL fragment embedding, no-password-prompt flow |
| Sam (Expired) | Generic dead-end screen, identical message for all failure states |
| Brute Force | Limited password attempts, attempt counter, secret destruction on exhaustion |
| Chris (Admin) | Docker deployment, minimal config, no admin UI |

## Web App Technical Requirements

**Application Type:** SPA (Single Page Application)
- Sender view: single form (text input, expiration dropdown, password field, password-in-link checkbox, generate button)
- Recipient view: password prompt -> confirmation -> secret reveal -> auto-clear
- Dead-end view: generic "This secret is not available" for all error states

**Browser Support:** Modern browsers only (last 2 versions) - Chrome, Firefox, Safari, Edge. Enables use of modern APIs (Clipboard API, CSS Grid/Flexbox, ES2020+).

**Responsive Design:** Desktop-first, works on mobile (recipients may open links on phones). Minimal UI adapts naturally to any width.

**Architecture:** Pure request-response. No WebSockets, polling, or server-sent events. No SEO, SSR, or meta tags needed. Lightweight frontend - server handles all encryption, client is a thin form over TLS. Static assets served by the same container as the backend.

## Functional Requirements

### Secret Creation

- **FR1:** Sender can input secret text content into a text field
- **FR2:** Sender can select an expiration period (1 hour, 24 hours, 7 days)
- **FR3:** Sender can provide a password required to access the secret
- **FR4:** Sender can generate a random password via the UI instead of typing one manually
- **FR5:** Sender can optionally include the password in the generated link (via URL fragment)
- **FR6:** Sender can generate a unique, shareable link for the created secret
- **FR7:** Sender receives the generated link automatically copied to their clipboard

### Secret Retrieval

- **FR8:** Recipient can open a secret link and be prompted for a password
- **FR9:** Recipient with a password-in-link URL can access the secret without a separate password prompt
- **FR10:** Recipient can view a confirmation warning before the secret is revealed
- **FR11:** Recipient can view the decrypted secret content after providing the correct password and confirming
- **FR12:** Recipient can copy the revealed secret content

### Secret Lifecycle

- **FR13:** System destroys the secret immediately after it has been viewed once
- **FR14:** System destroys unclaimed secrets automatically when their expiration period elapses
- **FR15:** System destroys the secret after the maximum number of failed password attempts is reached
- **FR16:** System tracks remaining password attempts per secret

### Security & Encryption

- **FR17:** System encrypts secret content server-side using the sender-provided password before storing
- **FR18:** System stores only encrypted ciphertext - plaintext never persists in storage
- **FR19:** System decrypts secret content in memory only at the moment of authorized retrieval
- **FR20:** System discards plaintext and password from memory immediately after encryption or decryption

### Error Handling

- **FR21:** System displays an identical generic message for all failure states (expired, viewed, burned, nonexistent)
- **FR22:** System reveals no information about whether a secret ever existed, was already viewed, or was destroyed

### Deployment

- **FR23:** Administrator can deploy the application using a single `docker-compose up` command
- **FR24:** Administrator can configure basic settings (port, database) via environment variables

## Non-Functional Requirements

### Performance

- Page load time under 1 second on standard broadband connections
- Secret creation (encryption + storage) completes in under 500ms server-side
- Secret retrieval (decryption + delivery) completes in under 500ms server-side
- UI remains responsive during link generation (no blocking operations on the client)
- Minimal JS bundle size

### Security

- All client-server communication over TLS (HTTPS only)
- No logging of secret content, passwords, or decrypted data at any point
- No logging of request bodies on secret creation or retrieval endpoints
- Encryption keys (user passwords) never written to disk, logs, or persistent storage
- Failed password attempts do not reveal whether the secret exists
- URL fragments (password-in-link mode) are never sent to the server by the browser

### Scalability

- System supports thousands of concurrent users worldwide
- Stateless request handling - any instance can serve any request
- Database operations are simple key-value lookups (O(1) by secret ID)
- Horizontal scaling possible by adding container instances behind a load balancer
- Storage grows linearly with active (unexpired) secrets only - expired secrets are cleaned up
- Single small VPS (1-2 GB RAM) handles typical usage; scales horizontally for worldwide adoption

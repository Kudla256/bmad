---
stepsCompleted: [1, 2, 3, 4, 5]
inputDocuments:
  - _bmad-output/brainstorming/brainstorming-session-2026-02-12.md
date: 2026-02-12
author: Nick
---

# Product Brief: Securer

## Executive Summary

Securer (Secret Drop) is a self-hosted, zero-knowledge, one-time secret sharing web application for developers. It provides a radically simple interface — paste a secret, set an expiration, provide a password, get a link — backed by server-side encryption where plaintext never touches storage. Secrets are destroyed after a single view or upon expiration. Deployable via Docker, open source, and designed to eliminate the need for insecure secret sharing through chat, email, or persistent pastebins.

---

## Core Vision

### Problem Statement

Developers routinely share sensitive data — API keys, database credentials, tokens, config snippets — through insecure channels like Slack, email, and pastebins. These channels persist messages, index content, and provide no guarantees of confidentiality or ephemerality. A leaked credential in a Slack channel can be searched and found months later.

### Problem Impact

Credential leaks are a leading vector for security breaches. When secrets live in chat histories and inboxes, every person with channel access becomes an attack surface. The longer secrets persist, the higher the exposure risk. Teams that lack a simple, secure sharing tool default to the most convenient (and least secure) option available.

### Why Existing Solutions Fall Short

Current secret-sharing tools suffer from one or more of these issues:
- **Account requirements** — friction that pushes users back to copy-paste in chat
- **Client-side encryption complexity** — WebCrypto APIs, key derivation in JS, browser compatibility headaches
- **Server can read secrets** — encryption exists but the provider holds the keys
- **Feature bloat** — dashboards, teams, audit logs, pricing tiers for what should be a 10-second operation
- **No self-hosting** — users must trust a third-party server with their most sensitive data

### Proposed Solution

Secret Drop is a single-page web app with a minimal form: text input, expiration dropdown (1h/24h/7d), password field, optional password-in-link checkbox, and a generate button. The server encrypts the secret in memory using the user-provided password, stores only the encrypted blob, and discards the plaintext immediately. Recipients enter the password to decrypt (with limited attempts before destruction), view the secret once with a countdown timer, and the backend deletes the record the moment it's served. Deployment is a single `docker-compose up`.

### Key Differentiators

- **Zero-knowledge architecture** — server stores only encrypted blobs it cannot decrypt without the user-provided password
- **Always one-time view** — every secret self-destructs after first access; expiration only controls the unclaimed shelf life
- **Two-factor access** — link + password required; link alone is worthless (with optional convenience mode embedding password in URL fragment)
- **Radical simplicity** — one page, no accounts, no API, no navigation, no branding clutter
- **Self-hosted and open source** — Docker deployment, full audit transparency, zero third-party trust required
- **Fail-secure defaults** — limited password attempts, generic error screens preventing information leakage, auto-clear timer on viewed secrets

## Target Users

### Primary Users

**Persona 1: Alex — The Developer**
Backend developer who regularly shares API keys, database credentials, and config snippets with teammates. Currently pastes them into Slack DMs knowing it's not great, but it's fast. Wants something just as fast but actually secure.

**Persona 2: Dana — The Team Lead / Ops**
Manages onboarding and distributes access credentials to new team members. Needs to send passwords, tokens, and connection strings to people who aren't always technical. Values something dead-simple that doesn't require the recipient to sign up for anything.

**Persona 3: Sam — The Non-Technical User**
A project manager, designer, or business user who occasionally receives or shares sensitive info — a license key, a shared account password, a confidential document link. Doesn't care how encryption works, just needs it to be obvious and one-step.

### Secondary Users

**Instance Administrator** — The person who runs `docker-compose up` and maintains the self-hosted deployment. Likely a developer or sysadmin on the team. Their concern is easy setup and minimal maintenance.

### User Journey

1. **Discovery** — Teammate sends them a Secret Drop link, or their company deploys an internal instance
2. **First Use (Sender)** — Lands on a single page, pastes secret, sets expiration, enters password, clicks generate, gets a link copied to clipboard. Done in under 10 seconds.
3. **First Use (Recipient)** — Opens link, enters password, sees the secret, acknowledges it will be destroyed. Views it, copies what they need, page clears.
4. **"Aha" Moment** — Realizes there's nothing to sign up for, no account, no friction — just works.
5. **Routine** — Becomes the default way to share anything sensitive. "Just Secret Drop it."

## Success Metrics

### User Success
- **Task completion** — A user can create and share a secret in under 15 seconds on first use
- **Zero-friction onboarding** — No signup, no tutorial needed; recipients can retrieve a secret without prior knowledge of the tool
- **Reliability** — Secrets are always retrievable (before expiry/view) and always destroyed after

### Business Objectives
- **Adoption** — Becomes the default method for sharing sensitive info within the deploying team/org
- **Security improvement** — Eliminates secrets persisting in chat logs, emails, and pastebins
- **Low maintenance** — Runs unattended via Docker with minimal ops overhead

### Key Performance Indicators
- Secrets created per week (adoption signal)
- Percentage of secrets viewed before expiry (indicates the tool is actually being used end-to-end)
- Zero plaintext exposure incidents (the whole point)
- Instance uptime (self-hosted reliability)

## MVP Scope

### Core Features

**Sender Flow:**
- Single-page form: text input, expiration dropdown (1h / 24h / 7d), password field, "include password in link" checkbox, generate button
- Server-side encryption using user-provided password (AES-256 or similar)
- Zero-knowledge architecture — plaintext never touches storage, encrypted in memory only
- Auto-copy generated link to clipboard
- QR code representation of the link

**Recipient Flow:**
- Password prompt page (minimal — input + submit only)
- Limited password attempts (3-5) — secret destroyed on exhaustion
- Confirmation before reveal ("This will be permanently deleted. Continue?")
- Secret display with syntax highlighting for code/JSON/YAML
- View countdown timer with auto-clear (cosmetic — backend deletes immediately on serve)
- Generic dead-end screen for all failure states (expired, burned, viewed, nonexistent — same message)

**Security:**
- Server-side encryption/decryption in memory, ciphertext-only storage
- One-time view — secret deleted from DB the moment it's served
- Expiration-based auto-deletion for unclaimed secrets
- Password-in-link mode (password embedded in URL fragment) as optional convenience
- No logging of secret content

**Deployment:**
- Docker / docker-compose single-command deployment
- Open source

**Additional:**
- About/explanation page describing the security model
- "Copy with message" option (pre-written sharing text alongside the link)

### Out of Scope for MVP
- Public hosted instance (no SaaS — self-hosted only)
- User accounts or authentication
- API access
- File/attachment sharing (text secrets only, 10-50KB limit)
- Admin dashboard or analytics UI
- Rate limiting (can be added later)
- Audit logs

### MVP Success Criteria
- A user can create, share, and retrieve a secret end-to-end in under 30 seconds
- Zero plaintext ever stored at rest — verifiable by code audit
- Docker deployment works with a single `docker-compose up`
- All failure states return identical generic responses (no information leakage)

### Future Vision
- Public hosted instance as a showcase / marketing tool
- Rate limiting and abuse prevention for public deployment
- Browser extension for quick secret creation
- CLI tool for developer workflows
- Team/org features if demand emerges
- Mobile-optimized UI refinements

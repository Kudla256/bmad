---
title: Securer
status: final
created: 2026-02-12
updated: 2026-09-04
---

# Product Brief: Securer

## Executive Summary

Securer (working name "Secret Drop") is a self-hosted, zero-knowledge, one-time secret sharing web application for developers. One page, one form: paste a secret, set an expiration, provide a password, get a link. The server encrypts in memory with the sender's password, stores only ciphertext, and discards the plaintext immediately. Every secret self-destructs after a single view or when its expiration elapses.

The problem it addresses is mundane and constant: developers share API keys, database credentials and tokens over Slack, email and pastebins because those channels are the fastest thing at hand. Those channels persist, index and replicate what passes through them, so a credential shared once stays discoverable for months. Existing secret-sharing tools lose to Slack not on security but on friction — accounts, dashboards, third-party trust.

Securer wins by being faster than the insecure default while giving up nothing. It deploys with a single `docker-compose up`, requires no accounts, and is open source, so a team can read the code that holds their credentials. If sharing a secret through it is quicker than pasting into a DM, adoption follows without a mandate.

## The Problem

Developers routinely move sensitive data — API keys, database credentials, tokens, config snippets — through channels built for conversation, not confidentiality. Slack keeps history and indexes it. Email keeps copies on every hop. Pastebins outlive the need that created them. A credential dropped into a team channel can be searched and found long after the person who shared it has forgotten it exists.

The cost compounds. Every person with channel access becomes an attack surface, and the exposure window is measured in months rather than minutes. Credential leaks remain a leading breach vector, and the leaks that matter are rarely sophisticated — they are secrets sitting in a chat log that nobody thought to clean up.

Teams know this. They still default to Slack, because the secure option is always the slower one. Without a tool that is genuinely faster than a DM, the insecure path stays the convenient path.

Existing tools fall short in familiar ways:

- **Account requirements** — friction that pushes users straight back to copy-paste in chat.
- **Client-side encryption complexity** — WebCrypto, key derivation in JS, browser compatibility headaches.
- **Server can read secrets** — encryption exists, but the provider holds the keys.
- **Feature bloat** — dashboards, teams, audit logs and pricing tiers around what should be a ten-second operation.
- **No self-hosting** — the most sensitive data a team has must be handed to a third party.

## The Solution

A single-page web app with a minimal form: text input, expiration dropdown (1h / 24h / 7d), password field, an optional "include password in link" checkbox, and a generate button. On submit, the server encrypts the secret in memory using the sender's password, persists only the encrypted blob, and drops the plaintext and password before responding. The generated link lands on the sender's clipboard automatically.

The recipient opens the link and is asked for the password. Limited attempts protect against guessing — exhausting them destroys the secret. A confirmation step warns that viewing is permanent, then the secret appears once. The backend deletes the record the moment it is served; a countdown and auto-clear on screen mirror what has already happened server-side.

Every failure state — expired, already viewed, burned by failed attempts, never existed — returns the same generic dead-end screen, so a link reveals nothing about whether a secret was ever there.

Deployment is one command. No admin UI, no user management, no background workers to babysit.

## What Makes This Different

- **Zero-knowledge architecture** — the server holds ciphertext it cannot decrypt without the sender's password.
- **Always one-time view** — every secret self-destructs after first access; expiration governs only the unclaimed shelf life.
- **Two-factor access** — link plus password; an intercepted link alone is worthless. An optional convenience mode embeds the password in the URL fragment, which the browser never sends to the server.
- **Radical simplicity** — one page, no accounts, no API, no navigation, no branding clutter.
- **Self-hosted and open source** — Docker deployment, full audit transparency, no third-party trust required.
- **Fail-secure defaults** — limited attempts, uniform error screens, auto-clear on viewed secrets.

The honest read on the moat: none of these primitives is novel, and a competitor could copy any of them. The advantage is the combination held at a level of simplicity that competitors keep abandoning as they grow features. Execution discipline is the moat.

## Who This Serves

**Alex — the developer.** Backend developer sharing API keys, database credentials and config snippets with teammates several times a week. Currently uses Slack DMs knowing it is not great, because it is fast. Success is a tool that is just as fast and actually secure.

**Dana — the team lead.** Runs onboarding and distributes access credentials to new hires with varying technical skill. Needs something dead-simple that requires no signup and no explanation on the recipient's side. The password-in-link mode exists for her.

**Sam — the non-technical recipient.** A PM, designer or business user who occasionally receives something sensitive — a license key, a shared account password. Does not care how encryption works and should not have to. Success is: click link, enter password, see secret, done.

**Chris — the instance administrator** (secondary). Runs `docker-compose up` and maintains the deployment. Cares about setup simplicity and zero ongoing maintenance; there is deliberately no admin surface for them to operate.

Full persona depth and the discovery-to-routine adoption arc are in `addendum.md`.

## Success Criteria

**User success**

- A first-time user creates and shares a secret in under 15 seconds, with no prior knowledge of the tool.
- Recipients retrieve secrets with no signup — a link and a password, nothing else.
- Secrets are always retrievable before view or expiry, and always destroyed after.

**Business success**

- Secrets created per week grows organically after deployment (adoption signal).
- A meaningful share of secrets are viewed before expiry (proves end-to-end use, not just creation).
- Zero plaintext exposure incidents.
- The tool becomes the default for sensitive sharing within the deploying team — "just Secret Drop it."

**Technical success**

- `docker-compose up` works first try on a fresh machine, with no manual configuration steps.
- Zero plaintext stored at rest, verifiable by reading the code.
- Runs comfortably on a single small VPS (1–2 GB RAM) and scales horizontally without expensive infrastructure.
- End-to-end create → share → retrieve completes in under 30 seconds.

## Scope

**In for v1**

- Single-page sender form: text input, expiration (1h / 24h / 7d), password field, random password generator, password-in-link checkbox, generate button.
- Server-side zero-knowledge encryption; plaintext never persisted.
- One-time view with expiration-based shelf life for unclaimed secrets.
- Password-required retrieval with limited attempts; secret destroyed on exhaustion.
- Password-in-link convenience mode via URL fragment.
- Auto-copy of the generated link to the clipboard.
- Confirmation before reveal.
- Uniform generic dead-end screen for every failure state.
- Docker / docker-compose single-command deployment.

**Explicitly out for v1**

- Public hosted instance (self-hosted only — no SaaS).
- User accounts or authentication.
- API access.
- File or attachment sharing (text only).
- Admin dashboard or analytics UI.
- Rate limiting and abuse prevention (needed only for a public instance).
- Audit logs.

Near-term deferrals that were considered and consciously parked — QR codes, syntax highlighting, an About page explaining the security model, "copy with message" — are recorded in `addendum.md` with the reasoning.

## Vision

If Securer works, the shape it grows into stays deliberately small. A public hosted instance serves as a showcase rather than a business, with rate limiting and abuse prevention added to make that safe. A browser extension and a CLI put secret creation where developers already are. Team or org features arrive only if real demand shows up, never speculatively.

The end state is not a platform. It is the tool a team installs once, never thinks about again, and would notice immediately if it disappeared — the moment "just Secret Drop it" replaces pasting a credential into a channel.

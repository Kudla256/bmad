---
title: Securer — Brief Addendum
status: final
created: 2026-02-12
updated: 2026-09-04
---

# Addendum: Securer

Depth captured during brief discovery that belongs downstream (PRD, UX, architecture) rather than in the brief itself.

## Personas in depth

**Alex — the developer.** Backend developer who regularly shares API keys, database credentials and config snippets with teammates. Currently pastes them into Slack DMs knowing it is not great, but it is fast. Wants something just as fast but actually secure. Power-user posture: will use keyboard, will not read instructions, measures the tool against the ten seconds it takes to paste into a DM.

**Dana — the team lead / ops.** Manages onboarding and distributes access credentials to new team members. Sends passwords, tokens and connection strings to people who are not always technical. Values something dead-simple that does not require the recipient to sign up for anything. Her constraint drives the password-in-link mode: asking a new hire to receive a password over a second channel, per credential, does not survive contact with a first day.

**Sam — the non-technical user.** A project manager, designer or business user who occasionally receives or shares sensitive info — a license key, a shared account password, a confidential document link. Does not care how encryption works, just needs it to be obvious and one-step. Lands on an unfamiliar link and must immediately feel safe.

**Instance administrator (secondary).** The person who runs `docker-compose up` and maintains the self-hosted deployment — likely a developer or sysadmin on the team. Their entire concern is easy setup and minimal maintenance. Deliberately given no admin UI.

## Adoption arc

1. **Discovery** — a teammate sends a Securer link, or the company deploys an internal instance.
2. **First use (sender)** — lands on a single page, pastes the secret, sets expiration, enters a password, clicks generate, gets a link on the clipboard. Under 10 seconds.
3. **First use (recipient)** — opens the link, enters the password, acknowledges the secret will be destroyed, views it, copies what they need, page clears.
4. **"Aha" moment** — realises there is nothing to sign up for, no account, no friction. It just works.
5. **Routine** — becomes the default way to share anything sensitive. "Just Secret Drop it."

The recipient-becomes-sender conversion is the growth mechanism: a recipient who has a good experience is the next sender, without any marketing surface.

## Deferred features and the reasoning

Considered during discovery, cut from v1 to protect the solo-developer timeline. None were rejected on merit.

| Feature | Why deferred |
|---|---|
| QR code representation of the link | Genuinely useful for phone hand-off, but adds a dependency and UI surface for a case v1 does not need to prove. |
| Syntax highlighting for code / JSON / YAML on the recipient view | Pure polish on an already-working reveal. Adds a highlighting library to a page whose whole selling point is that it is thin. |
| View countdown timer with auto-clear | Cosmetic — the backend deletes on serve, so the timer communicates rather than enforces. Ships when the reveal screen gets its second pass. |
| About page explaining the security model | Load-bearing for recipient trust and arguably the product's best marketing. Deferred only because it is content work, not product work. Worth revisiting early. |
| "Copy with message" (pre-written sharing text) | Small, cheap, and helps senders explain the link. First candidate to pull forward if timeline allows. |

## Risk register

**Technical.** Do not roll custom crypto — use the platform's established libraries. Keep the architecture dead simple, because a solo developer maintains it. The destructive-read semantics (retrieval *is* deletion) is the one place where a subtle bug becomes a correctness and security failure at once; it deserves disproportionate test attention.

**Market.** The tool must be faster than the insecure alternative, or it loses to Slack regardless of its security properties. Under 10 seconds end-to-end is the adoption threshold, not an aspiration. Organic spread depends on recipients becoming senders.

**Resource.** Solo developer. Every feature must justify itself against "could I ship sooner without it?" Aggressive deferral of the Phase 1.5 set is the mitigation. The absolute floor is: create a secret, retrieve a secret, deploy with Docker.

## Constraints carried forward

- Text secrets only, with a practical size ceiling in the 10–50 KB range.
- Modern browsers only (last two versions), which unlocks the Clipboard API and modern CSS without polyfill work.
- Desktop-first, but recipients will open links on phones, so the layout must survive narrow widths.
- Pure request-response. No WebSockets, polling or server-sent events. No SEO, SSR or meta-tag concerns.
- Storage grows only with active, unexpired secrets — expired records are removed, so the database does not accumulate.

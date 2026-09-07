# Brainstorm Intent: Secret Drop

Distilled from `.memlog.md` (27 ideas, SCAMPER + Role Playing, 2026-02-12). This is the intent a downstream skill should build on — the chosen and load-bearing discoveries only, not the session record.

## The product in one line

A self-hosted, zero-knowledge, one-time secret sharing tool for developers — as simple as Pastebin, as secure as it gets.

## The governing constraint

Radical simplicity, applied as a veto rather than an aspiration. Every feature proposed during the session was met with a push to simplify further, and the decisions that survived are the ones that made the product smaller: no accounts, no API, no client-side crypto, no public hosting for v1. Simplicity is treated as the security feature — fewer moving parts, fewer attack vectors.

## Architecture that was settled

- **Zero-knowledge, server-side encryption.** The server encrypts in memory on receipt, stores only ciphertext, and discards plaintext and password immediately. It cannot read secrets, where competitors merely choose not to.
- **No client-side encryption.** No WebCrypto, no key derivation in JS. TLS protects transit; server-side encryption protects storage; the client stays a form. Double encryption was considered and dropped — simplicity won.
- **Sender-provided password as the key.** The sender sets it; link and password travel over different channels.
- **Always one-time view.** Every secret dies on first view. Expiration (1h / 24h / 7d) governs only how long an unclaimed link waits. Access policy and availability window are separate concepts.
- **Two-factor access.** The link alone is worthless. Two pieces of knowledge required, where competitors require one.
- **Limited attempts.** 3–5 wrong passwords destroys the secret. Fail-secure; most competitors do not limit attempts at all.
- **Text only, 10–50KB.** The ceiling is a feature: it keeps the product focused and prevents abuse.

## Experience that was settled

- **Zero-chrome sender page.** One page, one form. No accounts, signup, landing copy or navigation. Anti-design as a feature.
- **Smart default expiration.** Pre-set to a sensible default so the default does the thinking and casual users never touch it.
- **Auto-copy on generate.** The link hits the clipboard the instant it exists; the user's next action is paste.
- **Minimal recipient page.** Password field, submit, and an attempts-remaining line after a failure. No branding, no explanation — it trusts the human channel rather than cluttering the UI.
- **Confirmation before reveal.** Reverses the click-and-it-is-gone-before-you-realised problem by giving the recipient a conscious moment.
- **Generic dead-end screen.** Expired, burned, already viewed and never existed all show one identical message. Information-leakage prevention as a first-class design principle.
- **Password-in-link mode.** One checkbox embeds the password in the URL fragment, trading the second factor for one-link convenience on lower-stakes secrets.

## Distribution that was settled

- **Open source** — turns "trust us" into "verify us"; the zero-knowledge claim is auditable or it is marketing.
- **Self-hostable via Docker** — one `docker-compose up` gives a company its own instance behind its own firewall, which eliminates the biggest objection outright.
- **Public instance is a demo, not the product.** Deferred out of v1 entirely. No freemium, no pricing tiers.

## Connections worth carrying forward

- **One-time view and the auto-clear timer are one idea.** Destruction is always immediate and server-side; everything the user sees about timing is communication, never mechanism. Any implementation that makes the countdown load-bearing has misread this.
- **The generic dead end is what makes the attempt burn safe.** Burning a secret after failed attempts only avoids handing an attacker a signal because the dead end refuses to distinguish a burn from an expiry. The two features are a single security property and must not be weakened independently.
- **Anti-pastebin positioning and the zero-chrome page are the same bet.** Familiarity of form, inversion of philosophy — anyone who knows Pastebin understands this product with zero learning curve.

## Deliberately refused

- An API. Refusing one is a security and simplicity decision — no API keys, no auth system, no abuse vectors.
- Client-side cryptography, with its WebCrypto and browser-compatibility burden.
- Accounts, sessions, and any notion of user identity.
- A public hosted instance in v1.

## Deferred, kept

QR code representation of the link · syntax-aware highlighting on the recipient view · copy-with-message pre-written sharing text · an about page explaining the security model. All four survived prioritisation on merit and were sequenced behind the core loop, not rejected.

## Left unmined

Chaos Engineering was recommended as the third technique and never run. Deliberate failure-mode exploration — what happens when storage is unavailable mid-reveal, what a partial destruction looks like, how the system behaves under concurrent access to one secret — was therefore never done in the session. Those questions surfaced later as open items in the PRD and architecture spine, and a follow-up session on that seam would be well spent.

---
name: Securer
description: Clean-futurism visual identity for a zero-knowledge one-time secret sharing tool — whitespace-dominant, typography-led, indigo-accented, with a single glass surface reserved for the sender form.
status: final
created: 2026-02-13
updated: 2026-09-04
sources:
  - ../../prds/prd-Securer-2026-02-13/prd.md
  - ../../briefs/brief-Securer-2026-02-12/brief.md
companions:
  - EXPERIENCE.md

colors:
  canvas: '#FFFFFF'
  canvas-muted: '#FAFAFA'
  surface: '#F9FAFB'
  border: '#E5E7EB'
  text-primary: '#0A0A0A'
  text-secondary: '#6B7280'
  text-tertiary: '#9CA3AF'
  accent: '#6366F1'
  accent-strong: '#4F46E5'
  accent-soft: '#818CF8'
  accent-tint: '#EEF2FF'
  success: '#10B981'
  error: '#EF4444'
  warning: '#F59E0B'
  warning-surface: '#FFFBEB'
  warning-border: '#FDE68A'
  warning-text: '#92400E'

typography:
  display:
    fontFamily: Inter
    fontSize: '2.5rem'
    fontWeight: 600
    lineHeight: 1.2
  h1:
    fontFamily: Inter
    fontSize: '2rem'
    fontWeight: 600
    lineHeight: 1.2
  h2:
    fontFamily: Inter
    fontSize: '1.5rem'
    fontWeight: 600
    lineHeight: 1.2
  body-large:
    fontFamily: Inter
    fontSize: '1.125rem'
    fontWeight: 400
    lineHeight: 1.6
  body:
    fontFamily: Inter
    fontSize: '1rem'
    fontWeight: 400
    lineHeight: 1.6
  label:
    fontFamily: Inter
    fontSize: '0.875rem'
    fontWeight: 500
    lineHeight: 1.4
  small:
    fontFamily: Inter
    fontSize: '0.875rem'
    fontWeight: 400
    lineHeight: 1.6
  tiny:
    fontFamily: Inter
    fontSize: '0.75rem'
    fontWeight: 500
    lineHeight: 1.4
  mono:
    fontFamily: JetBrains Mono
    fontSize: '0.875rem'
    fontWeight: 400
    lineHeight: 1.6

rounded:
  sm: '6px'
  md: '10px'
  lg: '16px'
  xl: '24px'
  full: '9999px'

spacing:
  '1': '4px'
  '2': '8px'
  '4': '16px'
  '6': '24px'
  '8': '32px'
  '12': '48px'
  '16': '64px'
  form-gap: '20px'
  page-margin-mobile: '16px'
  form-max-width: '480px'
  reveal-max-width: '520px'
  deadend-max-width: '360px'

components:
  button-primary:
    background: '{colors.accent}'
    backgroundHover: '{colors.accent-strong}'
    color: '#FFFFFF'
    radius: '{rounded.md}'
    fontSize: '0.9375rem'
    fontWeight: 600
    minHeight: '44px'
    width: 'full within form'
  button-secondary:
    background: 'transparent'
    backgroundHover: '{colors.accent-tint}'
    color: '{colors.accent}'
    border: '1px solid {colors.accent}'
    radius: '{rounded.md}'
    minHeight: '44px'
  button-ghost:
    background: 'none'
    color: '{colors.accent}'
    textDecorationHover: 'underline'
    border: 'none'
  input:
    background: '{colors.canvas}'
    border: '1px solid {colors.border}'
    radius: '{rounded.md}'
    padding: '12px 16px'
    placeholderColor: '{colors.text-tertiary}'
    focusBorder: '{colors.accent}'
    focusRing: '3px rgba(99,102,241,0.1)'
  textarea:
    extends: 'input'
    fontFamily: '{typography.mono.fontFamily}'
    minHeight: '120px'
    resize: 'vertical'
  select:
    extends: 'input'
    chevron: 'custom, {colors.text-secondary}'
  checkbox:
    size: '18px'
    accent: '{colors.accent}'
    radius: '{rounded.sm}'
  glass-card:
    background: 'rgba(255,255,255,0.75)'
    backdropFilter: 'blur(20px)'
    border: '1px solid rgba(255,255,255,0.8)'
    radius: '{rounded.xl}'
    shadow: '{elevation.soft}'
  split-panel:
    background: 'linear-gradient(135deg, {colors.accent-strong}, {colors.accent}, {colors.accent-soft})'
    color: '#FFFFFF'
  secret-display:
    background: '{colors.surface}'
    border: '1px solid {colors.border}'
    radius: '{rounded.md}'
    fontFamily: '{typography.mono.fontFamily}'
    fontSize: '{typography.mono.fontSize}'
    whiteSpace: 'pre-wrap'
    wordBreak: 'break-all'
  toast-success:
    background: '{colors.success}'
    color: '#FFFFFF'
    radius: '{rounded.md}'
  warning-banner:
    background: '{colors.warning-surface}'
    border: '1px solid {colors.warning-border}'
    color: '{colors.warning-text}'
    radius: '{rounded.md}'
  status-icon-success:
    size: '64px'
    shape: '{rounded.full}'
    color: '{colors.success}'
  status-icon-deadend:
    size: '56px'
    shape: '{rounded.full}'
    color: '{colors.text-tertiary}'
---

# DESIGN.md — Securer

Visual identity. `EXPERIENCE.md` owns behavior and references these tokens by name. Where a mock, wireframe or import disagrees with this file, this file wins.

## Brand & Style

Clean futurism. The visual posture is a professional instrument, not a security product — quiet competence rather than fortification. A recipient lands on an unfamiliar link holding a credential someone trusted them with, and the page has about five seconds to look legitimate enough that they will type a password into it. That judgment is made on visual quality alone, before a word is read.

Three references define the register. **Google Search** for the whitespace-dominant single-focal-point layout: one thing to do, vast quiet around it, confidence expressed as restraint. **Apple product pages** for typography-first hierarchy with no UI chrome — no boxes, no borders, no decorative furniture; type and space do the work. **Linear** as proof that a developer tool can be visually polished without becoming utilitarian.

The aesthetic reads 2026 through material rather than ornament: one frosted-glass surface, generous corner radii, soft diffuse shadows, a single gradient. Everything else is white space and near-black type.

What this identity explicitly is not: no dark hacker aesthetic, no shields or padlock badges, no "military-grade encryption" language, no default component-library look. Security theater is the failure mode — a page that overcompensates with security imagery reads as less trustworthy, not more. The security model speaks through the copy and the behavior; the visuals stay calm.

## Colors

The palette is near-monochrome with one accent. Color carries meaning; it never decorates.

- **`{colors.canvas}`** — pure white, the ground for everything. `{colors.canvas-muted}` differentiates a section only when a boundary genuinely helps; most of the time nothing is needed.
- **`{colors.surface}`** — the faintest fill, used for content the user reads *out of* rather than *acts on*: the revealed secret, the generated link box.
- **`{colors.border}`** — ultra-subtle hairline for form fields. Present so a field is findable, never so it is emphatic.
- **`{colors.text-primary}`** — near-black at roughly 18:1 on white. Sharp and modern; not a soft gray that reads as unfinished.
- **`{colors.text-secondary}`** for labels and supporting information, **`{colors.text-tertiary}`** for placeholders, counters and the dead-end lock. Three levels, no more.
- **`{colors.accent}`** — indigo, the single accent. It marks exactly one thing per screen: what to do next. It was chosen for the intersection of "modern tech" and "trustworthy institution" — the register of Linear, Stripe's docs, Vercel — avoiding both the generic blue of legacy enterprise tools and trend colors that will date. `{colors.accent-strong}` on hover, `{colors.accent-soft}` only as the third stop of the sender gradient, `{colors.accent-tint}` as the secondary-button hover wash.
- **`{colors.success}`** — emerald, and only for confirmation: the copied badge, the success icon. Never a general-purpose "good" color.
- **`{colors.error}`** — red, and only for an incorrect password and form validation. It is deliberately rationed; the dead-end screen — the most common failure a user meets — uses no red at all.
- **`{colors.warning}`** with `{colors.warning-surface}` / `{colors.warning-border}` / `{colors.warning-text}` — amber, reserved for destruction. It appears at the pre-reveal confirmation and the post-reveal "this has been deleted" banner, and nowhere else. Amber means *irreversible*, so it must stay rare enough to still mean it.

The dead-end screen gets no semantic color at all. It is a neutral terminal state, not an error — treating it as one would make routine ephemerality feel like a malfunction.

## Typography

Two families. **Inter** for everything the interface says: geometric, highly legible at small sizes, variable-weight. **JetBrains Mono** for everything the user's own data occupies — the secret input, the revealed secret, the generated link URL. The switch to mono is semantic: it tells a developer "this is your content, verbatim, nothing was reflowed."

The ramp is a 1.25 scale from a 16px base: `{typography.display}` for the product name, `{typography.h1}` for page headings, `{typography.h2}` for section headings, `{typography.body-large}` for key instructions, `{typography.body}` for prose, `{typography.small}` for hints and metadata, `{typography.tiny}` for badges and counters. `{typography.label}` is the form-label role — 14px at weight 500, tight leading.

Line height carries the calm: 1.2 on headings, 1.6 on body, 1.4 on labels and other compact runs. Nothing goes below 12px, and every size is expressed in rem so browser text scaling works.

Fonts load as variable with `font-display: swap`, sized to avoid layout shift on swap.

## Layout & Spacing

An 8px base unit generates the scale. `{spacing.1}` for inline icon gaps, `{spacing.2}` for internal field padding, `{spacing.4}` between a label and its field, `{spacing.6}` between form groups, `{spacing.8}` between major sections, `{spacing.12}` and `{spacing.16}` for page-level separation. `{spacing.form-gap}` is the specific 20px rhythm between form groups in the sender card.

The layout is a centered single column that never becomes multi-column. Forms cap at `{spacing.form-max-width}`, the revealed secret at `{spacing.reveal-max-width}`, the dead-end message at `{spacing.deadend-max-width}` — narrow, because a short message centered in a wide field reads as an accident.

The page should feel roughly 60% empty. Minimum `{spacing.6}` between form elements; when in doubt, more. Whitespace is the primary trust signal in this identity, which makes it functional rather than stylistic — a denser version of this layout is a worse product, not merely a different look.

There is no chrome: no header bar, no sidebar, no footer navigation. The product name and a single "How does this work?" link are the only elements that are not the form.

Breakpoints are 768px and 1024px. Below 768px the sender's gradient panel is removed entirely and the form goes full-width inside `{spacing.page-margin-mobile}` gutters. Type sizes do not scale responsively — 16px base is correct at every width. No surface may scroll horizontally at any width.

## Elevation & Depth

Depth is soft and diffuse, never hard-edged. There are exactly two elevated things in the product: the sender's glass card and the pre-reveal dialog. Everything else sits flat on the canvas.

- **`{components.glass-card}`** — the sender form: a frosted surface at `rgba(255,255,255,0.75)` over `blur(20px)`, a near-white 1px edge, and a wide low-opacity shadow. Blurred indigo orbs sit behind it in the page background to give the blur something to resolve. `backdrop-filter` is progressive enhancement — behind `@supports`, the fallback is solid white, and the layout is identical either way.
- **Dialog** — a centered overlay over a dimmed backdrop. The dim is what creates the depth; the dialog itself stays visually quiet.

On mobile the blur radius is reduced or dropped for solid white. The glass is an enhancement, never load-bearing.

## Shapes

Corner radii are deliberately large. `{rounded.md}` on inputs, selects and buttons; `{rounded.lg}` on mid-size containers; `{rounded.xl}` on the glass card; `{rounded.full}` on status icon circles and badges.

The logic is emotional. Security tools default to sharp corners and hard edges, which read as institutional and slightly hostile — precisely the feeling that stops a non-technical recipient from typing a password. Generous radii make the same interface approachable without making it look unserious. The radius is the single largest contributor to the product feeling modern rather than corporate.

## Components

Full behavioral specifications — states, interaction, accessibility semantics — live in `EXPERIENCE.md`. This section owns appearance.

**`{components.button-primary}`** — solid `{colors.accent}`, white text, `{rounded.md}`, 15px at weight 600, full-width inside a form, minimum 44px tall. Exactly one is visible per screen. It carries the confirm-and-destroy action too: destruction is what the recipient came to do, so it is styled as the intended path, not discouraged with red.

**`{components.button-secondary}`** — indigo outline on transparent, washing to `{colors.accent-tint}` on hover. Alternatives beside a primary, or standalone low-stakes actions.

**`{components.button-ghost}`** — indigo text, no border, underline on hover. "How does this work?", "Create a new secret", the "Random" password generator.

**`{components.input}`** — full-width, 12–16px padding, hairline `{colors.border}`, `{rounded.md}`. Focus is an indigo border plus a 3px indigo glow at 10% opacity — visible enough to satisfy the keyboard-navigation floor without shouting. Placeholders in `{colors.text-tertiary}`.

**`{components.textarea}`** — the input treatment in `{typography.mono.fontFamily}`, minimum 120px, vertically resizable. The mono face is the cue that pasted content will survive intact.

**`{components.select}`** — matched to the input exactly, with a custom chevron so no native browser styling leaks in. Three options only.

**`{components.checkbox}`** — 18px, indigo when checked, inline with its label on one row.

**`{components.split-panel}`** — the sender page's left column: a 135° indigo gradient with white text carrying the tagline, the security model in one short passage, and a checkmark feature list. It exists to answer "what is this?" for a first-time visitor without competing with the form. Hidden entirely below 768px.

**`{components.secret-display}`** — `{colors.surface}` fill, hairline border, `{rounded.md}`, content in mono at 14px with `pre-wrap` and `break-all` so long tokens wrap instead of overflowing. A header row carries the label and the copy control.

**`{components.warning-banner}`** — the amber trio, `{rounded.md}`. Only at the pre-reveal confirmation and the post-reveal deletion notice.

**`{components.toast-success}`** — emerald with a checkmark, `{rounded.md}`, auto-dismissing. Used for "Copied to clipboard" and nothing else.

**`{components.status-icon-success}`** — a 64px emerald circle above the generated link. **`{components.status-icon-deadend}`** — a 56px `{colors.text-tertiary}` lock circle above the dead-end message: gray, not red, because the dead end is normal.

## Do's and Don'ts

**Do**

- Let whitespace carry the trust. When a screen feels thin, that is the design working.
- Keep exactly one `{components.button-primary}` per screen, and make it the obvious next step.
- Use `{typography.mono.fontFamily}` for every piece of user data — secret input, revealed secret, generated URL.
- Reserve `{colors.warning}` for irreversible destruction, and `{colors.success}` for completed confirmations.
- Render the dead-end screen in neutral grays with no error styling.
- Use type, weight and spacing to build hierarchy before reaching for a box, border or fill.
- Treat `backdrop-filter` as enhancement, with a solid-white fallback that is equally acceptable.

**Don't**

- Don't add shields, padlock badges, or "military-grade encryption" claims. Security theater costs trust.
- Don't use a dark or terminal aesthetic — black grounds, green mono. It alienates the non-technical recipient the product depends on.
- Don't ship default component-library styling. A generic look signals side project, which is fatal for a tool holding credentials.
- Don't introduce a second accent color, or use indigo for anything that is not an action.
- Don't use red for the dead-end screen, or for any state that is not an incorrect password or a validation failure.
- Don't add navigation, headers, sidebars or footers. There are three surfaces and no way to get lost.
- Don't apply the glass treatment to recipient surfaces. They are single-task screens where clarity beats material.
- Don't tighten the spacing scale to fit more in. If something does not fit, it does not belong.
- Don't animate for delight. Motion is functional only, and every transition honors `prefers-reduced-motion`.

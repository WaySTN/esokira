# Esokira — Design System (Mobile / Flutter)

> **Canonical path:** `docs/design/DESIGN-SYSTEM.md`  
> **Version:** `0.1.0`  
> **Date:** 30 October 2026  
> **Status:** `REVIEWED_INTERNAL — PENDING_OWNER_APPROVAL`  
> **Document type:** Visual/UI/UX design specification and implementation guidance; **not** a claim that components already exist  
> **Scope:** R0 application foundation, R1 MVP, R2 planning, R3 advanced simulation. R4 remains discovery only.  
> **Owner:** Esokira Product Owner; proposed design decisions need owner approval.  
> **Primary platform:** Flutter mobile, Android and iOS planned.  
> **Language and locale:** Indonesian / `id-ID`, IDR-first are **PROPOSED**, not a completed localization policy.

---

## 0. Purpose, authority, and rules for AI coding agents

Esokira is a **personal financial life planning and simulation application**. Its UI should help users distinguish *their current financial position* from *a plan*, *a projection*, and *a model-based risk estimate*. This document specifies how to display those distinctions consistently across Flutter screens and components.

The design direction is: **calm, precise, approachable, information-dense where necessary, and transparent about uncertainty**. Decorative polish must never obscure financial meaning.

### 0.1 Decision-status legend

| Status | Meaning | Agent behavior |
|---|---|---|
| `DECIDED` | Explicitly chosen by the Product Owner in the project discussions | Respect it; do not silently substitute alternatives. |
| `REQUIRED` | Invariant derived from the project product requirements, security, financial domain, or cross-platform accessibility | Must hold in every implementation; flag conflict before shipping. |
| `PROPOSED` | Recommended token, pattern, library approach, or UI architecture | Candidate for design review; **not** automatically production policy. |
| `TBD` | Product/design/technical decision not yet made | Document options and dependency; do not fabricate final policy. |
| `BLOCKED` | No production build until the prerequisite is settled or tested | Prototype only with obvious demo/synthetic labeling, never fake user values. |
| `OUT OF SCOPE` | Not committed for the relevant release | Do not implement as a completed feature. |

`REVIEWED_INTERNAL` means this draft was cross-checked against earlier Esokira documentation. It **does not imply Product Owner approval**. No Flutter UI, widget test, device screenshot, or production accessibility certification was performed to create this document.

### 0.2 Source of truth; do not duplicate ownership

| Question | Authoritative document / artifact |
|---|---|
| Why Esokira exists and whom it serves | [`../product/PRODUCT-VISION.md`](../product/PRODUCT-VISION.md) |
| Which features and acceptance criteria exist | [`../product/PRD.md`](../product/PRD.md), `FR-001`–`FR-024`, `NFR-001`–`NFR-010` |
| Release priorities and dependencies | [`../product/FEATURE-MATRIX.md`](../product/FEATURE-MATRIX.md) |
| Actual/planned/forecast meanings; financial invariants | [`../financial/FINANCIAL-DOMAIN-RULES.md`](../financial/FINANCIAL-DOMAIN-RULES.md), `DR-001`–`DR-024` |
| Formula definitions and calculation assumptions | [`../financial/CALCULATION-SPECS.md`](../financial/CALCULATION-SPECS.md), `CS-001`–`CS-016` |
| Numerical test vectors and invariants | [`../financial/FINANCIAL-TEST-CASES.md`](../financial/FINANCIAL-TEST-CASES.md) |
| System-wide trust boundaries | [`../architecture/SYSTEM-ARCHITECTURE.md`](../architecture/SYSTEM-ARCHITECTURE.md) |
| Go calculation and transaction responsibilities | [`../architecture/BACKEND-ARCHITECTURE.md`](../architecture/BACKEND-ARCHITECTURE.md) |
| Flutter app state, repository, caching, and retries | [`../architecture/FRONTEND-ARCHITECTURE.md`](../architecture/FRONTEND-ARCHITECTURE.md) |
| Schema ownership, actual vs scenario storage | [`../architecture/DATABASE-DESIGN.md`](../architecture/DATABASE-DESIGN.md) |
| Endpoint/request/response semantics | [`../architecture/API-GUIDELINES.md`](../architecture/API-GUIDELINES.md), later `contracts/openapi.yaml` after approval |
| Visual tokens, component behavior, content presentation | **This document** |
| Exact routes, final navigation hierarchy, screen-level interaction | Future `docs/design/SCREEN-SPECS.md` / `docs/design/USER-FLOWS.md`; **not yet asserted to exist** |

**Conflict protocol:** cite both affected document sections and requirement IDs; mark `CONFLICT`, propose a decision, and **stop only the affected scope**. Do not let a design component re-define an accounting, scenario, API, or authorization rule.

### 0.3 Non-negotiable implementation rules

1. **No fabricated financial data:** do not display example balances, goals, success rates, or charts as though they belong to a signed-in user.
2. **Actual ≠ planned ≠ forecast ≠ modeled:** use text/semantics labels in addition to visual distinctions.
3. **Go remains the source of financial calculation truth**. Flutter formats and interacts; it does not invent financial formulas, compute official net worth, or reinterpret server numerical outputs.
4. **Use shared semantic theme tokens**; no ad-hoc color, font size, radius, spacing, gradient, or shadow values in individual feature widgets without review.
5. **Accessibility is functional behavior**, not a late aesthetic step: scalable text, sufficient tap targets, focus order, screen readers, contrast, non-color cues, and chart alternatives.
6. **Financial actions require truthful states:** submitting, confirmed, rejected, or ambiguous outcome must not be conflated.
7. **No silent financial defaults:** absent income, rate, inflation assumption, or dates must remain missing/unknown unless explicitly specified by the approved calculation contract.
8. **Handle empty/loading/error/stale/permission/partial data** before calling a screen complete.
9. Never show production routes for `FR-023` or `FR-024` while they are `R4 — DISCOVERY`.
10. Do not claim a screenshot, golden test, widget, or build passed unless it was actually run against code.

---

## 1. Experience principles and anti-patterns

### 1.1 Five user-facing principles

| ID | Principle | Implementation consequence |
|---|---|---|
| `DS-P01` | **Clarity before decoration** | Primary number has label, unit, source, and time; decoration cannot compete with it. |
| `DS-P02` | **Explain the consequence** | Scenario/goal outputs show what changed and why, not only a predicted figure. |
| `DS-P03` | **Visible uncertainty** | Forecast assumptions and modeled risks are easy to inspect and never sound guaranteed. |
| `DS-P04` | **Calm by default** | Restrained colors, stable hierarchy, readable density; alert colors reserved for meaningful states. |
| `DS-P05` | **Progressive disclosure** | Basic view first; statistical detail, formulas, and model metadata behind explicit secondary actions. |

### 1.2 What Esokira must *not* look like

- Not a generic fintech dashboard built from eight oversized gradient cards and unrelated icons.
- Not every section in a glowing glass container with fake depth and neon financial graphs.
- Not a full screen filled with pie charts of arbitrary categories when a number, comparison, or short explanation would serve the task better.
- Not green = good / red = bad across every financial number. Negative net worth, falling debt, and lower spending have different meanings.
- Not a congratulatory “target achieved” animation based on **planned contributions** that were never actually recorded.
- Not fake activity, dummy goals, “AI insight”, portfolio returns, or success probabilities in user-facing production UI.
- Not four bottom tabs that duplicate the same dashboard content; not nested navigation that hides primary actions.
- Not ambiguous technical jargon (“P90”, “drawdown”, “inflation adjusted”) without an explanation path.

### 1.3 Information hierarchy contract

On a financial screen, arrange information in this order (adapt to task):

1. **Context:** what object/timeframe/source is being viewed?
2. **Primary answer:** one clearly identified number, status, or decision.
3. **Comparison:** relevant prior state, baseline, target, or alternative.
4. **Explanation:** key drivers, assumption summary, and important risks.
5. **Action:** relevant next step that the user can deliberately choose.
6. **Advanced details:** formulas, breakdown, model/run identifiers, optional data table.

If the primary answer needs more than two disclaimer paragraphs to be interpreted safely, redesign the presentation rather than hiding disclaimers under a tiny information icon.

---

## 2. Brand direction (`PROPOSED`)

**Working name:** Esokira. **Working tagline:** “Hitung kemungkinan. Rancang masa depan.” Neither the name nor its commercial availability is certified by this design document.

**Brand attributes:** foresight, practicality, human scale, trust, analytical clarity, subtle optimism. Indonesia-first, not an imitation of bank compliance dashboards.

**Visual direction:** dark midnight navy for identity and deep teal for action; neutral off-white surfaces for reading; restrained blue and violet for distinct data sources and scenarios. Use whitespace and typographic hierarchy instead of gradients as the main design tool.

**Logo / mark concept** (idea, not an asset): intersecting paths or trajectories converging on a milestone; avoid coins, dollar signs, upward arrows, generic wallets, and AI sparkles as primary brand glyphs. A logo must be designed, legally checked, and exported separately before use.

**Brand graphic rules:** any illustration on onboarding/empty states must teach a concrete concept, use a restrained stroke system, and be optional to understanding the task. Do not block product development on unapproved illustrations. **No logo files or font files are included in this Markdown package.**

---

## 3. Semantic color system (`PROPOSED`, pending visual approval)

### 3.1 Token philosophy

Use **semantic roles**, not raw palette values in widgets. A widget asks for `color.text.primary`, `color.surface.card`, or `color.chart.actual` and receives the correct value for light/dark mode. Flutter `ColorScheme`/`ThemeData` is the foundation; custom financial source/status/chart roles can live in a well-defined theme extension after design approval.

A chart series color is **not equivalent** to a semantic success or error state. Likewise, a net worth card is not necessarily a success color merely because the value is positive.

### 3.2 Proposed light-mode tokens

| Semantic token | Hex | Use |
|---|---|---|
| `color.bg.app` | `#F7F9F8` | Main application background. |
| `color.surface.default` | `#FFFFFF` | Cards, dialogs, raised controls. |
| `color.surface.muted` | `#EFF3F2` | Group sections, subtle chips, side-by-side comparisons. |
| `color.text.primary` | `#172E35` | Headlines, amounts, high-priority labels. |
| `color.text.secondary` | `#425A61` | Subtitles, metadata, explanatory text. |
| `color.border.default` | `#CCDAD7` | Dividers, outline controls; non-text contrast depends on adjacency. |
| `color.action.primary` | `#0D5B50` | Primary CTA and selected control. |
| `color.action.onPrimary` | `#FFFFFF` | Content on primary action. |
| `color.state.successText` | `#176344` | Explicit achieved/confirmed statuses only. |
| `color.state.warningText` | `#805500` | Caution, stale, insufficient confidence when appropriate. |
| `color.state.errorText` | `#A32421` | Failure, destructive operations, validation errors. |
| `color.chart.actual` | `#115C86` | Actual/historical series with solid line. |
| `color.chart.forecast` | `#0D6E59` | Future projection with dashed line. |
| `color.chart.scenario` | `#7148A4` | Alternative version / comparison with dotted line. |

### 3.3 Proposed dark-mode tokens

| Semantic token | Hex | Use |
|---|---|---|
| `color.bg.app` | `#0C171C` | Main background. |
| `color.surface.default` | `#14252A` | Primary card/dialog surface. |
| `color.surface.muted` | `#1B3237` | Grouped/elevated visual area. |
| `color.text.primary` | `#F4FAF8` | Primary reading foreground. |
| `color.text.secondary` | `#BCD3D0` | Supporting text. |
| `color.border.default` | `#395157` | Subtle boundaries; verify against final surfaces. |
| `color.action.primary` | `#79DCCA` | Primary interactive accent. |
| `color.action.onPrimary` | `#092D2A` | Foreground on primary action. |
| `color.state.successText` | `#81DEB2` | Confirmed success. |
| `color.state.warningText` | `#FBCB75` | Caution and stale indicators. |
| `color.state.errorText` | `#FFB4A9` | Error and destructive action indicators. |
| `color.chart.actual` | `#8AC8FF` | Actual/historical solid series. |
| `color.chart.forecast` | `#7ADECE` | Forecast dashed series. |
| `color.chart.scenario` | `#C5A3FF` | Comparison dotted series. |

**Do not derive the entire dark palette by inverting light colors.** Surfaces, borders, contrast, charts, and elevation need independent checks.

### 3.4 Contrast evidence and caveats

The following foreground/background ratios were calculated using WCAG relative luminance, rounded to two decimals. This audit verifies **the listed pairs only**; it is not a full Flutter visual audit.

| Foreground / background | Contrast | Reference |
|---|---:|---|
| Light primary text / light app background | `13.43:1` | Text pass |
| Light secondary text / light app background | `6.93:1` | Text pass |
| Light primary action / white | `7.99:1` | Foreground pass for this pairing |
| White CTA text / light primary action | `7.99:1` | CTA text pass for this pairing |
| Dark primary text / dark app background | `17.21:1` | Text pass |
| Dark secondary text / dark card surface | `10.07:1` | Text pass |
| Dark primary action / dark card surface | `9.72:1` | Foreground pass for this pairing |
| Dark CTA text / dark primary action | `9.08:1` | CTA text pass for this pairing |
| Light actual / white | `7.24:1` | Chart line contrast pass in this context |
| Light forecast / white | `6.19:1` | Chart line contrast pass in this context |
| Light scenario / white | `6.63:1` | Chart line contrast pass in this context |
| Dark actual / dark card | `8.88:1` | Chart line contrast pass in this context |
| Dark forecast / dark card | `9.92:1` | Chart line contrast pass in this context |
| Dark scenario / dark card | `7.57:1` | Chart line contrast pass in this context |

WCAG 2.2 AA reference targets: normal text ≥ `4.5:1`, large text ≥ `3:1`, and essential non-text graphical/UI information ≥ `3:1`. See [W3C WCAG 2.2](https://www.w3.org/TR/WCAG22/) and [Flutter accessibility guidance](https://docs.flutter.dev/ui/accessibility). Check states, neighboring surfaces, overlays, disabled/read-only controls, chart labels, and theme blends **after implementation**.

### 3.5 Status-specific visual language

| Meaning | Text badge / label | Optional visual cue | Never use as sole cue |
|---|---|---|---|
| Posted, confirmed financial data | `Aktual` / `Tercatat` | Solid border or series | Blue alone |
| Goal plan or intended contribution | `Rencana` | Outlined tag or square marker | Gray alone |
| Projection or financial model | `Proyeksi` | Dashed series | Teal alone |
| Compared scenario | `Skenario B` | Dotted line + scenario name | Purple alone |
| Stale calculation | `Perlu diperbarui` | Warning icon + explanatory message | Amber alone |
| Simulation running | `Sedang dihitung` | Progress state without invented % | Spinner alone |
| Insufficient information | `Data belum cukup` | Checklist/next-action | Blank values / zero |
| Financial shortfall | `Kekurangan dana` | Negative amount + explanation | Red alone |

No brand color can supersede the `actual vs hypothetical` labeling requirement.

---

## 4. Typography and numeric readability (`PROPOSED`)

### 4.1 Font strategy

Prefer platform-safe system fonts for R0/R1: Android system UI (typically Roboto) and iOS system UI (San Francisco), letting Flutter/native stacks handle fallback. A bundled brand typeface is **TBD** and requires license, localization, and build-size review. Do not insert unlicensed font binaries into the repo. Test Indonesian diacritics, numeric separators, and long dynamic labels.

Typography uses a small number of roles, not a different font size per screen:

| Role token | Base logical sp | Weight | Example |
|---|---:|---|---|
| `type.display` | 32 | 600–700 | Net worth figure or primary simulation result; use sparingly. |
| `type.headline` | 24 | 600 | Screen title / key goal milestone. |
| `type.title` | 20 | 600 | Main section heading. |
| `type.subtitle` | 16 | 600 | Card heading / context name. |
| `type.body` | 15–16 | 400 | Reading text, paragraphs, forms. |
| `type.label` | 14 | 500–600 | Field labels, main buttons, data headers. |
| `type.caption` | 12–13 | 400–500 | Metadata, timestamps, qualifiers — not for essential risk caveats. |

Token sizes are an initial visual proposal; text scaling should not be disabled to preserve these numbers. Avoid 10px mini labels on crowded financial charts.

### 4.2 Monetary number typography

- Use **tabular figures** where available for columns, balances, comparisons, and animated counters; avoid text jitter.
- Separate currency/amount/source/time through hierarchy: e.g., `Total kekayaan bersih` → `Rp48.500.000` → `Aktual · Per 28 Okt 2026` (number/date is **synthetic UI example**, not personal data).
- Negative amounts show an explicit minus and optionally a category label; no color-only negatives.
- Large numbers must not be clipped: use responsive text wrapping, alternate line layout, or a clearly marked compact notation with full-value accessible label.
- Align numeric table columns to the right in left-to-right layouts; keep decimal units comparable.
- Do not use bold on every monetary figure. Reserve emphasis for the current screen's central figure.
- Respect user text scaling; where a number cannot fit, allow vertical expansion rather than silently reducing its semantics.

### 4.3 Line lengths and whitespace

Mobile paragraph copy should be concise. Plan for Bahasa Indonesia strings 20–40% longer than abbreviated English equivalents. Favor readable line height (~1.4–1.6 for body as an initial design range), consistent paragraph spacing, and natural multi-line wrapping. Text styling must not prevent selection/copy when the user needs a reference number.

---

## 5. Spacing, shape, borders, and elevation (`PROPOSED`)

### 5.1 Spacing scale (logical pixels)

| Token | Value | Usage |
|---|---:|---|
| `space.0` | 0 | Intentional flush alignment. |
| `space.1` | 4 | Icon/inline micro-gap. |
| `space.2` | 8 | Metadata pairs, tight list items. |
| `space.3` | 12 | Form label helper separation. |
| `space.4` | 16 | Standard internal card padding. |
| `space.5` | 20 | Section spacing in narrow mobile. |
| `space.6` | 24 | Screen horizontal inset, major component gap. |
| `space.8` | 32 | Separate major sections or visual groups. |
| `space.10` | 40 | Hero visual and primary action breathing room. |
| `space.12` | 48 | Rare larger separators / illustration region. |

Use a 4 px baseline and an 8 px structural rhythm without requiring every number to be divisible by eight. Compact dense tables may use 8–12 px vertical cell padding if tap targets remain adequate.

### 5.2 Shape and elevation

| Token | Value | Usage |
|---|---:|---|
| `radius.xs` | 6 | Field elements and very small chips. |
| `radius.sm` | 10 | Inputs, list selection, small surfaces. |
| `radius.md` | 14 | Typical cards and sheets. |
| `radius.lg` | 20 | Large modal or featured planner content. |
| `radius.pill` | Full | Status/filter chips only when semantically appropriate. |
| `border.subtle` | 1 px | Card/data section separation if background alone is not enough. |
| `elevation.default` | 0 | Most screens and cards. |
| `elevation.overlay` | Small, verified | Modal/sheet/menu over existing content. |

Use **borders and tone separation before shadows**. Avoid decorative hard-edged drop shadows, glass blur and excessive gradient fills. Sticky bottom action bars must not obscure keyboard, scrollable form fields, focus, or safe-area content.

---

## 6. Layout and adaptive behavior

### 6.1 Mobile first, not one-phone-only

Suggested **layout breakpoints** are visual-development candidates, not universal device specifications:

| Width in logical px | Layout expectation |
|---|---|
| `<360` | Narrow handset: one column, short labels, optional secondary actions in overflow, no horizontal overflow. |
| `360–599` | Default handset: one column, bottom navigation if approved, standard screen insets. |
| `600–839` | Large mobile/foldable/tablet narrow: allow two-column composition for meaningful comparisons; adapt nav if justified. |
| `≥840` | Expanded view: consider navigation rail and two-pane forms/charts, never force a stretched 800 px card. |

These are **proposed presentation buckets**, not code constants until verified with supported Flutter layouts and target device matrix (`DS-Q06`). Layout must also be tested with system navigation insets, notches, keyboard, split-screen, text scaling, and orientation.

### 6.2 Screen scaffolding

A typical authenticated screen includes (only when useful): app-level navigation, contextual title, source/time metadata, primary content, contextual action(s), state handling, and persistent system-safe bottom inset. Avoid putting a FAB and a sticky primary CTA on the same screen without a clear hierarchy.

Screen content max width on larger viewports is a design choice to be validated. Avoid full-width lines of dense financial explanatory text in tablet landscape.

### 6.3 Layout principles

- At a glance, users should see one dominant task and/or meaningful number, not a 6-card marketing dashboard.
- Separate **read** views (summary, comparison) from **write** views (edit transaction, commit change), reducing accidental modification.
- Dense data tables may scroll horizontally only if there is a clear affordance, pinned key column if implemented, and a legible accessible alternative.
- Prefer skeletons only where predictable layout improves perceived continuity; never place ghost financial values that could be mistaken for real data.
- If a goal card needs five metrics, allow a details screen rather than shrinking five captions into illegible text.

---

## 7. Navigation, information architecture, and release boundaries (`PROPOSED`)

### 7.1 Candidate primary navigation

**Do not finalize routing here.** `FRONTEND-ARCHITECTURE.md` governs secure route behavior; `USER-FLOWS.md` and `SCREEN-SPECS.md` will determine exact screen hierarchy. Candidate bottom destinations for later product review:

| Area | Main jobs | Earliest release |
|---|---|---|
| `Beranda` | Actual financial overview, changes, and next recommended action | R1 |
| `Keuangan` | Accounts, transactions, assets, liabilities | R1 |
| `Tujuan` | Financial goals, feasibility, target details | R1 |
| `Rencana` | Life events, projections, scenario comparisons | R2 |
| `Profil` or app overflow | Settings, security, export/delete | R1 |

Do **not** include disabled fake `Simulasi`/`Financial GPS` tabs in R1 as though they exist. R3 analytical features can live in `Rencana` or another reviewed navigation structure when available. Select destinations after task testing; navigation count is not an absolute requirement.

### 7.2 Navigation behavior

- Back navigation must not silently discard an unsaved financial edit. Provide a clear “Buang perubahan?” confirmation if needed.
- Deep links to private resources require authenticated access and server-side ownership checks; do not expose names/amounts in unauthorized fallback screens.
- An interrupted create/edit flow should return the user to an intelligible state without manufacturing a completed transaction.
- Selected tabs and filters may persist as non-sensitive presentation preferences according to frontend storage policy.
- Notification entry points should open a safe generic screen when access/session changed; notifications must not contain sensitive amounts by default.

---

## 8. Component inventory and composition rules

### 8.1 Foundation components (R0)

| Candidate component name | Purpose | Mandatory states / semantics |
|---|---|---|
| `EsAppScaffold` | Consistent safe area, navigation, background | Protected/guest contexts; no dead navigation controls. |
| `EsSectionHeader` | Title, summary, optional visible action | Heading hierarchy, accessible action label. |
| `EsPrimaryButton` | Main submit/continue action | Enabled, disabled with reason, loading, success/failure controlled by caller. |
| `EsSecondaryButton` | Secondary action | Correct priority; no duplicate primary action. |
| `EsTextField` | General input | Label, optional, helper, validation, focus/error. |
| `EsAmountField` | Financial amount entry | Exact input, currency, localized keyboard/parse policy, validation; **serialization TBD**. |
| `EsDateField` | Effective date / due date | Locale, timezone semantics, min/max per domain. |
| `EsStatusLabel` | Actual/plan/forecast/queued/stale status | Text and semantics in addition to color. |
| `EsAsyncContent` | Repository-backed content states | Loading, empty, error, partial, stale, content. |
| `EsConfirmationSheet` | Irreversible/high-impact action prompt | Explicit action, consequences, focus order, cancel. |
| `EsCurrencyText` | Exact monetary view model | Currency, sign, status/time, copy, scale. |
| `EsInfoDisclosure` | Explain an assumption, definition, or warning | Accessible text and dismissal. |

Names are **suggested architecture patterns**, not a mandate to create a widget for every line on day one.

### 8.2 Financial components (R1)

| Candidate | Use | Key guardrail |
|---|---|---|
| `AccountRow` | Account balance and category | As-of/status; not confused with external asset valuation. |
| `TransactionRow` | Signed amount, date, source/type | Internal transfer has separate semantics; not duplicated as expense. |
| `NetWorthSummary` | Assets minus liabilities | No double counting; partial data must be disclosed. |
| `GoalProgressCard` | Target and confirmed funding | Distinguish actual funded vs planned contributions; `Q-11` remains unresolved. |
| `ContributionEstimate` | Required vs available set-aside | Explain shortfall and assumptions; no automatic target mutation. |
| `CompoundResultPanel` | Initial amount, contributions, estimated gains, horizon | Rate basis, timing, inflation/fees assumptions explicit. |
| `FormErrorSummary` | Validation and failed mutations | Navigation/announce errors; preserve input. |

### 8.3 Planning and advanced components (R2/R3)

| Candidate | Use | Key guardrail |
|---|---|---|
| `LifeEventTimeline` (R2) | Planned milestones with effect type | Events are plans, never automatically posted transactions. |
| `ScenarioSelector` (R2) | Baseline/version selection | Always show scenario identity and last-calculated metadata. |
| `ScenarioComparison` (R2) | Show two or more consistent projections | Same unit, horizon, baseline compatibility; explain differences. |
| `FinancialLineChart` (R2) | Past vs future | Date/amount labels, solid/dashed series, accessible table. |
| `StressTestPanel` (R2) | Shock input and cash resilience | State shock assumptions and limitations. |
| `MonteCarloDistribution` (R3) | Percentiles and model outcomes | No unvalidated probability; model/version/seed/run count surfaced. |
| `GoalAllocationExplorer` (R3) | Allocation options and infeasible shortfall | Objective & constraints readable, proposed changes never silently posted. |
| `RetirementRunway` (R3) | Withdrawal/depletion simulation | Nominal vs real and decumulation assumptions clear. |
| `FinancialGPSAlternatives` (R3) | Compare suggested paths | Transparent trade-offs, never command/guarantee. |

Avoid an “omnicomponent” that accepts dozens of boolean props to behave as every card. Prefer composition of simple, focused, testable primitives.

### 8.4 Component state specification template

Every reusable component intended for release should document:

```text
Component: <name>
Owner / feature: <feature>
Purpose / supported data: <...>
Input semantic types: <...>
States: default | focus | disabled | loading | error | empty | stale (as applicable)
Action: <what it actually triggers>
Accessibility: semantics label, focus and reading order, tap target
Responsive: narrow / standard / wide behavior
Financial source/time meaning: actual / planned / forecast / modeled
Dependencies: FR, DR, CS, API field(s)
Tests: widget, golden, semantics, edge cases
```

For a component that displays a financial number, the semantic type (actual/forecast/plan) must be an explicit input rather than inferred from the visual style.

---

## 9. Input forms, money, dates, and rate handling

### 9.1 Exact monetary input

- UI may show localized separators while editing, but must preserve the **exact intended amount** through parsing and API serialization.
- Backend/API monetary representation is pending `Q-04`, `DB-Q02`, `API-Q03`; **do not** implement production serialization with `double` or assume fractional-IDR rules.
- Do not strip negative sign silently; permit/disallow negatives according to the specific domain field.
- Display currency unit within reach of the input label (e.g., `Nominal (IDR)`) and error messages.
- Distinguish an empty field from `0`. An absent value must not become a zero rate or zero balance by default.
- For preview, show normalized input below or on blur after verified parsing. Never normalize to a changed value without making it visible.
- Avoid confusing thousands separators with decimal separators when pasting (`1.250.000`, `1250000`, locale variants); ambiguous strings must be rejected with guidance rather than guessed.
- Never print the user's amount in application logs, analytics payloads, or crash breadcrumbs.

### 9.2 Rates and percent fields

- Field label must include the period and basis, such as `Estimasi imbal hasil efektif per tahun (%)` when that is the active approved formula basis.
- A UI slider is optional, **not** the sole control; provide a precise editable field, keyboard accessibility, and clear min/max/step.
- Input validation must not invent “typical” investment returns. Defaults require explicit design/domain approval, label as assumptions, and user review.
- Show a warning when the selected horizon or assumptions make extrapolation misleading; warning copy must be based on reviewed policy.

### 9.3 Dates and time

- Label **tanggal efektif**, **tanggal dibuat**, **terakhir diperbarui**, and **tanggal proyeksi** distinctly.
- Store/render dates according to the API contract; device local time is not automatically the economic effective date.
- Avoid ambiguous `10/11/26`; prefer an explicit Indonesian date such as `10 November 2026` for key actions.
- Long-horizon scenarios should present years and meaningful periods, not hundreds of tiny month labels.

### 9.4 Form interaction pattern

1. Present field meaning and optionality.
2. Accept and validate locally without claiming server acceptance.
3. Preserve user input when an API request fails.
4. Submit with a meaningful label (`Simpan tujuan`, `Catat transaksi`, `Hitung estimasi`).
5. Disable duplicate tap during in-flight, **and** rely on server idempotency for financial mutation.
6. On ambiguous outcome, show “Menunggu konfirmasi”, not “Gagal” or “Berhasil” until server confirms.
7. On confirmed mutation, update/invalidate relevant cached views using the frontend state policy.

### 9.5 Example validation copy

| Situation | Preferred copy | Avoid |
|---|---|---|
| Missing target | `Masukkan nominal target terlebih dahulu.` | `Invalid input.` |
| Missing calculation rate | `Masukkan asumsi imbal hasil untuk menghitung proyeksi.` | Silently assuming a market return. |
| Unparseable amount | `Format nominal belum dikenali. Contoh: 1.250.000.` | Saving `0`. |
| Goal shortfall | `Kebutuhan setoran melebihi dana yang tersedia.` | `Tujuan gagal selamanya.` |
| Unknown mutation result | `Permintaan sudah dikirim. Kami sedang memeriksa hasil pencatatannya.` | `Transfer gagal, coba lagi` with a new idempotency key. |
| Expired sign-in | `Sesi berakhir. Masuk kembali untuk melanjutkan.` | Losing unsaved non-sensitive draft without warning. |

---

## 10. Financial semantics and display contracts

### 10.1 Display model (conceptual; not an API schema)

```text
FinancialValueView {
  exactValue: <type agreed by API/DB>
  currency: ISO currency code
  economicSource: actual | planned | forecast | modeled
  asOfOrPeriod: <time / horizon>
  approximation: exact | rounded-for-display | estimated
  scenarioRef?: <scenario/version>
  assumptionsRef?: <approved source or result metadata>
}
```

The UI model is a **presentation adapter**, not the authoritative API shape; types and fields are placeholders until approved OpenAPI. Display source, period, and unit must not be reconstructed from a screenshot or hardcoded widget logic.

### 10.2 Required display labels

| Data class | Minimum visible meaning | Example label; illustrative only |
|---|---|---|
| Actual | It has been recorded; as-of date if relevant | `Aktual · Terakhir diperbarui 28 Okt 2026` |
| Planned | A user's intention, not a funded account | `Rencana · Belum tercatat sebagai saldo aktual` |
| Forecast | Calculated future state, assumptions and horizon | `Proyeksi · 10 tahun · Asumsi dapat dilihat` |
| Modeled risk | Distribution-dependent metric and model context | `Berhasil dalam 720 dari 1.000 simulasi model` |
| Stale | Baseline or assumptions have since changed | `Perlu diperbarui · Berdasarkan data sebelumnya` |
| Partial actual | Known inputs not complete | `Ringkasan belum lengkap · Tambahkan data aset/utang` |
| Pending write | Server has not yet confirmed | `Menunggu konfirmasi pencatatan` |

**Never** use a model percentile as a confirmed bank balance. Never call a model success frequency “peluang pasti”. Never use planned allocation as a funded amount without an approved and recorded funding definition (`Q-11`).

### 10.3 Precision and compact display

- Formatting is display-only. Do not round before calculations or substitute the rounded text into a subsequent API request.
- When showing compact `Rp1,2 jt`, provide full exact value in an accessible label, detail view, or on user action; avoid on-screen ambiguity of `jt` with date/timeline context.
- Distinguish **nominal** from **nilai riil (setelah inflasi)** in labels, legends, and export data. A toggle must select real calculated series; changing text alone is forbidden.
- When a negative value is possible, preserve its sign, semantic label, and appropriate explanatory copy.
- On graphs, separate data precision (calculation) from tick precision (visualization) and document the chosen formatting rules.

---

## 11. Financial charts and data visualization

### 11.1 Use the simplest effective visual

| User question | Preferred presentation | Avoid |
|---|---|---|
| How much money do I have now? | Number + components + timestamp | Unnecessary pie chart. |
| What changed this month? | Month comparison + signed delta + context | Random growing animation. |
| Am I on track to reach a goal? | Progress indicator + remaining amount + date | Progress based on unposted planned savings. |
| What does a contribution schedule look like? | Line or stacked area for principal vs estimated gains, plus labels | Color area without accessible legend. |
| Which life event changes cash flow? | Timeline + before/after comparison | 3D timeline decoration. |
| Which scenario is better? | Common-axis line chart + summary comparison table | Different y-axis scales that exaggerate differences. |
| How variable are results? | Percentile band or distribution + assumptions + model caveat | Single certain-looking number. |
| What should I do next? | Ranked alternatives with trade-offs and constraints | “AI says invest now” card. |

### 11.2 Actual vs forecast chart language

- Actual history: solid stroke and `Aktual` legend with visible cut-off date.
- Forecast: dashed stroke and `Proyeksi` legend; starts explicitly at the chosen baseline.
- Alternative scenario: dotted or dash-dot stroke, unique readable scenario title.
- Show transition from actual to future using a labeled vertical marker such as `Awal proyeksi` (render only when time semantics valid).
- Support chart summary text and alternative data table, not just color or fine-grained pointer hover.
- Do not interpolate across missing actual data as if it were observed.

### 11.3 Axis, units, and interaction

- Time axis uses honest spacing; avoid categorical positions when time intervals are materially unequal.
- Amount axis shows unit, currency, and whether real/nominal; zero baseline is appropriate for bars but not always for financial line comparisons, provided the chosen scale is made clear.
- Tooltip: exact period, value, source label, and relevant scenario; no ambiguous bare number.
- Highlight a selected point using both marker/shape and text value. Ensure accessibility for screen reader and keyboard equivalents where applicable.
- Charts with two competing y-axes are discouraged because they are easy to misread.
- Avoid excessive grid lines, smooth-spline overshoot that depicts impossible negative balances, 3D perspectives, and marketing color gradients.
- When a screen has dense data, provide filter, interval selector, or secondary screen instead of shrinking the figure until illegible.

### 11.4 Monte Carlo visualization (`R3`, `BLOCKED` until model approval)

Minimum contents for a model result display:

1. Metric and definition, e.g., goal met at horizon according to the selected model.
2. Number of simulation paths and evaluated success count, if applicable.
3. Percentiles with explicit convention (which tail is P10/P90), horizon, currency, and nominal/real status.
4. Model return/volatility assumptions and whether a normal, bootstrap, or other sampling method is used; cannot be fabricated by frontend.
5. Baseline version, engine version, and result freshness; seed/details accessible through advanced disclosure when supported.
6. Warning that results depend on assumptions and do not guarantee market outcomes.

Avoid calling a `P10–P90` band a statistical confidence interval if it is a **distribution of simulated outcomes** rather than a confidence interval for an estimated parameter. Avoid implying “90% confidence” unless the relevant validated model and definition justify it.

### 11.5 Chart states and empty data

- `loading`: structural loading; do not show fabricated axes or value series.
- `empty`: explain why no data exists and present an appropriate setup action.
- `partial`: indicate which series or periods lack reliable observations.
- `stale`: keep historical result available with a non-dismissive warning, if backend permits; do not silently rewrite a prior calculation.
- `failed`: display error and safe retry only when the request semantics allow it.
- `blocked model`: show feature unavailable/exploratory; no fake Monte Carlo demo on actual profile data.

### 11.6 Accessibility alternative

Each meaningful chart must have a concise screen-reader summary (e.g., “Proyeksi naik dari ... ke ... dalam horizon ... berdasarkan asumsi ...”), plus a structured alternative representation of important data points. Do not build the summary by estimating numbers from plotted pixels; use actual result data.

---

## 12. Goals, timelines, scenarios, and Financial GPS

### 12.1 Goal progress (`R1`)

An ideal goal card displays: goal name, target, deadline, funded amount **only if its source and domain rule are approved**, gap, and one clear next action. Planning contribution estimates belong in a separate section and must not inflate funded progress.

**Progress definition policy (`Q-11`, `BLOCKED`):** until goal-to-account funding is defined and approved, render goal target/date and a separate “Perkiraan kebutuhan setoran”; do not render a funded progress percentage by guessing an allocation.

### 12.2 Life events (`R2`)

Use a time-ordered list or timeline with event label, year/age if provided, amount/range, assumptions, and effect category. Display the word `Rencana` and clearly distinguish an event from an actual transaction. Moving a proposed house purchase earlier changes only the scenario, never the recorded bank balance.

### 12.3 Scenario creation/comparison (`R2`)

The comparison view should show both scenario names, common starting point, period, selected differences, result metrics, and baseline compatibility. Where scenarios cannot be compared fairly, show the incompatibility rather than forcing a side-by-side chart.

A `Recalculate` action creates or requests a new result/version according to the backend contract; do **not** overwrite historically meaningful versioned results in-place if reproducibility requires retaining them.

### 12.4 Financial GPS (`R3`)

Present **alternatives**, not instructions or a hidden autonomous action. Each alternative states what changes, projected effect, key risk/trade-off, and which assumption produced the result. A “Terapkan pada skenario” action (if later approved) must change only the planning scenario; an actual transfer or financial account edit is always a separate explicit operation.

A recommendation is not an investment endorsement. Avoid “Pasti berhasil” and “Sistem menjamin”; prefer “Dalam skenario ini ...” and “Coba bandingkan ...”.

---

## 13. Asynchronous jobs and uncertainty-safe feedback (`R3`)

The UI may be built around the conceptual state model in `FRONTEND-ARCHITECTURE.md`, not a fabricated server contract:

```text
Draft -> Submitting -> Queued -> Running -> Succeeded
                       |          |        |
                       |          +------> Failed
                       +-----------------> Cancelled (if backend supports it)
```

**Rules:**

- If backend returns only `queued/running`, show an indeterminate progress indicator, not fictional completion percentages.
- Use `simulation_run_id`/request identity only according to the eventual OpenAPI contract. Never synthesize an ID that looks server-issued.
- If the app closes/reopens, resume by server status when the API supports it; do not resubmit a new Monte Carlo job blindly.
- `Failed` includes a comprehensible reason and a retry action only if safe under job idempotency policy.
- `Succeeded` shows results with model metadata, not a success celebration for personal finances.
- If a result is stale because actual data changed, show provenance and a deliberate recalculation action.
- Do not promise “Your forecast is ready” in push notifications until the backend and notification/privacy policies are implemented and tested.

---

## 14. Interaction motion, haptics, and feedback (`PROPOSED`)

**Motion is functional:** it should confirm navigation, reveal new information, or explain changes in a chart. Avoid gratuitous looping money animations, spinning finance globes, ticker counters, and attention-grabbing parallax on financial input screens.

Candidate durations (not hard technical requirements): micro state change `100–180 ms`; sheet/navigation transition `180–280 ms`; chart transition `200–350 ms`. Follow system preferences for reduced motion where supported; no essential meaning may exist only in an animation.

Haptics are optional and subtle for explicit confirmation, not when showing potentially alarming modeled results. Dangerous/destructive actions require a visible confirmation and result state, not only a vibration.

For balances and simulation numbers, default to **static accurate rendering**; do not count up from zero to the amount in a way that could appear as financial growth or transient wrong values.

---

## 15. Content design and Bahasa Indonesia-first localization

### 15.1 Voice and tone

Use plain, factual, nonjudgmental Indonesian. Speak about **choices and trade-offs**, not moralizing spending habits. Avoid slang in risk/permission/transaction screens even if the broader brand is friendly. Default to short headings with explanatory supporting text.

| Topic | Preferred | Avoid |
|---|---|---|
| Projection | `Estimasi nilai dalam 10 tahun` | `Uangmu pasti menjadi ...` |
| Goal feasibility | `Target membutuhkan setoran lebih besar dari kemampuan saat ini.` | `Kamu gagal mengatur uang.` |
| Current balance | `Aktual · Per 30 Okt 2026` | `Total pasti kamu` without data basis. |
| Monte Carlo | `Berhasil dalam 720 dari 1.000 simulasi model` | `Peluang 72% kamu akan kaya` |
| Stale results | `Hasil ini memakai data sebelum perubahan terakhir.` | `Hasil valid` without baseline info. |
| Privacy | `Data ini digunakan untuk membuat perhitunganmu.` | `Kami memakai AI untuk semua hal` |
| Destructive action | `Hapus akun dan data` plus consequences | `Reset` for irreversible deletion. |
| Uncertain transfer | `Status pencatatan belum dapat dipastikan.` | `Coba kirim lagi sekarang` |

Example numerical values and dates in this table are **synthetic**. The production app must use values from actual domain results only.

### 15.2 String resource policy

- All production labels/errors/help text belong in localization resources, not repeated inline literals in dozens of widgets.
- Proposed locale baseline: `id-ID`; English option is a future localization decision, not assumed in R1.
- Use locale-aware formatter for `Rp`, grouping, decimal/rate, dates, and plurals, but **never** use rendered text as the amount sent back to the backend.
- “Juta” and “miliar” may be abbreviated in axis ticks only; full amounts remain available in details and accessibility labels.
- Translation must preserve distinction between `estimasi`, `proyeksi`, `rencana`, `tercatat`, `aset`, `kewajiban`, and `kekayaan bersih`.

### 15.3 Help and financial literacy

Add optional inline explanations for compound interest, inflation, safe withdrawal assumptions, percentiles, and Monte Carlo. Help must not drown the screen or masquerade as personalized financial advice. Long-form education is not an R1 release requirement unless PRD specifies it.

---

## 16. Accessibility baseline and test policy (`REQUIRED`)

Use **WCAG 2.2 AA targets** as a practical accessibility baseline where applicable, while honoring native Flutter/platform accessibility expectations. WCAG is not, by itself, a certification of the mobile app.

### 16.1 Minimum interaction criteria

| Criterion | Required behavior |
|---|---|
| Contrast | Normal text ≥4.5:1, large text ≥3:1; essential non-text ≥3:1 for the actual background/state. |
| Tap targets | Aim for Flutter guidance **at least 48×48 logical px** for tappable controls; on iOS review platform guidance as well. |
| Text scale | No clipped amounts, unreachable action, broken modal, or horizontal text overflow at large accessibility scaling. |
| Screen reader | Test TalkBack and VoiceOver for screens, dialogs, navigation, forms, table/graph summaries. |
| Focus | Logical traversal; errors and destructive confirmations announced and visibly focused appropriately. |
| Non-color cues | Shape, stroke, text, icon, and status labels accompany color meaning. |
| Inputs | Visible associated labels and correction hints; placeholders are not labels. |
| Charts | Text summary + data alternative; no required tiny drag gesture to read values. |
| Motion | Respect reduced-motion setting where supported; no meaning only in animation. |
| Dynamic layout | Safe areas, keyboard, small screens, landscape, and high scaling all remain usable. |

### 16.2 Financial screen-reader example

**Avoid:** “Forty-eight million five hundred thousand” with no currency/source.  
**Prefer conceptual semantics:** “Total kekayaan bersih, empat puluh delapan juta lima ratus ribu rupiah, data aktual, diperbarui [date], detail tersedia.”

Do not turn unverified data into a spoken confidence statement. Semantics must use actual result metadata, not strings guessed by widget layout code.

### 16.3 Accessibility regression matrix

For each critical R1 screen, verify: narrow device + large text, light/dark contrast, keyboard focus, TalkBack/VoiceOver navigation, form error announcement, logout/relogin behavior, and missing-data states. For R2/R3 add chart alternatives and probabilistic model caveat comprehension.

**References:** [Flutter Accessibility](https://docs.flutter.dev/ui/accessibility), [Flutter UI accessibility styling](https://docs.flutter.dev/ui/accessibility/ui-design-and-styling), [WCAG 2.2](https://www.w3.org/TR/WCAG22/).

---

## 17. Light/dark themes and theme switching (`PROPOSED`)

- Support both light and dark palettes in the design contract even if the first release's **user-selectable theme setting** is not yet approved (`DS-Q05`).
- If theme selection is available, consider `System / Light / Dark` with immediate preview and no lost input.
- Confirm consistent system status/navigation bar contrast and keyboard/dialog appearance on Android/iOS.
- Test charts independently in both modes; alpha-filled percentile bands and grid lines often fail against dark surfaces.
- Do not use fixed “white” backgrounds in a component with dark-mode text tokens.
- A screenshot in one theme does not prove the alternate theme is usable.

---

## 18. Privacy, permission, authentication, and trust UI

The design system implements privacy signals **without pretending the UI can replace server authorization**.

1. Explain why financial profile fields are collected and whether each is optional.
2. Do not expose balances, goal names, or transaction descriptions in unauthenticated push notifications by default.
3. Use appropriate screen privacy/obscuring policies only after reviewing user needs and platform capability; **do not claim screenshots are blocked** without testing.
4. Protect navigation and data visibility on logout/account switch; purge or isolate sensitive presentation cache according to approved frontend policy.
5. Password/token information must never appear in error sheets, copy-to-clipboard diagnostics, or debug banners.
6. Export/delete screens present consequences and status honestly. A successful tap does not mean data has already been exported or purged.
7. Important state distinctions: `session expired`, `access denied`, `resource missing`, `network unavailable`, `server unknown outcome`, `deleted/confirmed`.
8. Analytics instrumentation, if approved, must avoid raw monetary amounts and personally identifying descriptors, consistent with PRD privacy constraints.

---

## 19. Flutter implementation blueprint (guidance, not existing code)

### 19.1 Candidate paths in the monorepo

```text
frontend/lib/
  app/
    theme/
      es_theme.dart               # ThemeData/ColorScheme mapping (proposed)
      es_theme_tokens.dart        # Custom financial tokens (proposed)
      es_typography.dart
    router/                       # Router choice remains TBD
  core/
    design_system/
      primitives/
      components/
      financial/
      charts/
    formatting/
      money_format.dart           # Format only; exact data semantics from contract
      date_format.dart
      percent_format.dart
    localization/
  features/
    accounts/
    transactions/
    goals/
    dashboard/
    planning/                    # R2 only
    simulations/                 # R2/R3 per FR and release
```

This is a **proposed directory organization**, not an instruction to generate empty folders/components everywhere. Match the eventual `FRONTEND-ARCHITECTURE.md` and task-specific file structure. Avoid importing backend models directly into Flutter UI.

### 19.2 Token implementation guidelines

- Base Flutter visual behavior on `ThemeData` and `ColorScheme` with Material 3 as a useful foundation; **do not** make Material defaults the unreviewed final Esokira palette.
- Map approved tokens into light/dark themes once; reference semantic roles from widgets.
- For additional `actual/planned/forecast/scenario/stale` display roles, prefer a typed theme extension or similarly centralized token object.
- Keep brand color tokens distinct from statuses (success/warning/error) and graph series colors.
- Component APIs should accept validated presentation models and callback intents, not JSON maps or business repositories directly.
- Avoid adding a third-party component, chart, font, animation, or icon library without recording why it is needed and how it affects app size, license, performance, and testing.
- Navigation library, state management, offline persistence, auth provider, and API money type are **not** decided by this file; see frontend/API/database decision registers.

### 19.3 Illustrative Dart theme wiring (not a compiled project)

```dart
// Conceptual wiring only; adapt exact roles to the reviewed Flutter version.
const esPrimaryLight = Color(0xFF0D5B50);
final colorScheme = ColorScheme.fromSeed(
  seedColor: esPrimaryLight,
  brightness: Brightness.light,
);

final theme = ThemeData(
  useMaterial3: true,
  colorScheme: colorScheme,
  // Map approved semantic brand/financial roles via component themes
  // and a typed ThemeExtension. Do not hardcode widget-by-widget colors.
);
```

This snippet does **not** implement the full hex palette above or certify accessibility. The design-implementation task must map the approved tokens explicitly, run tests, and inspect real screens. Flutter's Material 3 defaults and ColorScheme roles are documented at [Flutter Material 3](https://docs.flutter.dev/release/breaking-changes/material-3-default) and [ColorScheme API](https://api.flutter.dev/flutter/material/ColorScheme-class.html).

### 19.4 Independent preview / Storybook-like catalog

Recommended during R0/R1: a development-only component catalog showing each primitive in light/dark, narrow/expanded widths, long Indonesian strings, scaled text, disabled/error states, and synthetic money examples labeled `DEMO`. The catalog must never ship as a financial dashboard populated with fabricated personal data.

---

## 20. Testing and design-quality gates

### 20.1 Required validation layers

| Gate | What to verify | Evidence |
|---|---|---|
| `DS-T01` Tokens | No unexplained raw colors/spacing outside central theme, exception list reviewed | Static review/search or lint convention |
| `DS-T02` Contrast | Relevant text, chart/UI role combinations and interactive states at actual backgrounds | Contrast report with audited pairs/screens |
| `DS-T03` State matrix | Loading/empty/error/success/partial/stale for each eligible feature | Widget tests & review captures |
| `DS-T04` Money display | Exact input preserved; negative, zero, huge, unknown, IDR, long numbers | Formatter tests + golden screenshots |
| `DS-T05` Data provenance | Actual/planned/forecast/model source labels, dates/assumptions visible | Widget tests and manual review |
| `DS-T06` Critical navigation | Focus order, protected route, back behavior, unsaved edit dialogs | Integration/widget tests |
| `DS-T07` Accessibility | TalkBack/VoiceOver, text scaling, touch targets, non-color cues | Device/manual accessibility checklist |
| `DS-T08` Chart honesty | Same axes, series semantics, actual/forecast cutoff, alt table, incomplete data | Chart widget + data rendering assertions |
| `DS-T09` Retry safety | UI does not claim completed transfer after ambiguous response | Frontend integration test with fault injection |
| `DS-T10` Dark/size variants | Narrow, typical, expanded, light/dark, keyboard visible | Golden/test matrix with stable fixtures |
| `DS-T11` Copy and localization | No uncategorized hardcoded strings; financial caveats accurate | Localization audit + PRD review |
| `DS-T12` Performance | Chart/data-list rendering and animation fit measured budgets | Profile-mode measurement; no invented target values |

### 20.2 Golden screenshot coverage (candidate set)

- App shell: empty/onboarding and authenticated state (synthetic).
- Accounts/transactions: positive/negative/zero/missing amount, transfer type, list/filter.
- Net worth: positive, negative, and **partial data**.
- Goal: new goal, feasible plan, infeasible plan, **funded status withheld** while `Q-11` unresolved.
- Compound: assumptions, zero rate, long horizon, unreasonably large input warning (if approved policy exists).
- Forms: inline error, keyboard overlay, unsaved change confirmation.
- Forecast/scenario (R2): actual/history cutoff, stale scenario, incompatible compare, insufficient data.
- Monte Carlo (R3): queued/running/failed, results plus model metadata **only after statistical policy approved**.
- Financial GPS (R3): alternatives, drivers, trade-off and confirmation scope.

Golden tests detect visual regressions; they do not prove financial formulas. Numerical correctness is checked against `FINANCIAL-TEST-CASES.md` and server-side Go tests.

### 20.3 Definition of Ready / Done for design implementation

**Ready:** linked FR, screen state diagram, data source/provenance, API contract maturity, approved token roles, component accessibility requirements, and fixture scope. If blocked by missing domain policy, stop the affected portion instead of making up numbers.

**Done:** reviewed screens match approved tokens, all states render truthfully, no sensitive data in debug copy, accessibility and golden/widget tests performed with actual logs/results, data provenance correctly labeled, conflicts documented, and no placeholder completed features. “Looks polished” alone is not Done.

---

## 21. Traceability to product and architecture

### 21.1 Feature-specific design obligations

| Scope | Requirements | Design deliverables and dependencies |
|---|---|---|
| Authentication/onboarding | `FR-001`, `FR-002` | Consent copy, auth error screens, profile forms, protected navigation; provider TBD. |
| Accounts/transactions | `FR-003`, `FR-004` | Exact money input, posted/ambiguous mutation states, transfer semantic labels. |
| Assets/net worth | `FR-005`, `FR-006` | Ownership-safe summary, no double count, partial-as-of labels. |
| Goals and calculators | `FR-007`, `FR-008`, `FR-009` | Separate target/funded/required set-aside; assumptions and infeasible variants. |
| Data lifecycle | `FR-021` | Export/delete consent, confirmation, status, privacy-safe copy. |
| Budgets/events | `FR-010`, `FR-011` | Future planned content visibly different from posted actuals. |
| Projection/scenario | `FR-012`, `FR-013`, `FR-014` | Version/source/horizon labels, common-axis comparison, stale state. |
| Stress and insights | `FR-015`, `FR-016`, `FR-022` | Shock explanation, nonjudgmental action, privacy-respecting notifications. |
| Optimization | `FR-017` | Constraint explanation, infeasible outcome and alternatives. |
| Monte Carlo/retirement | `FR-018`, `FR-019` | Model-based uncertainty, distribution semantics, nominal/real display. |
| Financial GPS | `FR-020` | Explainable alternatives without auto-posting financial decisions. |
| Discovery only | `FR-023`, `FR-024` | No production navigation/routes pending future scope approval. |

### 21.2 Non-functional requirements

| NFR | Design response |
|---|---|
| `NFR-001` Security and authorization | Sensitive navigation, privacy notices, safe unauthorized states; server checks remain mandatory. |
| `NFR-002` Financial correctness | Exact-value formatting, no numerical invention, golden-case-compatible presentation. |
| `NFR-003` Reproducibility | Input assumptions, baseline/scenario/engine provenance, old-result freshness labels. |
| `NFR-004` Performance | Lightweight view composition, measured chart/list performance, no invented latency promises. |
| `NFR-005` UX and accessibility | Contrast, scaling, component states, local formatting, chart alternatives. |
| `NFR-006` Reliability | Ambiguous result and safe retry states; not a replacement for server idempotency. |
| `NFR-007` Privacy-first observability | No financial raw amounts in events/crash details. |
| `NFR-008` Consent and data lifecycle | Clear consent, export, deletion and retention communication. |
| `NFR-009` Quality gates | Widget, golden, accessibility, contract and scenario UI testing. |
| `NFR-010` Data boundary | Persistent visual and interaction separation: actual vs planned vs modeled. |

### 21.3 Financial invariants and calculation contracts

**Design cannot override** `DR-001`–`DR-024` nor `CS-001`–`CS-016`. All are inherited by reference, not redefined in this document. Particularly important:

- `CS-003/004`: posted accounting and net worth meaning; no visual double counting.
- `CS-005/006/007`: compound, inflation, goal feasibility; no changing rate conventions in UI.
- `CS-009/012`: deterministic projection and scenario versions; provenance and stale labels.
- `CS-013/014/015/016`: advanced optimizer, Monte Carlo, retirement, explainable insights; not production-ready without model-specific approval.

### 21.4 Engineering enablers

| EN | UI impact |
|---|---|
| `EN-001` | App shell, reusable design primitives, format/lint/test pipeline in Flutter. |
| `EN-002` | Do not claim owner-safe data based on route UI; database/API enforce ownership. |
| `EN-003` | Typed API/validation/error integration; UI does not invent response schema. |
| `EN-004` | Formatted output uses one verified financial calculation engine. |
| `EN-005` | Advanced job lifecycle UI only after durable worker/status protocol exists. |
| `EN-006` | Error/telemetry privacy and app environment separation. |

---

## 22. Decision register: unresolved design decisions

All entries are `PROPOSED` or `TBD`, not a substitute for Product Owner approval. Do not silently pick an option as the final production policy.

| ID | Decision | Current proposal | Gate / owner question |
|---|---|---|---|
| `DS-Q01` | Final visual identity, Esokira branding | Navy / deep teal / off-white, path motif | Approve colors and brand direction after mock review. |
| `DS-Q02` | Exact font family and license | Platform system fonts initially | Does the product require a custom brand face? |
| `DS-Q03` | Design tokens, typographic scale, radii | Tables in this file | Approve token roles before implementation. |
| `DS-Q04` | Navigation destinations and exact route hierarchy | Up to 5 candidate areas | Approve after `USER-FLOWS` / `SCREEN-SPECS` and task testing. |
| `DS-Q05` | Theme behavior | Both palettes drafted; system/light/dark option candidate | Ship light only, system-aware, or full selector in R1? |
| `DS-Q06` | Supported form factors and screenshot baselines | Narrow/standard/expanded buckets | Which devices, OS versions, accessibility scales? |
| `DS-Q07` | Chart library / performance and accessibility | Pick after prototype and benchmarks | Which library meets interaction/semantics needs? |
| `DS-Q08` | Component naming and directory layout | `Es*` candidate convention | Align with Flutter coding standard and implementation. |
| `DS-Q09` | Numeric compact notation and copy policies | Full amount on detail/a11y, short axis labels | Exact approved formatter and threshold rules? |
| `DS-Q10` | Goal funded progress UI | Withhold true funded % until `Q-11` approved | How are funded amounts actually recorded? |
| `DS-Q11` | Financial assumptions defaults/UI controls | No hidden defaults | Which inputs/ranges/units have approved domain policies? |
| `DS-Q12` | Monte Carlo/optimizer risk communication | Model-provenance cards and distribution summaries | Await validated `CS-013`/`CS-014` models. |
| `DS-Q13` | Offline/cache privacy presentation | Show freshness and error honestly | Need approved offline storage and session policies. |
| `DS-Q14` | Haptic/motion policy | Minimal functional motion | Confirm reduced-motion and device testing. |
| `DS-Q15` | Illustration and brand asset package | No production assets created | Who approves illustrations, icons, and legal clearance? |
| `DS-Q16` | Localization rollout | Indonesian-first, IDR-first | English/multicurrency availability and copy review? |

**Cross-document decisions that remain blockers:** PRD `Q-02` auth provider, `Q-03` ledger, `Q-04` precision/timing, `Q-05` model assumptions, `Q-06` scenario baseline, `Q-07/08` optimizer/statistical model, `Q-09` notifications, and `Q-11` goal funding; backend/frontend/database/API-specific Q-ids also remain open. Designing an attractive card cannot resolve those blockers.

---

## 23. Rollout plan for this design system

| Release | Minimum deliverable | Visual validation |
|---|---|---|
| `R0` | Theme tokens, typography, spacing, shared button/field/state, component catalog prototype | Light/dark contrast sample, narrow/large text, navigation + error shell. |
| `R1` | Realistic onboarding, accounts, transactions, actual dashboard, goals, calculators, export/delete | Every screen has loading/error/empty/partial variants; no synthetic financial success in production. |
| `R2` | Life timeline, source-aware projection, scenario comparison, cash flow, insights | Actual vs forecast distinction, common axes, stale/version UX, accessible charts. |
| `R3` | Optimizer, Monte Carlo, retirement, Financial GPS | Model metadata, percentiles, infeasible states, async jobs, risk copy reviewed. |
| `R4` | Discovery only | No production routes or committed visual specs for shared goals/portfolio. |

**Suggested iteration sequence:** (1) approve direction/tokens → (2) build one R0 reference screen with all states → (3) review accessibility and copy → (4) expand component catalog → (5) build R1 screens following PRD dependencies. Do **not** spend weeks polishing R3 visuals before financial model constraints are decided.

---

## 24. Internal review checklist and handoff

### 24.1 Document consistency check (not app test)

- [x] Treats Flutter/Go/PostgreSQL/monorepo as decisions and the visual palette as a proposal.
- [x] Inherits Product Vision principles and PRD release levels (`R0`–`R4`).
- [x] Covers every `FR-001`–`FR-024` by release; R4 remains discovery only.
- [x] Addresses `NFR-001`–`NFR-010` and `EN-001`–`EN-006`.
- [x] References financial rules and calculation contracts without redefining monetary or stochastic formulas.
- [x] Distinguishes actual, planned, projection, model, stale, pending, and incomplete information.
- [x] Explicitly refuses to fabricate financial output or imply pending operations were confirmed.
- [x] Color contrast calculated for the **listed example pairs**, not represented as a complete device audit.
- [x] Contains component state, visualization, accessibility, privacy, and design QA requirements.
- [x] Keeps unresolved design/financial/API decisions visible and blocks affected production behavior.

### 24.2 Product Owner design review questions

1. Does the navy/teal/off-white visual direction feel right for Esokira, or should the palette change?
2. Should the R1 navigation prioritize `Keuangan` and `Tujuan` before exposing the `Rencana` area in R2?
3. Do you prefer native system typography for the first release or a licensed custom font?
4. Will light/dark theme selection ship in R1, or should we initially follow system mode?
5. What are the first Android/iOS test devices and screen widths?
6. Which financial simulation disclosures should be visible immediately versus behind a clear `Lihat asumsi` action?

### 24.3 Next documents

Recommended project sequence after this file:

1. `docs/design/USER-FLOWS.md` — real task paths, failure paths, navigation approvals.
2. `docs/design/SCREEN-SPECS.md` — exact screen inventory, component states, data contracts, navigation and acceptance criteria.
3. Agent instructions and implementation planning when product/design/technical blockers are reviewed; do not manufacture completeness from documents alone.

---

## References and provenance

**Internal Esokira:** the 11 earlier product, financial, and architecture Markdown files listed in §0.2, as available for this drafting round. Their status is internal review / Product Owner approval pending.  
**External normative/background references:**

- W3C, **Web Content Accessibility Guidelines 2.2**: <https://www.w3.org/TR/WCAG22/>.
- Flutter, **Accessibility**: <https://docs.flutter.dev/ui/accessibility>.
- Flutter, **UI design and accessibility styling**: <https://docs.flutter.dev/ui/accessibility/ui-design-and-styling>.
- Flutter, **Material 3 is the default**: <https://docs.flutter.dev/release/breaking-changes/material-3-default>.
- Flutter, **ColorScheme API**: <https://api.flutter.dev/flutter/material/ColorScheme-class.html>.

**Document limits:** no UI assets, source widgets, figma files, screenshot golden files, platform/accessibility test evidence, font files, or brand clearance checks are contained in this package. All examples of balances, dates, forecast amounts, and model success frequencies are **synthetic illustrative content**, never real user data or an approved forecast.

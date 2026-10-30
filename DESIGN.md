---
version: alpha
name: Oracle Buffet
description: Calm, evidence-first desktop web interface for one private investor reviewing daily US stock scores, strategy versions, and trade results.
colors:
  primary: "#1F3A5F"
  on-primary: "#FFFFFF"
  accent: "#8A6A12"
  accent-light: "#E8C873"
  background: "#FAF8F3"
  surface: "#FFFFFF"
  on-surface: "#1B2430"
  muted: "#56606E"
  border: "#7C8594"
  positive: "#1E6B3A"
  negative: "#A3261F"
  notice: "#FBF1D9"
typography:
  display:
    fontFamily: Georgia
    fontSize: 32px
    fontWeight: 600
    lineHeight: 1.2
  headline:
    fontFamily: Georgia
    fontSize: 24px
    fontWeight: 600
    lineHeight: 1.3
  body:
    fontFamily: system-ui
    fontSize: 16px
    fontWeight: 400
    lineHeight: 1.5
  label:
    fontFamily: system-ui
    fontSize: 14px
    fontWeight: 600
    lineHeight: 1.4
  data:
    fontFamily: ui-monospace
    fontSize: 14px
    fontWeight: 400
    lineHeight: 1.4
rounded:
  sm: 4px
  md: 8px
  full: 9999px
spacing:
  xs: 4px
  sm: 8px
  md: 16px
  lg: 24px
  xl: 40px
components:
  button-primary:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 12px
  button-secondary:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.primary}"
    typography: "{typography.label}"
    rounded: "{rounded.sm}"
    padding: 12px
  card:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    rounded: "{rounded.md}"
    padding: 24px
  page:
    backgroundColor: "{colors.background}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body}"
  caption:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.muted}"
    typography: "{typography.label}"
  input:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body}"
    rounded: "{rounded.sm}"
    padding: 8px
  badge-buy:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.positive}"
    typography: "{typography.label}"
    rounded: "{rounded.full}"
  badge-sell:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.negative}"
    typography: "{typography.label}"
    rounded: "{rounded.full}"
  badge-hold:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.accent}"
    typography: "{typography.label}"
    rounded: "{rounded.full}"
  disclaimer-banner:
    backgroundColor: "{colors.notice}"
    textColor: "{colors.on-surface}"
    typography: "{typography.body}"
    padding: 16px
  nav-bar:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.on-primary}"
    typography: "{typography.label}"
    height: 56px
  nav-mark:
    backgroundColor: "{colors.primary}"
    textColor: "{colors.accent-light}"
    size: 32px
  control-border:
    backgroundColor: "{colors.surface}"
    textColor: "{colors.border}"
    height: 1px
---

# Oracle Buffet Design System

Owns the visual and frontend design language: brand and logo, color, typography, layout, components, accessibility, and security-relevant user interactions. Behavior: `REQUIREMENTS.md`. Security controls: `SECURITY.md`. Token values are in the front matter; the sections below name tokens and do not repeat their values.

## Overview

A private, single-user desktop web tool (NFR-1). The tone is a quiet ledger: patient, factual, long-term, like value investing itself. Every screen favors evidence (scores, reasons, backtest comparisons) over excitement. No element suggests urgency or trade execution (FR-3.9). The logo is `logo.svg`.

## Colors

Restrained palette: deep navy for structure, one muted gold for emphasis, warm paper background.

- **primary:** navigation, primary actions, focus ring, links.
- **accent:** "hold" state, version highlights, logo. Use as text only at 14px+ weight 600.
- **accent-light:** decorative only, on `primary` backgrounds (logo, nav marks).
- **positive / negative:** buy/sell, success/failure versus S&P 500. Never the sole signal. The required pairing (a word or icon, or text) is open: DQ-6.
- **muted:** secondary text, captions, axis labels.
- **border:** input borders, dividers that identify a control.
- **notice:** disclaimer banner background.

Verified WCAG 2.2 AA contrast pairs (all used pairs):

| Foreground | Background | Ratio | Use |
|---|---|---|---|
| on-surface | background / surface / notice | 14.75 / 15.65 / 13.93 | Body text |
| on-primary | primary | 11.48 | Buttons, nav |
| primary | background / surface | 10.82 / 11.48 | Links, focus ring, secondary button |
| muted | background / surface | 6.01 / 6.38 | Captions |
| positive | background / surface | 6.15 / 6.52 | Buy, success |
| negative | background / surface | 6.93 / 7.36 | Sell, failure |
| accent | background / surface / notice | 4.77 / 5.06 / 4.50 | Hold, highlights |
| border | background / surface | 3.51 / 3.72 | Control boundaries (3:1) |
| accent-light | primary | 7.08 | Logo, decorative marks |

## Typography

- **display / headline (serif):** page titles and section heads only.
- **body (sans):** all prose, form text, disclaimer.
- **label (sans, 600):** buttons, badges, table headers.
- **data (mono, tabular):** prices (USD, FR-1.2), scores 0–100, returns, dates, quantities. Align numbers right.
- Font stacks (NFR-8): Georgia or the system serif; the system sans; the system monospace. No font files are bundled.

## Layout

- Desktop only (NFR-1). Content max width 1280px, centered. At the NFR-7 minimum window width there is no horizontal page scroll (wide tables may scroll inside their card).
- 8px base grid using `spacing` tokens.
- Light theme only (NFR-7).
- Persistent top `nav-bar`, with the `disclaimer-banner` directly below it.
- Primary views implied by REQUIREMENTS.md: Login, Daily Scores, Stock Detail, Buffett Trades, Strategy and Model Versions, Proposals, My Trades (entry + CSV import), Trade Results, Change Password (current password plus new password, FR-6.4). A "Log out" action sits in the `nav-bar` on every authenticated view (FR-6.5). There is no sign-up view (FR-6.3).

## Elevation & Depth

Flat. Depth comes from `surface` cards on the `background` page plus a 1px `border` line. No shadows except one subtle shadow on modal dialogs (`0 4px 16px` at 16% black) so they stand out from the page.

## Shapes

`rounded.sm` for controls, `rounded.md` for cards and dialogs, `rounded.full` for badges. No other radii.

## Components

- **Score row:** symbol (data), score 0–100 (data, plus a horizontal bar in `primary`), recommendation badge, top features as reason text (FR-3.7).
- **No-result row:** score replaced by the text "No result" in `muted` plus the cause (FR-3.8). No badge.
- **Recommendation badge:** `badge-buy` / `badge-hold` / `badge-sell`; always shows the word; 1px border in the same text color.
- **Version card:** strategy and trade model shown as two linked but separate cards (FR-3.1), each with version number, date, and status (current / previous). Action: "Go back to this version" as `button-secondary` (FR-5.6).
- **Proposal comparison:** two columns, current versus proposed, backtest total return versus S&P 500 for each (FR-5.3); actions "Approve" (`button-primary`) and "Reject" (`button-secondary`) (FR-5.4).
- **Hidden-model state:** when the backtest does not beat the S&P 500 (FR-3.5), show an explanatory `card` instead of results.
- **Trade form:** one `input` with a visible label for each FR-4.1 field; CSV import as a separate action (FR-4.2).
- **Success marker:** text "Beat S&P 500" / "Did not beat S&P 500" in `positive` / `negative`.
- **Line chart:** price history and backtest curves are thin lines only. Trade model or stock series: solid 2px `primary`. S&P 500 baseline: dashed 2px `muted`. Each series has a text legend. Axis labels use `data` in `muted`.
- **Focus indicator:** 2px solid `primary` outline with 2px offset on all interactive elements.
- **Disclaimer banner:** fixed statement that output is not investment advice (FR-7.1). It shows on every authenticated view and cannot be dismissed, so the warning is always on screen.
- **Rendering and client state:** follow SC-025 and SC-026 in `SECURITY.md`.

## Do's and Don'ts

- Do show every number with its unit and comparison baseline (S&P 500).
- Don't use "Trade now", "Execute", or broker-like wording.
- Don't use animation to signal market movement.
- Don't introduce colors outside the token set.
- Don't use imagery or wording that NFR-9 prohibits.

## Open Questions

- **DQ-6:** Color pairing. Colors says to pair `positive` / `negative` with "a word or icon". Do's and Don'ts (before 2026-10-30) said "pair every color signal with text". Alternatives: (a) a word or an icon; (b) text in all cases. All current components use words. TO BE DECIDED.

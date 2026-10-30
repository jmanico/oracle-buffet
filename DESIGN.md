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

## Overview

A private, single-user desktop web tool (NFR-1). The tone is a quiet ledger: patient, factual, long-term, like value investing itself. Every screen favors evidence (scores, reasons, backtest comparisons) over excitement. No element suggests urgency or trade execution (FR-3.9). The "not investment advice" statement (FR-7.1) is always visible.

## Colors

Restrained palette: deep navy for structure, one muted gold for emphasis, warm paper background.

- **primary #1F3A5F:** navigation, primary actions, focus ring, links.
- **accent #8A6A12:** "hold" state, version highlights, logo. Use as text only at 14px+ weight 600.
- **accent-light #E8C873:** decorative only, on `primary` backgrounds (logo, nav marks).
- **positive #1E6B3A / negative #A3261F:** buy/sell, success/failure versus S&P 500. Never the sole signal; always pair with a word or icon.
- **muted #56606E:** secondary text, captions, axis labels.
- **border #7C8594:** input borders, dividers that identify a control.
- **notice #FBF1D9:** disclaimer banner background.

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
- Fonts are the operating system's built-in fonts only (Georgia or the system serif; the system sans; the system monospace). No font files are bundled or loaded from external services.

## Layout

- Desktop only. Content max width 1280px, centered. Minimum supported viewport width 1024px with no horizontal page scroll (wide tables may scroll inside their card).
- 8px base grid using `spacing` tokens.
- Light theme only. There is no dark theme.
- Persistent top `nav-bar`; persistent `disclaimer-banner` directly below it.
- Primary views implied by REQUIREMENTS.md: Login, Daily Scores, Stock Detail, Buffett Trades, Strategy and Model Versions, Proposals, My Trades (entry + CSV import), Trade Results.

## Elevation & Depth

Flat. Depth comes from `surface` cards on the `background` page plus a 1px `border` line. No shadows except an optional subtle shadow on modal dialogs (ASSUMPTION).

## Shapes

`rounded.sm` for controls, `rounded.md` for cards and dialogs, `rounded.full` for badges. No other radii.

## Components

- **Score row:** symbol (data), score 0–100 (data, plus a horizontal bar in `primary`), recommendation badge, top features as reason text (FR-3.7).
- **No-result row:** score replaced by the text "No result" in `muted` plus the cause (FR-3.8). No badge.
- **Recommendation badge:** `badge-buy` / `badge-hold` / `badge-sell`; always shows the word; 1px border in the same text color.
- **Version card:** strategy and trade model shown as two linked but separate cards (FR-3.1), each with version number, date, and status (current / previous). Action: "Go back to this version" as `button-secondary` (FR-5.6).
- **Proposal comparison:** two columns, current versus proposed, backtest total return versus S&P 500 for each (FR-5.3); actions "Approve" (`button-primary`) and "Reject" (`button-secondary`) (FR-5.4).
- **Hidden-model state:** when the backtest does not beat the S&P 500 (FR-3.5), show an explanatory `card` instead of results.
- **Trade form:** symbol, buy/sell, date, quantity, price (FR-4.1); `input` components with visible labels; CSV import as a separate action (FR-4.2).
- **Success marker:** text "Beat S&P 500" / "Did not beat S&P 500" in `positive` / `negative`.
- **Line chart:** price history and backtest curves are thin lines only. Trade model or stock series: solid 2px `primary`. S&P 500 baseline: dashed 2px `muted`. Each series has a text legend. Axis labels use `data` in `muted`.
- **Focus indicator:** 2px solid `primary` outline with 2px offset on all interactive elements.
- **Disclaimer banner:** fixed statement that output is not investment advice; not dismissible (ASSUMPTION).

## Do's and Don'ts

- Do show every number with its unit and comparison baseline (S&P 500).
- Do pair every color signal with text.
- Do keep the disclaimer visible on every authenticated view.
- Don't use "Trade now", "Execute", or broker-like wording.
- Don't use animation to signal market movement.
- Don't introduce colors outside the token set.
- Don't use photos of Warren Buffett, Berkshire Hathaway logos, or wording that suggests endorsement. The name "Oracle Buffet" is for private use only.


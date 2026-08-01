# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project overview

A single-page Korean-language leadership style quiz ("리더십 유형 진단" / "Leadership Type Diagnosis"). The user answers 16 Likert-scale (1–5) questions and receives one of four leadership type results (지시형/Directive, 위임형/Delegating, 참여형/Participative, 온정형/Affiliative) along with a per-type score breakdown and an SVG diamond/radar chart.

## Structure

The entire app is `index.html` — HTML, CSS (`<style>`), and JS (`<script>`) all inline in one file. There is no build system, package manager, bundler, framework, or external dependency (no CDN scripts/fonts). This is intentional; keep it a single self-contained file unless asked to change that.

## Development workflow

- No install/build step. Open `index.html` directly in a browser, or serve it with any static file server (e.g. `python3 -m http.server`) if `file://` access is restricted.
- No test suite, linter, or formatter is configured. Verify changes by manually clicking through the flow in a browser (intro → questions → result).

## Architecture

The page is three screens toggled by JS via `display`/class changes on fixed IDs — there is no router or framework:

- `#intro` — start screen
- `#diag` — one question at a time, shown/hidden via `startDiag()`
- `#result` — final result, shown via `showResult()` (adds `.show` class)

State is two globals: `cur` (current question index, 0–15) and `answers` (array of 16 scores, `0` = unanswered). Key functions in the single `<script>` block:

- `renderQ()` — paints the current question and its 5 score buttons from `answers[cur]`
- `select(v)` / `nextQ()` / `prevQ()` — record an answer and move between questions
- `showResult()` — sums per-type scores and picks the highest as the result
- `drawDmd(totals)` — renders the four-axis diamond chart by building raw SVG nodes (no charting library)
- `reset()` — returns to the intro screen

### Question → type mapping

`qs` is a 0-indexed array of the 16 question strings. `mapping` maps each type (`A`/`B`/`C`/`D`) to **1-indexed** question numbers (4 per type), e.g. `A:[2,7,9,15]` means questions at `qs[1]`, `qs[6]`, `qs[8]`, `qs[14]`. `showResult()` sums `answers[q-1]` for each type's question list to get that type's score; the type with the highest total wins.

When adding, removing, or reordering questions, `qs` and `mapping` **must be updated together** — mapping numbers are not derived automatically and are easy to desync from the array.

### Type metadata

`types` holds each type's display name, tagline, colors (`color`/`light`), and description shown on the result screen. Score-row/chart colors are duplicated separately in `showResult()` (`clrs`, `nms`) and must be kept in sync with `types` if colors or Korean labels change.

### Hardcoded question count

The total of 16 is hardcoded in several places (progress bar percentage math in `renderQ()`, `1 / 16` label, `answers` array length, the `cur===15` end-of-quiz checks) rather than derived from `qs.length`. Changing the number of questions requires updating all of these, not just `qs`.

## Conventions

- Code is written in a dense, semicolon-per-statement, minimal-whitespace style — match this when editing rather than reformatting to a more verbose style.
- UI copy is Korean; keep new copy consistent in tone/register with existing strings.
- Styling uses hardcoded hex colors per type (no CSS variables/theme system): A `#1A9E75`, B `#D85A30`, C `#555`, D `#2BAB7A` (brand green `#2BAB7A` doubles as D's color and the general accent color for intro/progress/buttons).

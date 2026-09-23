# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## What this is

"Missão Matemática" (Caça ao Conjunto) — a set of standalone, static HTML pages teaching Brazilian 6º ano math (set theory and fractions) through three mini-games on a space theme. Pure vanilla HTML/CSS/JS, no framework, no build step, no package.json, no server. See [FUNCIONALIDADES.md](FUNCIONALIDADES.md) for a full screen-by-screen and function-by-function reference — read it before making non-trivial changes instead of re-deriving behavior from scratch.

## Commands

There is no build, lint, or test tooling in this repo. To work on it:

- **Run/preview a page**: open the `.html` file directly in a browser (double-click, or `start "" index.html` on Windows). No dev server is needed or used.
- **Quick syntax check** after editing a file's inline `<script>` block (useful before committing, since there's no linter):
  ```bash
  node -e "
  const fs=require('fs');
  const html=fs.readFileSync('index.html','utf8');
  const m=html.match(/<script>([\s\S]*)<\/script>/);
  try{ new Function(m[1]); console.log('OK'); }catch(e){ console.log('ERROR', e.message); }
  "
  ```
  Run the same for `caca-ao-conjunto.html`, `diagrama-de-venn.html`, and `pocoes-magicas.html` after touching any of them.

## Architecture

### Four independent pages, tied together only by localStorage

- `index.html` — login/avatar screen + the map of planets (mission select), player rank and XP.
- `caca-ao-conjunto.html` — Game 1: falling-meteor classification (sort items into two sets before they land).
- `diagrama-de-venn.html` — Game 2: drag-and-drop Venn diagram placement (2 or 3 sets).
- `pocoes-magicas.html` — Game 3: fraction operations across 5 *independent* levels selected from an in-game level map (level 1 starts unlocked, finishing one unlocks the next, per-level 3-lives/5-question sessions, progress in `mq_pocoes_niveis`). Levels 1/2/4 answer via 3-option multiple choice (wrong options model common mistakes); levels 3/5 answer by tapping a 1–12 numeric keypad for numerator/denominator. Every fraction is always shown as a divided flask, never as inline "n/d" text. Completing a level for the first time unlocks a named collectible potion in the in-game Grimório (`GRIMOIRE` array).

Each file is fully self-contained: its own `<style>` and `<script>` inline in the `<head>`/before `</body>`. There is no shared CSS/JS file and no bundler — if a fix or style applies to more than one page, it has to be duplicated by hand into each file.

Every page uses a `store` wrapper (`try/catch`-guarded `localStorage.getItem/setItem`) and communicates purely through these keys:

| Key | Shape | Written by |
|---|---|---|
| `mq_player` | `{ name, avatar }` | index.html (on login) |
| `mq_sound` | `boolean` | any page (sound toggle) |
| `mq_progress` | `{ [nameLower]: { [gameId]: { best, stars } } }` | each game's `endGame()`, on legitimate match completion only (win or lives=0) |
| `mq_gas` | `{ [nameLower]: { gas, stars } }` | all three games (`addGas()`, on every correct answer — survives leaving mid-match); spent only via the trade UI in `index.html` |
| `mq_pocoes_niveis` | `{ [nameLower]: { unlocked[5], stars[5], best[5] } }` | `pocoes-magicas.html` only, on `finishLevel(true)`; its per-level `stars` average feeds `mq_progress['pocoes'].stars` |
| `mq_caca_grimorio` / `mq_venn_grimorio` | `{ [nameLower]: { [itemId]: true } }` | that game's `unlockItem(id)`, on specific milestones (round completion, perfect win, no-hint win, combo streak) — independent of `mq_progress`, purely a collectible log |

`gameId` is `'caca'`, `'venn'`, or `'pocoes'` (the `GAME_ID` constant at the top of each game's script).

### Per-game internal structure (same pattern in all three game files)

Each game script is one IIFE with, in order: a `$(id)` shortcut, the `store` wrapper, a `beep()`-based Web Audio sound layer (no audio files — all SFX are synthesized oscillator tones, gathered in an `sfx` object), a content array that fully defines that game's questions (`ROUNDS` in the first two games, `LEVELS` + per-level `gen*()` generator functions in the fractions game — this is where to add or tweak questions), then game state (`score`, `lives`, `combo`, etc.), the render/`paintX()` functions, and the answer-checking logic.

Screen flow differs slightly by game. Caça/Venn play as one continuous run: Início (rules) → Rodada intro → live gameplay → Pausa/Sair → Fim, with `endGame(won)` as the single place that computes the 0–3 star rating and writes `mq_progress` (leaving via the exit button skips it, so quitting mid-match doesn't save score or stars — only the gas bank survives that). Poções instead has an in-game level map between Início and gameplay (`showLevelMap`); each level is its own 3-lives/5-question session started by `startLevel()` and scored by `finishLevel(won)`, which writes per-level results to `mq_pocoes_niveis` and rolls their average into `mq_progress` via `syncOverallProgress()`.

### The gas economy (cross-file feature)

Points scored in any of the three games also feed a persistent, per-player gas bank (`mq_gas`) via `addGas()`, saved immediately on every correct answer so it can't be lost by quitting. The HUD's gas-cloud icon in each game is a passive visual only (grows toward 5000, no button). The actual trade UI — a `+` button next to `⭐ totalStars`, opening a "Trocar gás por estrela" modal — lives only in `index.html`; 5000 gas manually exchanges for 1 star there, and exchanged stars are added into the map's total star count (`showMap()`) alongside each game's own 0–3 rating.

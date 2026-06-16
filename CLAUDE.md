# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Pottery Journal is a single-file, dependency-free web app for tracking ceramic pieces through the stages of creation as a kanban board. The entire application — HTML, CSS, and JavaScript — lives in `index.html`. There is no build step, package manager, test suite, or backend.

## Running

Open `index.html` directly in a browser, or serve the directory (e.g. `python3 -m http.server`) and visit it. There is nothing to build or install.

## Architecture

State is a single `pieces` array of plain objects (`{ id, name, description, date, column, image }`), persisted to `localStorage` under the key `pottery-pieces`. `saveToDB()` writes the whole array on every mutation; `renderBoard()` re-renders the entire board from scratch via `innerHTML`. There is no diffing or component framework — every change is full-array-save followed by full-DOM-rebuild.

The five kanban columns are a fixed pipeline defined by the `columns` array (`thrown → trimmed → bisque → glazed → fired`). Column membership is stored per-piece in `piece.column`, and the column DOM (`id="cards-<column>"`, `id="count-<column>"`) is hardcoded in the markup to match these ids. Adding or renaming a stage requires editing both the `columns`/`columnOrder` arrays in the script and the corresponding `.column` blocks in the HTML.

Pieces advance through the pipeline two ways: the `→ Next` button (`moveToNext()`, which walks `columnOrder`) and HTML5 drag-and-drop (`dragStart`/`allowDrop`/`drop`, wired up per-column at load). Images are read from file inputs as base64 data URLs via `FileReader` and stored inline in the piece object — so images count against the `localStorage` size limit.

A single shared modal handles both add and edit. `editingId` (null = add mode) and `currentColumn` are module-level globals that `savePiece()` branches on to decide whether to mutate an existing piece or push a new one.

Card markup is built with template-literal string interpolation of piece fields directly into `innerHTML` (`renderBoard()`), so user-entered names/descriptions are not escaped.

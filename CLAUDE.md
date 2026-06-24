# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

Single-file meeting task management system (会議タスク管理) for a Japanese waste disposal company. The entire application is `task_manager.html` — one HTML file with inline CSS and JavaScript. There is no build tool, no npm, no framework.

## How to Run

Open `task_manager.html` directly in a browser. No build step required. Requires internet access for:
- Supabase JS client (CDN)
- Tabler Icons (CDN)
- Supabase database (remote)

## Architecture

All logic lives in `task_manager.html` as a single monolithic file. The structure within it:

1. **HTML structure** — tab-based navigation (Tasks, AI Minutes, Minutes, Members, Add Task) with modal overlays
2. **CSS** — custom design system with green theme (`--main: #7DBF3A`), responsive breakpoints via `isWide()`
3. **JavaScript** — all functions defined globally in one `<script>` block

### Backend: Supabase

Two tables in Supabase PostgreSQL:
- `task_list` — columns: `id, title, person, due, status, from_meeting, minutes_id, progress`
- `minutes_list` — columns: `id, title, date, members, body, task_ids`

Members are stored in-memory only (local array, not persisted to DB).

### Key Function Groups

**Data layer** — `loadAll()` fetches both tables on startup; `dbInsertTask()`, `dbUpdateTask()`, `dbDeleteTask()`, `dbInsertMinutes()`, `dbDeleteMinutes()` for writes; `setSyncBadge()` shows connection status.

**Rendering** — `renderAll()` orchestrates everything; `renderSummary()`, `renderTasks()`, `renderMinutes()`, `renderMembers()` render each tab section.

**Task CRUD** — `openTask()` opens the detail modal, `saveTaskEdit()`, `changeStatus()`, `deleteTask()`, `addManualTask()`.

**AI Minutes workflow (3-step)** — User pastes a meeting transcript → Step 1 generates an LLM prompt (works with ChatGPT, Claude, etc.) → Step 2 user pastes AI output and reviews minutes → Step 3 tasks are extracted and confirmed for bulk registration. Key functions: `goStep2()`, `goStep3()`, `confirmAndRegister()`, `saveMinutesOnly()`, `extractTasksFromText()`.

**Utilities** — `today()`, `daysLeft()`, `urgencyClass()`, `daysLabel()` for deadline logic; `escHtml()`, `fmtDate()` for output safety/formatting; `isWide()` for responsive layout.

### UI/UX Conventions

- Urgency badges: red for ≤2 days remaining, amber for ≤5 days
- Status values: `未着手` / `進行中` / `完了`
- Task extraction from Japanese text handles relative dates ("来週", "今月末", etc.)
- All user-facing text and data is in Japanese

## Editing Guidelines

Since there is no linter or formatter, maintain the existing inline style consistently. When adding features:
- New DB operations follow the `db*()` naming pattern and must update `renderAll()` as needed
- New UI tabs/modals should follow the existing HTML pattern (tab button + section div + modal div)
- `extractTasksFromText()` uses regex for Japanese date patterns — test edge cases manually in the browser

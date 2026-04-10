# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

This repository follows the LLM Wiki pattern described in `llm-wiki.md`.
Read `llm-wiki.md` for the high-level philosophy, then follow the concrete repository-specific rules below.

## Core rules
- Treat `raw/` as immutable source material.
- Treat `wiki/` as the maintained knowledge layer.
- Never edit files in `raw/`.
- When ingesting a new source, update existing wiki pages before creating new ones.
- Always update `index.md` and append an entry to `log.md`.
- Prefer explicit cross-links between related pages.
- Preserve source citations for claims.

## Repository Purpose

This is a **development documentation archive** — a collection of raw devlog entries from three related projects, intended to be processed into structured knowledge via LLM Wiki workflows.

There is no source code to build, lint, or test. The repository contains only markdown files.

## Structure

- `raw/` — immutable source devlogs, organized by project:
  - `raw/trilobase-devlog/` (~103 files) — trilobase taxonomy database project
  - `raw/scoda-engine-devlog/` (~97 files) — scoda-engine project
  - `raw/fsis2026-devlog/` (~89 files) — FSIS 2026 Django project
- `graphify-out/` — output from the `/graphify` skill (knowledge graph artifacts)
- `llm-wiki.md` — reference document describing the "LLM Wiki" pattern for building persistent knowledge bases from raw sources

## Devlog Naming Convention

Files follow the pattern `YYYYMMDD_NNN_description.md` where:
- Numeric prefix (e.g., `001`, `078`) = implementation/session logs
- `P` prefix (e.g., `P01`, `P62`) = planning documents
- `R` prefix (e.g., `R01`, `R03`) = review/retrospective documents

## Key Workflows

- **Graphify**: Use `/graphify` to convert raw devlogs into knowledge graphs with clustered communities, HTML visualization, and JSON output
- **LLM Wiki**: The `llm-wiki.md` file describes a pattern for incrementally building a structured wiki from raw sources — ingest, query, and lint operations

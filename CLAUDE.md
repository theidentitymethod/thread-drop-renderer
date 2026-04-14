@AGENTS.md

# thread-drop-renderer

## Project Overview
A Next.js companion app that renders Twitter-thread-style carousels as PNG slides.
This is the rendering engine for the `thread-drop` skill hosted at
github.com/theidentitymethod/identity-method-skills. Both projects share a locked
`config.json` schema — do not modify the schema without coordinating across repos.

## Tech Stack
- **Framework:** Next.js 16 (App Router, Turbopack)
- **Language:** TypeScript (strict)
- **Styling:** Tailwind CSS
- **Rendering:** Satori (JSX-to-SVG) + @resvg/resvg-js (SVG-to-PNG)
- **Bundling:** jszip + file-saver (client-side ZIP download)
- **Validation:** Zod (runtime schema validation against config.json)

## Scope Rules
- This repo is renderer-only: it takes thread config JSON and outputs PNG slides.
- No auth, no database, no user accounts.
- Do not add features outside the render pipeline without explicit approval.
- Keep API surface minimal — one POST endpoint, one preview route.

## Brand Guidelines (Neo-Brutalist Dark)
- **Background:** `#0A0A0A`
- **Accent:** `#FFCC33`
- **Display font:** Barlow Condensed (bold/semibold for headings)
- **Body font:** System sans-serif stack
- **Style:** Dark neo-brutalist — hard shadows, sharp edges, high contrast,
  no rounded corners, no gradients.

## Conventions
- Follow Next.js 16 idioms; read `node_modules/next/dist/docs/` before writing code.
- Prefer server components; use `"use client"` only when required.
- All render logic lives in `app/api/render/` and shared utils in `lib/`.
- Validate all incoming JSON with Zod before rendering.

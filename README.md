# ux-tools

Small, focused browser tools for UX work. Each tool is a **single HTML file** — no build step, no dependencies, no backend. Open it and it works, offline; your data never leaves the device.

**Live →** <https://marekbrze.github.io/ux-tools/apps/customer-journey/>

## The idea

Most "quick tools" grow into small apps: a bundler, a package manager, a server, an account. This collection goes the other way. Every tool is one self-contained `index.html`:

- **Vanilla JS only** — no framework, no runtime library, only what the browser ships.
- **No build step, no `node_modules`** — clone and open the file.
- **Offline by default** — no network requests, system fonts, inline SVG icons.
- **Local-only data** — `localStorage` for small state, `IndexedDB` for larger blobs. Nothing is uploaded; no API keys, no account.

The constraint is the point: each tool stays small enough to read in one sitting, and trustworthy enough to point at real client data.

## Tools

| Tool | What it does |
|------|--------------|
| [customer-journey](./apps/customer-journey/) | Customer-journey map editor — a stages × 7-slice grid (NN/g framework), with categories/personas CRUD, drag & drop, and PNG export. |

`image-slicer` (a sibling repo) is the tool this collection's shared design system grew out of.

## Shared design system

All tools share one system — light by default with a dark theme — built entirely on CSS custom properties (never hardcoded values). Same tokens, spacing and accessibility conventions across the set, so it reads as one product. The full spec lives in [`CLAUDE.md`](./CLAUDE.md) (in Polish).

## Run

Open the live demo above, or open any tool's `index.html` directly in a browser. No install.

## Status

A personal collection, growing. New tools land under [`apps/`](./apps/).

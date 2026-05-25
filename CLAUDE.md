# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

E-commerce data utilities project providing query functions over a SQLite database, written in TypeScript with ES2022 modules.

## Commands

```bash
npm run setup      # Install dependencies and run init script
npm run sdk        # Run sdk.ts via tsx (Claude Agent SDK example)
npx tsx src/main.ts  # Run the main entry point
```

There is no test runner configured (`npm test` exits with an error).

TypeScript type checking is run automatically by a post-edit hook (see Hooks below), but you can also run it manually:

```bash
npx tsc --noEmit
```

## Architecture

`src/main.ts` opens an in-process SQLite database (`ecommerce.db`) and calls `createSchema` to set up all tables. Query modules in `src/queries/` are independent — each exports async functions that accept a `Database` instance as their first argument.

`sdk.ts` demonstrates using `@anthropic-ai/claude-agent-sdk` (`query()`) to drive Claude programmatically with restricted tools.

## Query Pattern

All query functions use `async/await` directly with the `sqlite` wrapper (which returns Promises):

```typescript
export async function getCustomerByEmail(
  db: Database,
  email: string,
): Promise<any> {
  return await db.get(`SELECT * FROM customers WHERE email = ?`, [email]);
}

export async function listOrders(db: Database): Promise<any[]> {
  return await db.all(`SELECT * FROM orders`);
}
```

- `db.get()` — single row or undefined
- `db.all()` — array of rows
- Always use parameterized queries (`?` placeholders)

## Hooks

Configured in `.claude/settings.local.json` and run automatically on file edits:

**PreToolUse** (Write/Edit/MultiEdit):

- `hooks/query_hook.js` — uses Claude Agent SDK to check if a new query in `src/queries/` duplicates existing functionality; blocks the edit with feedback if so. Currently disabled by an early `process.exit(0)`.

**PostToolUse** (Write/Edit/MultiEdit):

- Runs `prettier --write` on the edited file
- `hooks/tsc.js` — runs a full TypeScript type check via the TS compiler API and blocks if there are errors

All tool calls are also logged to `pre-log.json` / `post-log.json`.

## Critical Guidance

- All database queries must be written in `./src/queries/`

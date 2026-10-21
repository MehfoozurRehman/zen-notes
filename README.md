# Cloud Notes

A full-stack notes board. Notes are stored in PostgreSQL and shown as coloured cards that can be edited, recoloured and deleted. Built with Next.js 14 Server Actions, React 18 and Prisma.

## The problem

Quick notes get lost in a single browser tab. This app keeps them in a database and shows them as a grid of cards, so they persist across visits. The interface is deliberately minimal: one page, no navigation, no setup beyond a database connection.

## What it does

- **Add a note**: a prompt asks for a title, and the note is created with a blue background and an empty body.
- **Edit in place**: the title, body and colour are editable on each card. Changing any field shows a save button, which calls a server action.
- **Delete**: a button on the card removes the note.
- **Newest first**: notes are read with `orderBy: { id: "desc" }` and laid out in a responsive grid (`repeat(auto-fill, minmax(300px, 1fr))`).
- **Required titles**: the add flow checks the title with Zod, and `createNote` and `updateNote` on the server reject an empty title.
- **Note route**: `/[noteId]` reads the id from the URL. It is a placeholder that prints the id.

## How it's built

- **Framework**: Next.js 14.2 App Router, React 18, TypeScript.
- **Data**: PostgreSQL with Prisma 5. One `Note` model (`id`, `title`, `content`, `color`, `createdAt`, `updatedAt`). The schema and a first migration are in `prisma/`.
- **Mutations**: `actions/note.ts` holds `"use server"` functions for create, update and delete. Each one calls `revalidatePath("/")`, so the list refreshes without a separate API layer.
- **Client components**: `AddNote.tsx` and `Card.tsx` call those actions with `useTransition`, so the buttons show a pending state.
- **Decision 1: no REST API.** The server actions are the only write path. Forms and buttons call them directly.
- **Decision 2: fresh reads and one Prisma client.** The home page is a server component that calls `unstable_noStore()`, so every visit reads the database. `lib/prisma.ts` keeps the Prisma client on `globalThis` in development, so hot reloads do not open new connections.
- **Types**: `prisma/interfaces.ts` is generated from the schema by `prisma-generator-typescript-interfaces`.

## Running it locally

You need a PostgreSQL database.

```bash
pnpm install
```

Create a `.env` file in the project root with a `DATABASE_URL` pointing at your database. Then:

```bash
pnpm migrate
pnpm dev
```

Open http://localhost:3000.

Other scripts: `pnpm generate` (regenerate the Prisma client), `pnpm studio` (Prisma Studio), `pnpm build`, `pnpm lint`.

## What I learned / what I'd do next

- The earlier README described categories, search and Markdown. None of those are in the code, so they are not listed as features.
- Server actions catch their errors and only call `console.error`. A failed save gives the user no message. Return an error state to the client.
- There is no authentication and no ownership model. Any visitor can edit or delete any note. Add user accounts before using this beyond a local demo.
- `/[noteId]` is a placeholder. Either build the note view or remove the route.
- The `.env` file is tracked in git. Move it out of the repository and keep only an example file.
- There are no tests.

## Author

Created by [Mehfooz-ur-Rehman](https://github.com/MehfoozurRehman).

# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a Nuxt 3 application integrated with Firebase, deployed on Vercel. It currently implements a simple kanban board (roadmap board) with three stages. The application uses Tailwind CSS for styling and supports both development and production Firebase environments.

## Development Commands

### Root Directory (Nuxt App)

- `npm run dev` - Start Nuxt dev server
- `npm run build` - Build for production
- `npm run preview` - Preview production build locally
- `npm run lint` - Lint TypeScript, JavaScript, and Vue files
- `npm run format` - Format code with Prettier
- `npm run fix` - Run formatter and linter with auto-fix

### Functions Directory

- `cd functions` first, then:
- `npm run serve` - Start Firebase emulator for functions (includes build step)
- `npm run build` - Compile TypeScript functions
- `npm run build:watch` - Watch mode for TypeScript compilation
- `npm run deploy` - Deploy functions to Firebase
- `npm run logs` - View Firebase function logs

## Architecture

### Firebase Integration

**Environment-Aware Configuration**: The app uses a dynamic environment variable naming scheme in `nuxt.config.ts`:

- Variables are prefixed with `NODE_ENV` (e.g., `development_apiKey`, `production_apiKey`)
- Firebase config is loaded via `runtimeConfig.public`
- Copy `.env` from `@.env` template and configure per environment

**Initialization Flow**:

1. `app/middleware/firebaseInit.global.client.ts` - Client-side Firebase app initialization as global middleware
2. `app/plugins/firebaseInit.global.ts` - (Note: deprecated/duplicate, middleware is the active initialization)
3. `app/plugins/useFirestore.ts` - Provides Firebase Auth and Firestore instances via `$auth`, `$db`, `$GoogleAuthProvider`, and `$ensureUserDoc` helper

**Functions Setup**:

- Located in `functions/` directory with separate package.json
- Region: `asia-northeast1`
- Development mode auto-connects to emulator on `localhost:5002`
- Global max instances set to 10 for cost control
- Requires Node.js 22

### Project Structure

```
app/
├── components/       # Vue components (e.g., KanbanBoard.vue)
├── pages/           # Nuxt pages with auto-routing
├── middleware/      # Global middleware (Firebase init)
├── plugins/         # Nuxt plugins (Firebase services)
└── assets/css/      # Global styles (SCSS + Tailwind)

functions/
├── src/
│   └── index.ts    # Firebase Cloud Functions entry point
├── lib/            # Compiled JavaScript output
└── package.json    # Separate dependency management
```

### Firebase Services Used

- **Firestore**: Database with user document management
- **Auth**: Google OAuth provider configured
- **Functions**: Cloud Functions with emulator support
- **Hosting**: Configured via firebase.json

The `ensureUserDoc` helper automatically creates/updates user documents in Firestore on authentication, storing `createdAt`, `updatedAt`, `email`, and `displayName`.

## Setup Notes

1. Install dependencies in both root and functions directories
2. Create `.env` from `@.env` template
3. Update `.firebaserc` with your Firebase project name
4. Run both `npm run dev` (root) and `npm run serve` (functions) for full local development

## Deployment

- Frontend: Automatically deployed to Vercel (configured with `nitro.preset: "vercel"`)
- Functions: Deploy via `npm run deploy` from functions directory
- Pre-deploy hooks automatically run lint and build in functions

## Key Implementation Details

- Tailwind CSS configured via `@nuxtjs/tailwindcss` module
- Global SCSS at `app/assets/css/common.scss`
- TypeScript throughout with strict configuration
- ESLint with Nuxt, TypeScript, and Prettier configurations

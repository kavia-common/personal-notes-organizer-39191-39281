# Notes Frontend (Svelte)

A modern Svelte app for creating, viewing, editing, and deleting personal notes. Styled with the Ocean Professional theme (blue primary with amber accents) and built on SvelteKit.

## Quick start

- Install dependencies:
  npm install

- Start dev server (port 3000, strictPort):
  npm run dev

- Build:
  npm run build

- Preview production build:
  npm run preview

The dev server is configured for preview systems on port 3000 with CORS enabled in vite.config.ts.

## Environment variables

The app reads the backend API base URL from the following (in order):
- VITE_API_BASE
- VITE_BACKEND_URL

Set one of these in your .env:

VITE_API_BASE=https://your-backend.example.com

No URLs are hardcoded in the code. If none are provided, the app will use a relative "/api" path.

Other available envs (predefined in the workspace): VITE_FRONTEND_URL, VITE_WS_URL, VITE_NODE_ENV, VITE_ENABLE_SOURCE_MAPS, VITE_PORT, VITE_TRUST_PROXY, VITE_LOG_LEVEL, VITE_HEALTHCHECK_PATH, VITE_FEATURE_FLAGS, VITE_EXPERIMENTS_ENABLED

## API expectations

This UI expects a backend with the following endpoints:
- GET    /notes           -> Note[]
- POST   /notes           -> Note
- PUT    /notes/:id       -> Note
- DELETE /notes/:id       -> { id: string }

Note model:
{
  id: string;
  title: string;
  content: string;
  tags: string[];
  createdAt?: string;
  updatedAt?: string;
}

## Features

- Sidebar with search and tag filter
- Notes list with quick selection
- Editor with title, content, and tags
- Create, update, delete actions
- Optimistic UI for create/update/delete
- Local storage state persistence (selection, filters, latest notes cache)
- Loading, empty, and error states
- Responsive layout

## Style guide

Ocean Professional:
- Primary: #2563EB
- Secondary/Accent: #F59E0B
- Error: #EF4444
- Background: #f9fafb
- Surface: #ffffff
- Text: #111827

Smooth transitions, subtle shadows, and rounded corners are used per the theme guidance.

## Notes

- For production deployments, configure the appropriate SvelteKit adapter if needed.
- Ensure your backend enables CORS for the frontend origin during development.

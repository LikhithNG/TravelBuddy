# TravelBuddy Frontend

The React frontend for TravelBuddy: a chat interface for planning trips, with maps, weather and flight search.

## Stack

React 18, TypeScript, Vite, Tailwind CSS, shadcn/ui (Radix), TanStack Query, React Router, Auth0 and the Google Maps JavaScript API.

## Pages

| Route | Page | Sign-in required |
|---|---|---|
| `/` | Landing page | No |
| `/waitlist` | Waitlist | No |
| `/auth-callback` | Auth0 sign-in callback | No |
| `/home` | Home | Yes |
| `/chat/:id` | Chat with the travel agents | Yes |
| `/chatmain/:id` | Main chat view | Yes |
| `/chatmaps` | Chat with a map | Yes |

## Settings

Create `client/.env`:

```
VITE_AUTH0_DOMAIN=your-tenant.auth0.com
VITE_AUTH0_CLIENT_ID=your-auth0-client-id
VITE_AUTH0_REDIRECT_URI=http://localhost:3000/auth-callback
VITE_AUTH0_AUDIENCE=your-api-identifier
VITE_BASE_URL=http://localhost:8000
VITE_GOOGLE_MAPS_API_KEY=your-google-maps-key
```

## Run

```bash
cd client
npm install
npm run dev
```

The app runs at http://localhost:3000 and forwards `/api` requests to the backend at http://localhost:8000, so start the backend too (see the main [README](../README.md)).

Other scripts: `npm run build`, `npm run preview`, `npm run lint`.

## Layout

```
src/
├── pages/        Landing, home, chat, map chat, flight search, waitlist
├── components/   Chat, map, weather, navigation and shadcn/ui components
├── api/          Backend calls (chat and users)
├── auth/         Auth0 provider and protected routes
├── hooks/        Chat, audio recording, auto-scroll and other hooks
├── db/           IndexedDB storage
└── AppRouter.tsx Routes
```

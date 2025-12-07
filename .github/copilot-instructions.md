Project overview
This repository contains a React frontend (single-page app) under `frontend/` and an (empty) `backend/` folder. The frontend uses client-side routing (`react-router`), Zustand stores for global state, `axios` wrapped in `frontend/src/lib/axios.js` for HTTP requests, and `socket.io-client` for real-time messaging in `frontend/src/store/useAuthStore.js` and `useChatStore.js`.

What to know up-front
- Frontend is the active code: `frontend/src/` houses the app. There is no committed `package.json` or lockfile in the workspace — verify package manager and scripts locally before running anything.
- Backend is not present in this repo; many features assume an API + socket server running on `http://localhost:3000` (or `VITE_BACKEND_PORT`).
- Authentication and socket cookies: `axiosInstance` is configured with `withCredentials: true`. The backend must support cookie-based auth and CORS with credentials.

Key files and patterns (use these as examples)
- `frontend/src/lib/axios.js`: central axios instance, base URL determined by `import.meta.env.MODE` and `VITE_BACKEND_PORT`. Response interceptor shows how errors are surfaced and user-facing `toast` messages are triggered.
- `frontend/src/store/useAuthStore.js`: zustand store that manages `authUser`, socket creation (`io()`), and reconnection handling. Agents should respect the socket lifecycle helpers `connectSocket` / `disconnectSocket`.
- `frontend/src/store/useChatStore.js`: chat logic — optimistic UI (temporary message IDs), a `pendingMessages` Set to avoid duplicate socket-inserted messages, and many socket event listeners (`newMessage`, `reactionAdded`, etc.). Use this file as canonical reference for realtime message flows and duplicate-avoidance strategies.
- `frontend/src/hooks/useKeyboardSound.js` and `public/sounds/`: audio is loaded from absolute `/sounds/*`; note code assumes static hosting at `/`.
- `frontend/src/pages/*` and `frontend/src/components/*`: UI split across pages and small presentational/connected components. `App.jsx` performs the initial `checkAuth()` call.

Developer workflows & important commands (verify locally)
- Inspect `package.json` in your local clone (if present). Common commands likely used for this Vite-style app:
  - Install deps: `npm install` or `pnpm install` or `yarn`
  - Dev server: `npm run dev` (or the script named `dev` in `package.json`)
  - Build: `npm run build`
  - Preview/Serve build: `npm run preview` or `serve -s dist`
- Backend: run your API/socket server on port `3000` (or set `VITE_BACKEND_PORT`) to match the frontend defaults. If backend is elsewhere, update `frontend/src/lib/axios.js` and `getSocketURL()` in `useAuthStore.js`.

Project-specific conventions (do not change without caution)
- Global state: use Zustand stores in `frontend/src/store/*` rather than React Context for the features implemented here.
- Socket handling: always attach/detach listeners in store methods. Prefer using provided helper methods (`subscribeToMessages`, `unsubscribeFromMessages`) — avoid attaching duplicate listeners.
- Optimistic messages: `useChatStore.sendMessage` creates temporary messages with `_id` like `temp-<timestamp>-<rand>` and tracks real messages in `pendingMessages` to prevent socket duplicates. When changing messaging flows, preserve this mechanism.
- Audio: toggles persisted in `localStorage` under `isSoundEnabled` (boolean). Audio files are played by constructing `new Audio('/sounds/…')`.
- Error handling: user-facing errors are shown via `react-hot-toast`. Changes that affect request/response shapes should update toast messages accordingly.

Integration & environment hints
- Env vars referenced in code: `import.meta.env.MODE` and `VITE_BACKEND_PORT` (frontend assumes Vite-style env injection). If adding other env vars, follow the same pattern and check the build tool's env handling.
- HTTP API paths: frontend calls paths under `/api` (see `axiosInstance.baseURL`) and specific endpoints like `/auth/*`, `/messages/*` — use these when mocking or implementing backend routes.
- Socket URL: default `http://localhost:3000`, falling back to `window.location.origin` in production.

When making changes
- Small UI tweaks: update components in `frontend/src/components` or `frontend/src/pages`. Run the dev server locally and test flows that touch auth and sockets.
- Backend changes or API mock: run a local API that responds to `/api/auth/*` and `/api/messages/*`, and provides socket.io endpoints compatible with `useAuthStore` listeners.
- Avoid editing socket connect options (transports, withCredentials) unless you understand cookie/CORS implications.

Examples to copy from the codebase
- How to send a message (optimistic): look at `useChatStore.sendMessage` for constructing a temporary message, posting `multipart/form-data`, updating `pendingMessages`, and replacing the optimistic message on success.
- How to connect socket safely: `useAuthStore.connectSocket` shows listener lifecycle and reconnection handling; new agents should follow the same pattern.

If something is missing or unclear
- I could not find a committed `package.json` or backend code in the repository. If those are in a sibling folder or ignored by Git, please add or point me to them so I can include exact run/build commands and backend integration details.

If this looks good, I will commit this file. Reply with any extra run commands or project-specific scripts you want included.

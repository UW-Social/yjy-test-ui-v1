# UW Social – Developer Onboarding

Welcome aboard! This guide highlights what you need to become productive quickly across the Vue/Vite frontend and Spring Boot backend.

## 1. Project at a glance
- **Frontend (`Frontend/`)**: Vue 3 + TypeScript app bootstrapped in `src/main.ts`. The `bootstrap()` function wires up Pinia, Element Plus, Firebase, Google Maps, and waits for the user store (`useUserStore.loadUser()`) before mounting to keep auth in sync.
- **Mobile-first routing**: `src/router/index.ts` uses hash history (required for GitHub Pages). Routes call `isMobile()` at runtime to pick the desktop or `src/components/mobile/` screens; keep hash routing intact when adding routes.
- **Data layer**: Firestore drives most content. `useEventStore` (`src/stores/event.ts`) reads the `events` collection, filters out past events, and powers cards/detail overlays.
- **Personalization**: `src/components/EventList.vue` first renders events newest-first, then reorders in the background using ONNX embeddings (`components/models/embedding_distance.ts`) matched against the signed-in user’s tags.
- **Backend (`Backend/demo/`)**: Spring Boot 3.4 app exposing `/api/users`. It persists Firebase-authenticated users to MySQL via Spring Data JPA and expects the frontend to send Firebase tokens (see `Frontend/src/api/index.ts`).
- **ML assets**: ONNX model files live under `Frontend/public/models` and tokenizers under `Frontend/public/tokenizer`; `embedding_distance.ts` references these paths directly.

## 2. Local setup
### Prerequisites
- Node 18+ and npm
- Java 21 (Adoptium or Azul) with Maven wrapper support
- A Firestore project, Firebase Storage bucket, and Google Maps API key
- MySQL instance for the Spring Boot service (can be local Docker)

### Environment variables
Create `Frontend/.env` with:
```
VITE_FIREBASE_API_KEY=...
VITE_FIREBASE_AUTH_DOMAIN=...
VITE_FIREBASE_PROJECT_ID=...
VITE_FIREBASE_STORAGE_BUCKET=...
VITE_FIREBASE_MESSAGING_SENDER_ID=...
VITE_FIREBASE_APP_ID=...
VITE_FIREBASE_MEASUREMENT_ID=...
VITE_GOOGLE_API_KEY=...
```

## 3. Frontend workflows
- Install dependencies once:
```pwsh
cd Frontend
npm install
```
- Start the dev server (hash routing served on port 5173):
```pwsh
npm run dev
```
  Visit `http://localhost:5173/#/` to keep routing consistent.
- Build for production:
```pwsh
npm run build
```
- Global boot: keep initialization changes inside the `bootstrap()` function in `src/main.ts` so the app still waits for `loadUser()` before mounting.
- Firestore writes: `EventForm.vue` persists both structured recurrence (`schedule` union in `src/types/event.ts`) and legacy `startTime`/`endtime` timestamps. Populate both when adding new write paths, or event listings and filters will break.
- Personalization: The ONNX helper caches sessions/tokenizers—reuse `getSession()` / `getTokenizer()` instead of instantiating per call to avoid wasm cost.
- Analytics: Follow the defensive `typeof window.gtag === 'function'` pattern (see `Navbar.vue`) when logging events so local dev without Analytics doesn’t break.

## 4. Backend workflows
- Run the Spring Boot API:
```pwsh
cd Backend/demo
.\mvnw.cmd spring-boot:run
```
  Ensure the MySQL schema exists and env vars above are set.
- Execute tests:
```pwsh
.\mvnw.cmd test
```
- `UserController` currently accepts POST `/api/users`; extend services/repositories under `Backend/demo/src/main/java/com/example/demo` (package-split into `controller`, `service`, `repository`, `model`). Keep JPA entities annotated with Jakarta imports.
- CORS adjustments should flow through `CORS_ALLOWED_ORIGINS`; avoid hardcoding origins in controllers.

## 5. Common conventions & pitfalls
- **Routing**: stick with `createWebHashHistory()`; switching to history mode will break GitHub Pages URLs.
- **Scheduling types**: any change to `EventSchedule` union requires updating `formatEventSchedule()` and ensuring downstream components handle new fields.
- **Event filtering**: `useEventStore.fetchEvents()` filters past events by checking `endtime`. Always set end timestamps in Firestore documents.
- **Mobile components**: keep mobile-specific UI in `src/components/mobile/`; desktop views live in `src/views/`. When adding new routes, provide both variants if needed.
- **API client**: All HTTP requests should go through `src/api/index.ts` to inherit token handling and 401 redirects.
- **Static assets**: When moving ONNX or tokenizer files, update paths in `embedding_distance.ts`; the current code falls back to CDN for wasm but expects local model files under `/models`.

## 6. Next steps for new developers
1. Set up Firebase + Maps credentials and confirm `.env` values.
2. Provision a MySQL schema locally; insert credentials via env vars.
3. Run the frontend and backend locally to ensure login + event flows work (create a dummy Firebase user).
4. Review `EventForm.vue` and `EventList.vue` to understand event creation and ranking logic before making UI changes.
5. Skim the Pinia stores (`src/stores/*.ts`) to learn where shared state lives.

Questions or gaps? Reach out in the project channel; keeping this doc current is part of the onboarding checklist.

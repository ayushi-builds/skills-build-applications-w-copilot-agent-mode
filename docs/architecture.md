# OctoFit Tracker architecture

OctoFit Tracker is a three-tier application for a school fitness program. The starter code includes API contracts and intentionally incomplete implementation points so you can use GitHub Copilot agent mode in a test-driven loop.

![Three-tier OctoFit architecture](../.github/images/octofit-architecture.svg)

## Tier boundaries

- **Presentation tier**: React 19, Vite, React Router, Bootstrap, and a shared API client in `octofit-tracker/frontend/src`.
- **Logic tier**: Express and TypeScript routes/services in `octofit-tracker/backend/src` expose `/api/*` behavior on port `8000`.
- **Data tier**: Mongoose models and seed data in `octofit-tracker/backend/src/models` and `src/scripts/seed.ts` persist to MongoDB on port `27017`.

## Ports and environment rules

- MongoDB: `27017`
- Express API: `8000`
- Vite frontend: `5173`
- The browser calls relative `/api` paths. Vite proxies them to `http://localhost:8000`, so the unauthenticated API port stays private in Codespaces. `VITE_API_BASE_URL` can override the base URL.

## API contract

| Capability | Method and path | Expected behavior |
| --- | --- | --- |
| Activity tracking | `GET /api/activities`, `POST /api/activities` | List and create valid activity records. POST accepts `userId`, `teamId`, `type`, and `minutes`; `userId` and `teamId` must be well-formed IDs for an existing user and team; otherwise respond with a deliberate 4xx (never a 500 from a cast error). A created activity stores the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt`. |
| Teams | `GET /api/teams`, `POST /api/teams` | List and create teams with members. Created and listed teams keep the submitted `name` and `mascot`. Reject a team without a name with `400`. |
| Leaderboard | `GET /api/leaderboard` | Return teams ordered by the sum of their activity points, highest first. Each entry has `teamName`, `totalPoints`, and a 1-based `rank`. |
| Workout suggestions | `GET /api/workouts/suggestions/:userEmail` | Recommend workouts from recent activity patterns for the matching seeded user's email address. The first suggestion's `focus` matches that user's goal or recent activity, with a `reason` explaining why. An email with no matching user returns a deliberate 4xx (for example, `404`). |

## Implementation decisions

Record your plan here before editing code.

- Presentation tier decision: Build the React 19 UI as a small Vite app that renders activities, teams, leaderboard, and workout suggestions as separate routes. Keep fetch logic centralized in the shared API client, and show loading, empty, success, and error states with accessible content so the UI can be tested without hidden mock behavior.
- Logic tier decision: Implement the `/api/*` experience with Express and TypeScript service functions. Keep route handlers thin, enforce business validation and deliberate 4xx responses, and isolate database-specific concerns from the HTTP layer.
- Data tier decision: Model users, teams, activities, and workouts with Mongoose schemas and stable seeded demo data. Use ObjectId references for relationships, keep seed operations idempotent, and reconcile existing demo-user team membership instead of clearing unrelated data.
- Activity tracking approach: Accept `userId`, `teamId`, `type`, and `minutes` for `POST /api/activities`, validate both references and required fields, and store the submitted `user`, `team`, `type`, and `minutes` plus computed `points` and `occurredAt` values.
- Team management approach: Require `name` and `mascot` for team creation, default `members` to an empty array, and expose member IDs on created/listed teams while rejecting invalid payloads with `400`.
- Leaderboard approach: Aggregate activity points by team, sort descending by total points, and return a leaderboard with `teamName`, `totalPoints`, and a 1-based `rank` for each team.
- Workout suggestions approach: Match a user email to a seeded user, derive the relevant training focus from recent activity patterns or fitness goals, and return ranked workout suggestions with a `title`, `focus`, `reason`, and related metadata while returning a deliberate 4xx for unknown users.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend` — validates Mongoose schema contracts and seed reliability before API behavior is implemented.
- API command: `npm run test:api --prefix octofit-tracker/backend` — verifies HTTP behavior, validation, list/create endpoints, leaderboard ordering, and workout suggestions against the backend contract.
- Frontend command: `npm test --prefix octofit-tracker/frontend` — checks that the React UI requests the right API routes and renders loading, empty, success, and failure states in an accessible way.
- Backend build command: `npm run build --prefix octofit-tracker/backend` — confirms the Express/TypeScript backend compiles cleanly with the expected interfaces and types.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend` — confirms the React/Vite frontend compiles and bundles without type or build errors.

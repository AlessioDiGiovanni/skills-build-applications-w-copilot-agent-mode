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

- Presentation tier decision:
    Use the existing routes and React components; every view handles loading, results, empty states, and accessible errors.
- Logic tier decision:
    Keep validation, business rules, and model access in backend services; Express routes translate failures into deliberate HTTP responses.
- Data tier decision:
    Complete the Mongoose schemas with required fields, types, and references that match the contracts; keep the seed idempotent.
- Activity tracking approach:
    Validate format and presence of user/team before requesting or saving; create activities with references, minutes, calculated points and date.
- Team management approach:
    Reject teams without name with 400; keep name and mascot and accept valid members.
- Leaderboard approach:
    Sum activity points by team, sort them by highest total and rank them from 1.
- Workout suggestions approach:
    Find the user by email and base the first suggestion on objective or recent activities; explain the reason and return the appropriate 4xx if the user does not exist.

## Implementation order

1. Complete the data models and pass the data contract tests.
2. Implement the API behavior and pass the API contract tests.
3. Connect the frontend views to the API and pass the presentation tests.

## Test plan

Record the commands you run and what each failure means.

- Data-tier command: `npm run test:data --prefix octofit-tracker/backend`
    Currently 4 failures out of 5; it points out missing constraints in the User, Team, Activity, and Workout schemas. The seed test passes.
- API command: `npm run test:api --prefix octofit-tracker/backend`
    Currently 4 failures out of 4; check validation and creation, leaderboard, and suggestions.
- Frontend command: `npm test --prefix octofit-tracker/frontend`
    currently 15 failures; check rendering of views and API state management.
- Backend build command: `npm run build --prefix octofit-tracker/backend`
    detects backend TypeScript errors.
- Frontend build command: `npm run build --prefix octofit-tracker/frontend`
    detects TypeScript or Vite compilation errors.

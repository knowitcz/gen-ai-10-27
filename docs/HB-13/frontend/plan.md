# HB-13: Client Screens — Frontend Plan

## Definition of Done
- Docs: the user flow ([user-flow.md](user-flow.md)) and the wireframes ([wireframes.md](wireframes.md)) exist.
- DEV: the client list and the client detail are connected to the API.
- QA: both screens have tests covering all screen states.
- OPS: the application can be built and run, the build is served by FastAPI.

## Stack
| Concern       | Choice                                   | Why                                                     |
|---------------|------------------------------------------|---------------------------------------------------------|
| Framework     | React + TypeScript                       | The most widely used SPA framework                      |
| Build tool    | Vite                                     | Standard build tool for new React projects              |
| Routing       | React Router                             | De-facto standard router for React                      |
| Styling       | Tailwind CSS + shadcn/ui                 | Ready-made accessible components (table, card, …)       |
| Tests         | Vitest + React Testing Library           | Shares the Vite configuration, tests what the user sees |

Use the latest stable versions (`npm view <package> version`) and do not follow outdated setups (Create React App, `tailwind.config.js` of Tailwind 3, …).

## Location
The frontend lives in `resources/web/` next to the other resources served by the backend. The production build goes to `resources/web/dist/`.

## API
The existing endpoints are used as they are, the backend is not changed.

| Screen        | Endpoint                   | Response                                                             |
|---------------|----------------------------|----------------------------------------------------------------------|
| Client list   | `GET /api/v1/client`       | `ClientRead[]` — `id`, `name`, `national_number`                      |
| Client detail | `GET /api/v1/client/{id}`  | `ClientDetailRead` — `ClientRead` + `accounts: AccountRead[]`         |

`AccountRead` contains `id`, `name`, `balance`, `type` (`CASH`, `SAVINGS`) and `client_id`. The shapes are taken from `/openapi.json`, not guessed.

### Findings
- The task asks for a client **number**, but the API exposes only `id`. The screens show `id` as the client number; a dedicated number is a question for the analysis, not for the frontend.
- The paths use singular `client`. Do not "fix" them to `clients` in the API client.
- The backend has no CORS configuration. During development the Vite dev server proxies `/api` to `http://localhost:8000`; in production the build is served from the same origin.
- `balance` is an integer without a currency. It is displayed as a formatted number, the currency is not invented.

## Routes
| Route          | Screen        |
|----------------|---------------|
| `/`            | redirect to `/clients` |
| `/clients`     | Client list   |
| `/clients/:id` | Client detail |
| anything else  | Not found     |

## Components
- `AppLayout` — header with the application name, a link to the client list, an outlet for the page.
- `ClientsPage` — loads the clients and renders them in a table; a row links to the detail.
- `ClientDetailPage` — loads one client, shows its data and a table of its accounts, has a link back to the list.
- `api/clients.ts` — typed functions `getClients()` and `getClient(id)`; a non-OK response becomes an error carrying the HTTP status.
- `types/client.ts` — `Client`, `ClientDetail`, `Account` matching the API.

Each page keeps its own `loading` / `error` / `data` state; no global state library is needed for two screens.

## Screen States
| State              | Client list                               | Client detail                                    |
|--------------------|-------------------------------------------|--------------------------------------------------|
| Loading            | spinner in place of the table             | spinner in place of the card                     |
| Error              | message + "Try again" button              | message + "Try again" button                     |
| Empty              | "No clients found."                       | "The client has no accounts." in the accounts table |
| Not found (404)    | —                                         | "Client not found." + link back to the list      |
| Invalid id in URL  | —                                         | handled as not found, no request is sent         |

## Tests
- `ClientsPage`: renders the clients, a row links to `/clients/{id}`, loading, error with retry, empty list.
- `ClientDetailPage`: renders the client and the accounts, no accounts, 404, other errors, invalid id.
- `api/clients.ts`: calls the right URLs, turns a non-OK response into an error with the status.
- The API is mocked by stubbing `fetch`; the tests do not need a running backend.

## Build and Run
- Development: `npm install`, `npm run dev` in `resources/web/` while `uvicorn app.main:app --reload` runs.
- Production: `npm run build`; FastAPI serves `resources/web/dist/index.html` on `/` and on every unknown path (after the API routers), and the assets from `resources/web/dist/assets/`. `/docs` stays available.
- `README.md` describes both.

## Verification
- `npm run lint`, `npm test` and `npm run build` pass.
- The screens are compared with the wireframes in a browser, including the error state (backend stopped) and the not-found state (`/clients/999`).
- `uv run pytest` still passes.

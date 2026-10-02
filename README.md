# TaskPilot ✈

An AI-assisted task planner. Describe a goal in plain English, and TaskPilot turns it into a
small, editable action plan you can save, complete and revisit.

A **React Native** app (Expo SDK 57) on top of a full-stack **PERN** backend:
**P**ostgreSQL, **E**xpress, **R**eact Native, **N**ode.js.

> "Prepare for a junior developer interview next week."
>
> → Research the company and role — 30 min
> → Review JavaScript and React fundamentals — 90 min
> → Practice behavioural interview answers — 45 min
> → Update portfolio links and CV — 30 min
> → Do one mock interview — 45 min

---

## Features

| Area | What it does |
| --- | --- |
| **Auth** | Register / log in with JWT, bcrypt-hashed passwords, protected routes, per-user ownership checks |
| **Tasks** | Create, view, edit, complete and delete tasks with a category, priority and optional due date |
| **Steps** | Add, tick off, edit and delete subtasks; live "3 of 5 complete" progress |
| **AI planner** | Turns a goal into a structured plan (title, summary, priority, steps + time estimates) that you **review and edit before saving** |
| **Dashboard** | Stats, category filters, search, empty states, loading and error states |
| **Footer** | A persistent bottom bar on every signed-in screen: **Home**, **Add task** and **Edit** - adding or revising work is always one tap, whatever is on screen |
| **Honest AI** | No key means the planner says so and refuses - never returns filler steps disguised as AI output |

### Design

One warm, earthy palette, defined once in `client/src/theme.js`:

| Role | Colour | Used for |
| --- | --- | --- |
| Page | `#FBF6EC` cream | background |
| Surface | `#FFFDF8` | cards, panels |
| Terracotta | `#C2673F` | the **only** saturated colour - buttons, focus, the AI accent |
| Green | `#6B8A57` | progress bars, completed checkboxes, "done" |
| Text | `#2E2822` warm near-black | body copy |

Terracotta and green sit together well because both are low-saturation warm tones. Nothing
competes with the one colour the user is meant to tap. Muted earth tones also read as a
considered product rather than a generated one.

The AI planner is deliberately one screen: type a goal, press **Make a plan**, read the steps.
Category and time budget are hidden behind one line, because three always-visible fields made it
feel like a form to fill in rather than a thing to ask.

Adding work used to be awkward: a small "Add manually" ghost button in the dashboard header, plus a
"+ New task" button that only existed when the list was empty. Neither was reachable once you had
scrolled, and the empty-state one vanished as soon as you created a task. So every signed-in screen
now ends with the same three-slot bar: **Home**, **Add task**, **Edit**.

| Slot | Behaviour |
| --- | --- |
| **Home** | Back to the task list from anywhere. Shown *selected* on the dashboard, the way a tab behaves, so it never offers a tap that does nothing. |
| **Add task** | The dominant action - centred, twice the width of its neighbours, in the terracotta that `theme.js` already reserves for "the one thing to tap". |
| **Edit** | Context-aware rather than duplicated per screen: on the dashboard it opens a searchable task picker and lands the chosen task **already in its edit form**; on a task it edits that task. |

Two deliberate details. The bar is a flex sibling of the content rather than a floating overlay, so
the last card is never hidden underneath it. And **Edit** is idempotent, not a toggle - tapping it
while the form is open must not close it and silently discard what the user had typed, so **Cancel**
lives on the form itself, next to **Save**.

---

## Tech stack

**Mobile app** — React Native 0.86, Expo SDK 57, Expo Router (file-based), React 19.2,
AsyncStorage, Ionicons, Safe Area Context. Plain `StyleSheet` (no CSS, no UI framework).

**Backend** — Node.js 22.13+, Express 5 (ESM), Prisma 7, PostgreSQL, JWT, bcrypt, Zod,
OpenAI SDK (Structured Outputs), Helmet, Morgan, Nodemon

---

## Project structure

```
taskpilot_project/
├── package.json                 # npm workspaces root
└── taskpilot/
    ├── server/
    │   ├── prisma/
    │   │   ├── schema.prisma     # Models, relations, enums
    │   │   ├── migrations/
    │   │   └── seed.js          # Demo data
    │   ├── prisma.config.js      # Prisma 7 config (schema path + datasource URL)
    │   ├── .env / .env.example
    │   └── src/
    │       ├── config/env.js     # Validated environment
    │       ├── controllers/      # HTTP layer: read req, call service, send res
    │       ├── services/         # Business logic + database + AI calls
    │       ├── routes/           # Endpoints and middleware wiring
    │       ├── middleware/       # auth, validation, error handling
    │       ├── validators/       # Zod request schemas
    │       ├── utils/            # prisma client, jwt, ApiError, asyncHandler
    │       ├── app.js            # Express app wiring
    │       └── server.js         # Entry point, graceful shutdown
    │
    └── client/                     # React Native app (Expo)
        ├── app.json                # Expo config (name, icon, splash, plugins)
        ├── babel.config.js
        ├── assets/                 # App icon / splash / adaptive icon
        ├── app/                    # Expo Router: file-based routes
        │   ├── _layout.jsx         # Providers + auth guard
        │   ├── index.jsx
        │   ├── (auth)/             # Unauthenticated screens
        │   │   ├── login.jsx
        │   │   └── register.jsx
        │   └── (app)/              # Authenticated screens
        │       ├── dashboard.jsx
        │       └── task/[id].jsx
        └── src/
            ├── api/client.js     # fetch wrapper + token storage
            ├── context/          # AuthContext (session state)
            ├── components/       # Reusable UI
            ├── lib/              # Constants and formatting helpers
            └── theme.js          # Design tokens (colours, spacing, radius)
```

### MV(C) mapping

| Layer | Responsibility | Where |
| --- | --- | --- |
| **Model** | Data + relations | `server/prisma/schema.prisma`, queries in `services/` |
| **View** | What the user sees | `client/app` (screens), `client/src/components` |
| **Controller** | Validate input, call a service, shape the response | `server/src/controllers/` |
| **Routes** | URL → middleware → controller | `server/src/routes/` |
| **Service** | Business logic, AI calls, DB workflows | `server/src/services/` |

---

## Getting started

### 1. Prerequisites

- **Node.js 22.13+** (required by Expo SDK 57; Node 22.18+ is also needed for the Prisma 7
  client, which is generated as TypeScript and relies on Node's native type stripping)
- **PostgreSQL 14+** running locally
- To run on a device: the **Expo Go** app, or an Android emulator / iOS simulator

### 2. Install dependencies

```bash
npm install
```

This installs both workspaces from the repository root.

### 3. Configure the database and environment

Create the database (once):

```sql
CREATE DATABASE taskpilot;
```

Then copy the env template and fill it in:

```bash
cd taskpilot/server
cp .env.example .env      # Windows: copy .env.example .env
```

```env
DATABASE_URL="postgresql://postgres:YOUR_PASSWORD@localhost:5432/taskpilot?schema=public"
JWT_SECRET="<a long random string>"
CLIENT_URL="http://localhost:8081"   # Metro dev server; used for CORS
OPENAI_API_KEY=""          # optional - leave empty to run without AI
OPENAI_TIMEOUT_MS=30000    # cap on the OpenAI call, in ms
AI_RATE_LIMIT=10           # plan generations per user per minute
```

Generate a strong secret with:

```bash
node -e "console.log(require('crypto').randomBytes(48).toString('base64url'))"
```

### 4. Create the database schema

```bash
npm run db:migrate     # applies migrations and generates the Prisma client
npm run db:seed        # optional: demo account + sample tasks
```

### 5. Run the app

You need the API running first. In one terminal:

```bash
npm run dev:server     # API -> http://localhost:5000
```

Then start the mobile app in a second terminal:

```bash
npm run dev:client     # Metro bundler + QR code for Expo Go
```

Scan the QR code with **Expo Go** on your phone, or press `a` / `i` in the terminal for an
Android emulator / iOS simulator.

### 6. Running on a physical phone (same Wi-Fi)

The phone and the computer must be on the **same Wi-Fi network**. On a device, `localhost`
means the phone itself, so the app is told the laptop's LAN address instead.

1. Find the computer's LAN IP (Windows: `ipconfig`, look for IPv4 on Wi-Fi):

   ```bash
   ipconfig
   ```

2. Point the app at it in `taskpilot/client/.env` (git-ignored, created for you):

   ```env
   EXPO_PUBLIC_API_URL=http://192.168.0.114:5000/api
   ```

   **Re-check this IP after rejoining a network** — it changes, and a stale value is the most
   common reason the phone shows "Cannot reach the API".

3. Allow the ports through Windows Firewall when prompted (Node.js, inbound, **Private**
   networks only):

   ```powershell
   New-NetFirewallRule -DisplayName "TaskPilot API" -Direction Inbound -LocalPort 5000 -Protocol TCP -Action Allow -Profile Private
   New-NetFirewallRule -DisplayName "Expo Metro" -Direction Inbound -LocalPort 8081 -Protocol TCP -Action Allow -Profile Private
   ```

4. Start both, with Metro in LAN mode so it serves the QR on your network interface:

   ```bash
   npm run dev:server                 # terminal 1
   cd taskpilot/client && npx expo start --lan    # terminal 2
   ```

5. Open **Expo Go** on the phone and scan the QR code in the terminal.

Verify before scanning — both should return `200`:

```bash
curl http://192.168.0.114:5000/api/health     # {"status":"ok",...}
curl http://192.168.0.114:8081/status          # packager-status:running
```

> **iOS simulator** reaches the host normally, but a **physical iPhone** will not open a plain
> `http://` LAN URL for `fetch` in every configuration. If requests fail only on iOS, see
> [ATS / cleartext](#ios-cannot-reach-the-api).

#### If the phone cannot reach the API

| Symptom | Cause | Fix |
| --- | --- | --- |
| "Cannot reach the API at http://192.168.0.114:5000/api" | Phone is on a different network, or client IP is stale | Re-run `ipconfig`, update `client/.env`, restart Metro |
| Request hangs then fails | Windows Firewall blocking inbound | Add the two rules above (step 3) |
| App loads, login fails | Server not running, or bound to localhost | Server listens on all interfaces; confirm `curl` from the phone's browser |
| Works on web, fails on device | Metro not in LAN mode | Use `npx expo start --lan`, not `expo start` |

#### iOS cannot reach the API

iOS blocks cleartext `http://` by default (App Transport Security). Expo Go permits LAN
`http://` for development, so this normally works out of the box. If it does not, a development
build needs an ATS exception in `app.json` under `ios.infoPlist.NSAppTransportSecurity`. Plain
Android is unaffected.

Other useful commands:

```bash
npm run android        # open directly on a connected Android device/emulator
npm run ios            # open on an iOS simulator (macOS only)
npm run web            # run in a browser via react-native-web
```

Sign in with the seeded account:

```
demo@taskpilot.app / Demo1234!
```

Or register a new account from the sign-up screen.

### Connecting a real device

`localhost` on a phone means the phone itself, not your laptop. The app reads
`EXPO_PUBLIC_API_URL` if you set it, and otherwise derives your machine's LAN address from the
Metro dev server, so a device on the same Wi-Fi usually just works. To point it somewhere
specific:

```bash
EXPO_PUBLIC_API_URL="http://192.168.1.20:5000/api" npm run dev:client
```

> Android emulators reach the host at `10.0.2.2` rather than `localhost`.

---

## Enabling the AI planner

The OpenAI key lives **only on the server** — the app never sees it. The app calls the Express
API, and Express is the only process that talks to OpenAI.

1. Put your key in `taskpilot/server/.env`:

   ```env
   OPENAI_API_KEY="sk-..."
   OPENAI_MODEL="gpt-4o-mini"
   ```

   **Or let the script do it** — it reads the key without echoing it, writes it to the
   git-ignored `.env`, and immediately verifies it against OpenAI:

   ```bash
   cd taskpilot/server
   npm run set:key
   ```

   The key is never passed as a command argument, so it does not end up in shell history. If
   OpenAI rejects it the script **removes it again**, so the app is never left looking enabled
   while every request fails.

2. Check it works, without opening the app:

   ```bash
   npm run doctor:ai
   ```

   ```
   TaskPilot AI doctor
     model          : gpt-4o-mini
     key configured : yes
     OK    source: ai
     OK    title : Prepare for a junior developer interview
     OK    steps : 5
   ```

3. Restart the server. The banner will read `AI planner : enabled`.

**Without a key the app still works** — every other feature is unaffected. The planner's *Make a
plan* button is disabled and the panel explains what to configure. It deliberately does **not**
return a generic starter plan: five hard-coded steps are indistinguishable from a real AI plan, and
a user would save them believing the model produced them. A keyless run fails loudly instead.

---

## The AI implementation

Nine files. This is the whole surface area:

| File | Role |
| --- | --- |
| `client/src/components/AiPlanner.jsx` | **The whole AI UI.** Goal box → *Make a plan* → editable preview → *Save plan* |
| `client/src/components/PlanSteps.jsx` | The editable step list shown in the preview |
| `server/src/routes/ai.routes.js` | One route: `POST /api/ai/generate-task-plan` |
| `server/src/middleware/aiRateLimit.js` | Per-user limit so the paid endpoint cannot be hammered |
| `server/src/controllers/ai.controller.js` | Shapes the response for the client |
| **`server/src/services/ai.service.js`** | **The actual integration** — prompt, OpenAI call, validation, retry |
| `server/src/validators/ai.validator.js` | `generatePlanSchema` (input) and `aiTaskPlanSchema` (output contract) |
| `server/src/config/env.js` | Key, model, timeout, rate limit |
| `server/doctor-ai.mjs` | `npm run doctor:ai` — checks the key end to end |

### How the planning stage works

```
goal text
   ↓  AiPlanner.jsx
POST /api/ai/generate-task-plan      ← auth → rate limit → validate body
   ↓  ai.service.js  generateTaskPlan()
   OpenAI Responses API  ·  Structured Outputs  ·  zodTextFormat(aiTaskPlanSchema)
   ↓
re-validate → clean → clamp minutes → check the time budget
   ↓
{ source, plan, budget, warnings, substitutions }
   ↓
AiPlanner.jsx renders it for review — nothing is saved yet
   ↓  *Save plan*
POST /api/tasks   →  Task + Subtask rows
```

### The one thing that matters most

**The model is never trusted.** Structured Outputs guarantees the *shape* of the reply, not that
its values are sensible, so `ai.service.js` re-validates everything before it reaches the client:

- Titles are trimmed and markdown stripped; blank titles are dropped rather than replaced
- Minutes are clamped into 5–180 — **and any value the model did not actually produce is flagged**
  in `warnings`, so a placeholder is never shown as if the model chose it
- If the plan overruns the hours the user gave, the response says so rather than silently
  truncating the model's work
- **If every step comes back unusable, the plan is demoted to the local fallback** — it is never
  reported as `source: 'ai'`

That last rule matters more than it looks. An earlier version substituted five hard-coded steps
when the model returned nothing usable but still claimed `source: 'ai'`. That is exactly the kind
of quiet lie this whole guardrail set exists to prevent.

### Two design decisions worth keeping

- **The AI never sets `category` or `dueDate`.** The user picks the category — letting the model
  override a deliberate choice is worse than suggesting one. It never invents a due date either.
- **Nothing is auto-saved.** *Make a plan* only asks. The plan is shown, editable, and only written
  to the database when the user presses *Save plan*. The goal text is saved as the task
  `description` so the original intent survives.

### Testing it

```bash
npm run verify:ai     # 14 checks: retry rules, budget, clamping, honest fallback, end-to-end
npm run doctor:ai     # calls the real provider and names any problem
```

`verify-ai.mjs` runs the **real OpenAI SDK** against a local stand-in server, so the retry rules and
failure handling are exercised for real without needing a key or spending anything.

---

### Guardrails around the call

| Guard | What it does |
| --- | --- |
| Keyless server | `POST /api/ai/generate-task-plan` returns **503** with setup instructions. No filler steps. |
| Timeout | `OPENAI_TIMEOUT_MS` (default 30s) caps the call, so a hung provider cannot spin forever. |
| Retry | One extra attempt, only for transient failures (timeout, connection error, 429, 5xx). A 400/401 is never retried. |
| Budget check | If the plan exceeds the `availableHoursPerWeek` the user gave, the response carries a warning naming the real total. The plan is not silently truncated. |
| Substituted values | An estimate the model did not supply is filled with 30 minutes **and reported** in `warnings` / `substitutions`, so a number the model never produced is never shown as if it were. |
| Rate limit | `AI_RATE_LIMIT` (default 10) requests per user per minute → **429** with `Retry-After`. |

The local fallback is still reachable, but only when a key exists and the provider genuinely fails
— in which case `source` is `'fallback'` and the UI says the plan is not AI output.

> **Rate limiter caveat:** counters live in process memory. That is correct for this single-server
> app, but if it is ever deployed behind multiple instances each keeps its own counters. Swap the
> `Map` in `server/src/middleware/aiRateLimit.js` for Redis at that point.

---

## API reference

All `/api/tasks` and `/api/subtasks` routes require `Authorization: Bearer <token>`.

### Auth

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/auth/register` | Create an account, returns a JWT |
| `POST` | `/api/auth/login` | Log in, returns a JWT |
| `GET` | `/api/auth/me` | Current user |

### Tasks

| Method | Endpoint | Description |
| --- | --- | --- |
| `GET` | `/api/tasks` | List your tasks (`?category=`, `?completed=`, `?search=`) |
| `POST` | `/api/tasks` | Create a task (optionally with nested `subtasks`) |
| `GET` | `/api/tasks/:taskId` | One task with its steps and progress |
| `PATCH` | `/api/tasks/:taskId` | Update any subset of fields |
| `DELETE` | `/api/tasks/:taskId` | Delete a task (steps cascade) |
| `GET` | `/api/tasks/stats` | Totals and per-category breakdown |

### Subtasks

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/tasks/:taskId/subtasks` | Add a step |
| `PUT` | `/api/tasks/:taskId/subtasks` | Replace the whole plan (used by "regenerate") |
| `PATCH` | `/api/subtasks/:subtaskId` | Edit / tick a step |
| `DELETE` | `/api/subtasks/:subtaskId` | Remove a step |

### AI and utility

| Method | Endpoint | Description |
| --- | --- | --- |
| `POST` | `/api/ai/generate-task-plan` | `{ goal, category?, availableHoursPerWeek? }` |
| `GET` | `/api/health` | Liveness check |
| `GET` | `/api/config` | Reports whether the AI planner is enabled |

---

## Data model

```prisma
User  1 ────< Task  1 ────< Subtask
```

- `User → Task` and `Task → Subtask` both use `onDelete: Cascade`.
- `category` and `priority` are Prisma enums, mirrored in the Zod validators.
- Indexes on `Task(userId, createdAt)` and `Subtask(taskId)` back the ownership queries.

### Ownership

Every read and write is scoped by `userId` **inside the database query**, never by filtering in
JavaScript afterwards. Accessing someone else's task returns `404`, not `403`, so the API does
not leak which ids exist.

---

## Security notes

- Passwords are hashed with **bcrypt** (cost factor 12) and never returned by any endpoint.
- Login compares against a dummy hash when the email is unknown, so response timing does not
  reveal whether an account exists.
- The AI key and database credentials stay on the server.
- `helmet` sets security headers; all request bodies are validated with Zod before reaching a
  service.
- Unexpected errors are logged server-side and returned as a generic `500`.
- The session token is stored in AsyncStorage, which on a real device is app-sandboxed
  (iOS Keychain / Android app storage) rather than a shared browser localStorage.

---

## Scripts

Run from the repository root:

| Script | Description |
| --- | --- |
| `npm run dev:server` | Start the API with nodemon |
| `npm run dev:client` | Start Metro and show the Expo QR code |
| `npm run android` | Open the app on a connected Android device/emulator |
| `npm run ios` | Open the app on an iOS simulator (macOS only) |
| `npm run web` | Run the app in a browser via react-native-web |
| `npm run build` | Export production JS bundles for iOS, Android and web |
| `npm start` | Run the API in production mode |
| `npm run db:migrate` | Apply migrations + generate the Prisma client |
| `npm run db:seed` | Load demo data |
| `npm run db:studio` | Open Prisma Studio |
| `npm run db:reset` | Drop, re-migrate and re-seed the database |

---

## Troubleshooting

**`JWT_SECRET must be at least 16 characters`** — your `.env` is missing a strong secret.

**`Could not connect to PostgreSQL`** — check that the server is running and that
`DATABASE_URL` (including the password) is correct.

**`AI not configured` in the UI** — `OPENAI_API_KEY` is empty. Everything else still works; the
planner's *Generate plan* button is disabled and the panel says what to set. Add the key to
`taskpilot/server/.env`, restart the server, and run `npm run doctor:ai` in `taskpilot/server`
to confirm the provider is reachable before debugging through the app. See
[Enabling the AI planner](#enabling-the-ai-planner).

**Tapping a task shows "We could not open that task"** — the detail screen requests
`/api/tasks/<id>` with the id it received from the route. If the screen reads the wrong
`useLocalSearchParams()` key it asks for `/api/tasks/undefined`, the API answers 404, and the
not-found panel is shown for a task that does exist. The route is `task/[id].jsx`, so the
parameter is `id`. `client/verify.mjs` asserts the correct URL is requested, so this cannot
regress unnoticed.

**`The AI planner allows 10 requests per minute`** — the rate limiter is working. Wait for
`Retry-After` to elapse, or raise `AI_RATE_LIMIT` in `taskpilot/server/.env`.

**Port already in use** — change `PORT` in `taskpilot/server/.env`. If the app cannot reach the
API at all, check the console for the resolved API URL and set `EXPO_PUBLIC_API_URL`.

---

## Ideas for future versions

- AI task rewriter for vague goals ("study coding" → a specific, achievable task)
- AI weekly schedule built from due dates and available study hours
- Task templates ("Job application", "Exam preparation", "Portfolio project")
- Per-category completion insights
- Push notifications for due tasks

---

Built with React Native, Expo, Node.js, Express, PostgreSQL, Prisma and JWT authentication.
#   t a s k - p i l o t  
 
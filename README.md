# Angular 21 Auth Boilerplate

A beginner-friendly Angular 21 boilerplate with a complete authentication flow.

**Features:**
- Email sign up + email verification
- Login / logout
- JWT auth header for API requests
- Refresh tokens (cookie-based) + auto-refresh before access token expiry
- Forgot password + reset password
- Role-based authorization (User & Admin)
- Admin area for account management
- Profile area for viewing/updating your own account

---

## Table of Contents

1. [Prerequisites](#1-prerequisites)
2. [Clone & Run](#2-clone--run)
3. [Run with Fake Backend (no API needed)](#3-run-with-fake-backend)
4. [Run with a Real API](#4-run-with-a-real-api)
5. [Using the App](#5-using-the-app)
6. [How Authentication Works](#6-how-authentication-works)
7. [Authorization](#7-authorization)
8. [Project Structure](#8-project-structure)
9. [Troubleshooting](#9-troubleshooting)

---

## 1. Prerequisites

Make sure you have the following installed before continuing:

- [Node.js](https://nodejs.org/) (LTS version recommended)
- npm (included with Node.js)
- Angular CLI (optional but recommended):

```bash
npm install -g @angular/cli
```

---

## 2. Clone & Run

### Step 1 — Clone the repository

```bash
git clone https://github.com/g13-ter/angular-auth-boilerplate.git
cd angular-auth-boilerplate
```

### Step 2 — Install dependencies

```bash
npm install
```

### Step 3 — Start the app

```bash
npm start
```

The app will open automatically at `http://localhost:4200`.

> By default the app runs with a **fake backend** — no real API or database needed. See [section 3](#3-run-with-fake-backend) for details.

---

## 3. Run with Fake Backend

The fake backend is built into the app and runs entirely in the browser. It intercepts all HTTP calls and handles them in-memory using `localStorage`.

It is **enabled by default** — just run `npm start` and everything works out of the box.

**Default admin account:**

| Email | Password | Role |
|---|---|---|
| `admin@example.com` | `admin` | Admin |

You can also register new accounts through the UI.

> To reset the fake backend data, clear `localStorage` for `localhost:4200` in your browser DevTools, or remove the key `angular-15-signup-verification-boilerplate-accounts`.

---

## 4. Run with a Real API

### Step 1 — Set your API URL

Edit `src/environments/environment.ts`:

```ts
export const environment = {
  production: false,
  apiUrl: 'http://localhost:4000'   // <-- change this to your API URL
};
```

### Step 2 — Disable the fake backend

In `src/app/app.module.ts`, find and remove (or comment out) the `fakeBackend` entry in the `providers` array.

### Step 3 — Start the app

```bash
npm start
```

Your API must expose the endpoints listed in [section 6](#6-how-authentication-works).

---

## 5. Using the App

### A) Register & verify email

1. Click **Register** and fill in the form
2. With the fake backend, a verification link appears as an on-screen alert — click it
3. With a real API, check your email for the verification link
4. After verifying, log in with your credentials

### B) Login / Logout

- Use the **Login** page to sign in
- Click **Logout** in the nav bar to sign out

### C) Forgot password / Reset password

1. Go to **Forgot Password** and enter your email
2. Click the reset link (on-screen alert for fake backend, email for real API)
3. Set a new password

### D) Profile

- View and update your own account details under the **Profile** section

### E) Admin area

- Only accounts with the `Admin` role can access `/admin`
- Admins can view, create, edit, and delete accounts

---

## 6. How Authentication Works

This app uses two tokens:

| Token | Stored in | Lifetime |
|---|---|---|
| Access token (JWT) | Memory (`BehaviorSubject`) | Short-lived |
| Refresh token | HttpOnly cookie | Long-lived |

### Key files

| File | Purpose |
|---|---|
| `src/environments/environment.ts` | API base URL |
| `src/app/_services/account.service.ts` | Login, logout, register, refresh, etc. |
| `src/app/_helpers/app.initializer.ts` | Restores session on page load |
| `src/app/_helpers/jwt.interceptor.ts` | Attaches `Authorization: Bearer <token>` |
| `src/app/_helpers/error.interceptor.ts` | Auto-logout on 401 / 403 |

### Login flow

1. Login component calls `AccountService.login(email, password)`
2. API returns an `Account` object with `jwtToken`
3. Token is stored in memory and a refresh timer is scheduled
4. All subsequent requests get `Authorization: Bearer ...` via the JWT interceptor

### Refresh token flow

1. On page load, `APP_INITIALIZER` calls the refresh endpoint to restore the session
2. The refresh token cookie is sent automatically (`withCredentials: true`)
3. A new access token is returned and the timer resets

### Required API endpoints

```
POST   /accounts/authenticate
POST   /accounts/refresh-token
POST   /accounts/revoke-token
POST   /accounts/register
POST   /accounts/verify-email
POST   /accounts/forgot-password
POST   /accounts/validate-reset-token
POST   /accounts/reset-password
GET    /accounts              (Admin only)
GET    /accounts/:id
POST   /accounts              (Admin only)
PUT    /accounts/:id
DELETE /accounts/:id
```

---

## 7. Authorization

Routes are protected by `AuthGuard` (`src/app/_helpers/auth.guard.ts`):

- Not logged in → redirected to `/account/login`
- Logged in but insufficient role → redirected to `/`

Role restrictions use the `Role` enum from `src/app/_models/role.ts` and are configured in `src/app/app-routing.module.ts`.

---

## 8. Project Structure

```
src/app/
├── _helpers/       Guards, interceptors, app initializer, fake backend
├── _models/        Shared types: Account, Role, Alert
├── _services/      AccountService, AlertService
├── _components/    Shared UI components (e.g. Alert banner)
├── account/        Auth screens: login, register, verify, forgot, reset
├── profile/        User profile screens
├── admin/          Admin-only screens (account list, add/edit)
└── home/           Home screen
```

Bootstrap 5 is loaded via CDN in `src/index.html`.

---

## 9. Troubleshooting

### App redirects to login after page refresh

If using a real API, make sure it:
- Sets a `refresh-token` HttpOnly cookie on login
- Supports `POST /accounts/refresh-token`

### Cookies not being sent (cross-origin API)

This app uses `withCredentials: true` for auth requests. Your API must:
- Enable CORS with credentials
- Return `Access-Control-Allow-Credentials: true`
- Set `Access-Control-Allow-Origin` to the exact frontend origin (not `*`)

### Run unit tests

```bash
npm test
```

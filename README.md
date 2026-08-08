# Project Manager

Project Manager is a Node.js + Express backend API for simple project and membership management with a full authentication flow (email verification, refresh tokens, password reset) and MongoDB persistence.

## Features

- User registration & login with email verification
- Access + refresh JWT tokens (cookie + header support)
- Forgot / reset password flow via email
- Protected endpoints for current user, logout, change password
- Projects: create, update, delete, list
- Project membership: add, list, update role, remove
- Health check endpoint

## Tech stack

- Node.js (ES modules)
- Express 5
- MongoDB + Mongoose
- JSON Web Tokens (JWT)
- bcrypt, nodemailer, mailgen

## Project layout

```text
src/
   app.js                 # Express app, middleware, route mounting
   index.js               # bootstrap (DB connect + server)
   controllers/           # request handlers
   db/                    # MongoDB connection helper
   middlewares/           # auth, validation, upload
   models/                # Mongoose models (User, Project, ProjectMember, ...)
   routes/                # route definitions
   utils/                 # helpers (mail, responses, errors)
   validators/            # request validators
public/                  # served static files (images, etc.)
```

## Requirements

- Node.js (v18+ recommended)
- A reachable MongoDB instance
- An SMTP account for outgoing emails (verification / reset)

## Install & run

Install dependencies:

```bash
npm install
```

Start in development (uses `nodemon`):

```bash
npm run dev
```

Start in production:

```bash
npm start
```

## Environment variables

Required variables (create a `.env` file in project root):

- `PORT` — server port (default: `3000`)
- `MONGO_URI` — MongoDB connection string
- `CORS_ORIGIN` — comma-separated allowed origins (optional)
- `ACCESS_TOKEN_SECRET` — secret for access tokens
- `ACCESS_TOKEN_EXPIRY` — access token lifetime (e.g. `1d`)
- `REFRESH_TOKEN_SECRET` — secret for refresh tokens
- `REFRESH_TOKEN_EXPIRY` — refresh token lifetime (e.g. `10d`)
- `SMTP_HOST` — SMTP server host
- `SMTP_PORT` — SMTP server port
- `SMTP_USER` — SMTP username
- `SMTP_PASSWORD` — SMTP password
- `FORGOT_PASSWORD_REDIRECT_URL` — frontend URL used in password reset emails

Example `.env`:

```env
PORT=3000
MONGO_URI=mongodb://127.0.0.1:27017/projectmanager
CORS_ORIGIN=http://localhost:5173
ACCESS_TOKEN_SECRET=your-access-token-secret
ACCESS_TOKEN_EXPIRY=1d
REFRESH_TOKEN_SECRET=your-refresh-token-secret
REFRESH_TOKEN_EXPIRY=10d
SMTP_HOST=smtp.example.com
SMTP_PORT=587
SMTP_USER=your-smtp-user
SMTP_PASSWORD=your-smtp-password
FORGOT_PASSWORD_REDIRECT_URL=http://localhost:5173/reset-password
```

## Scripts

- `npm run dev` — start with `nodemon` (development)
- `npm start` — start production server

## API (base path `/api/v1`)

**Health**

- `GET /api/v1/healthcheck` — simple status check

**Auth**

- `POST /api/v1/auth/register` — register a new user
  - body: `{ email, username, password, fullName? }`
- `POST /api/v1/auth/login` — login with email + password
  - body: `{ email, password }`
- `GET /api/v1/auth/verify-email/:verificationToken` — verify email
- `POST /api/v1/auth/refresh-token` — refresh access token (uses `refreshToken` cookie or body)
- `POST /api/v1/auth/forgot-password` — request password reset
  - body: `{ email }`
- `POST /api/v1/auth/reset-password/:resetToken` — reset password
  - body: `{ newPassword }`
- `POST /api/v1/auth/logout` — logout (clears token cookies) — protected
- `POST /api/v1/auth/current-user` — fetch current user — protected
- `POST /api/v1/auth/change-password` — change password — protected
  - body: `{ oldPassword, newPassword }`
- `POST /api/v1/auth/resend-email-verification` — resend verification email — protected

> Protected routes require a valid access token provided either via the `Authorization: Bearer <token>` header or the `accessToken` cookie.

**Projects** (all project routes require authentication)

- `GET /api/v1/projects` — list projects for current user
- `POST /api/v1/projects` — create project
  - body: `{ name, description? }`
- `GET /api/v1/projects/:projectId` — get project details (permission checked)
- `PUT /api/v1/projects/:projectId` — update project (admin only)
- `DELETE /api/v1/projects/:projectId` — delete project (admin only)

**Project members**

- `GET /api/v1/projects/:projectId/members` — list members
- `POST /api/v1/projects/:projectId/members` — add/update member (admin only)
  - body: `{ email, role }` where `role` is one of `admin`, `project_admin`, `member`
- `PUT /api/v1/projects/:projectId/members/:userId` — update member role (admin only)
- `DELETE /api/v1/projects/:projectId/members/:userId` — remove member (admin only)

## Validation and response format

- Request validation is implemented with `express-validator`. Validation middleware returns `422` with details when payloads are invalid.
- Successful responses use a shared wrapper: `{ statusCode, data, message, success }`.
- Errors are thrown as `ApiError` and follow the same wrapper with `success: false`.

## Static files

- `public/` is served statically (uploads stored under `public/images`).

## Notes & caveats

- Cookies for tokens are set as `httpOnly` and `secure` in the code — when testing locally you may need to adjust cookie handling or run over HTTPS to persist cookies across browsers.
- Some internal links generated by controllers may reference different paths; the source of truth for routes is the files in `src/routes/`.

## License

ISC

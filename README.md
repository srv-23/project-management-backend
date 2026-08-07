# Project Manager

Project Manager is a Node.js and Express backend API for user authentication, email verification, password recovery, and protected user session handling. It uses MongoDB with Mongoose, JWT-based access and refresh tokens, and SMTP email delivery for verification and reset flows.

## Features

- User registration and login
- Email verification flow
- Refresh token support
- Protected current-user and logout endpoints
- Change password and forgot-password/reset-password flow
- Health check endpoint
- MongoDB persistence with Mongoose
- Cookie-based token storage with CORS support

## Tech Stack

- Node.js
- Express 5
- MongoDB
- Mongoose
- JSON Web Tokens
- bcrypt
- nodemailer
- mailgen

## Project Structure

```text
src/
  app.js                 # Express app configuration
  index.js               # Server bootstrap and database connection
  controllers/           # Route handlers
  db/                    # MongoDB connection
  middlewares/           # Auth and validation middleware
  models/                # Mongoose models
  routes/                # API route definitions
  utils/                 # Shared helpers and response/error wrappers
  validators/            # express-validator schemas
public/                  # Static assets
```

## Prerequisites

- Node.js 18 or newer
- MongoDB connection string
- SMTP account for outgoing emails

## Installation

1. Install dependencies:

   ```bash
   npm install
   ```

2. Create a `.env` file in the project root and set the required variables.

3. Start the development server:

   ```bash
   npm run dev
   ```

4. Start the production server:

   ```bash
   npm start
   ```

## Environment Variables

The application reads the following environment variables:

- `PORT` - Server port, defaults to `3000`
- `MONGO_URI` - MongoDB connection string
- `CORS_ORIGIN` - Allowed frontend origin or comma-separated origins
- `ACCESS_TOKEN_SECRET` - Secret used to sign access tokens
- `ACCESS_TOKEN_EXPIRY` - Access token lifetime
- `REFRESH_TOKEN_SECRET` - Secret used to sign refresh tokens
- `REFRESH_TOKEN_EXPIRY` - Refresh token lifetime
- `SMTP_HOST` - SMTP server host
- `SMTP_PORT` - SMTP server port
- `SMTP_USER` - SMTP username
- `SMTP_PASSWORD` - SMTP password
- `FORGOT_PASSWORD_REDIRECT_URL` - Frontend URL used in password reset emails

Example:

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

## API Base Path

All routes are mounted under `/api/v1`.

## Endpoints

### Health Check

- `GET /api/v1/healthcheck`

Returns a simple server status response.

### Auth

- `POST /api/v1/auth/register`
- `POST /api/v1/auth/login`
- `GET /api/v1/auth/verify-email/:verificationToken`
- `POST /api/v1/auth/refresh-token`
- `POST /api/v1/auth/forgot-password`
- `POST /api/v1/auth/reset-password/:resetToken`
- `POST /api/v1/auth/logout`
- `GET /api/v1/auth/current-user`
- `POST /api/v1/auth/change-password`
- `POST /api/v1/auth/resend-email-verification`

## Request Notes

- `register` expects `email`, `username`, and `password`, and the user model also requires `fullName`.
- `login` uses `email` and `password`.
- Protected routes require a valid access token in either the `Authorization: Bearer <token>` header or the `accessToken` cookie.
- Token refresh uses the `refreshToken` cookie or a `refreshToken` field in the request body.

## Response Format

The API uses a shared response wrapper with a consistent structure:

```json
{
  "statusCode": 200,
  "data": {},
  "message": "Success",
  "success": true
}
```

Errors use a similar wrapper with `success: false` and a status code/message pair.

## Static Assets

The `public/` directory is served statically by Express, so files placed there are available directly from the server.

## Notes

- Cookies for access and refresh tokens are set as `httpOnly` and `secure`.
- Email verification and password reset links are generated from the configured SMTP and frontend redirect settings.
- The server will fail to start if MongoDB cannot be reached.

## License

ISC

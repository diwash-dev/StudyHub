# StudyHub - A Web-Based Academic Resource Management System

StudyHub is a full-stack academic resource platform for organizing and accessing study materials by semester and subject. The application provides a React-based student-facing library and an Express API backed by PostgreSQL and Prisma.

The currently integrated resource workflow supports PDF notes, question papers, and lab reports. The client also includes syllabus and project browsing pages, while those pages currently use client-side data rather than registered server API routes.

## Project status

This repository is an active project rather than a packaged production release. Deployment URLs, repository URLs, and screenshots are intentionally left as placeholders until they are confirmed.

## Features

### For students and public visitors

- Browse the home page and academic resource categories.
- Browse the semester and subject catalog returned by the API.
- View and download published notes.
- View and download published question papers.
- View and download published lab reports.
- Filter resource listings by semester and subject in the client.
- Open syllabus and project pages included in the client application.
- Use informational About and Contact pages.

### For administrators

- Sign in through the admin login page.
- Authenticate API requests with a JWT stored by the client.
- Manage notes with create, read, update, and delete operations.
- Manage question papers, including draft/published status, paper type, year, and PDF replacement.
- Manage lab reports, including draft/published status, experiment number, title, and PDF replacement.
- Use the admin dashboard, resource management pages, and settings UI included in the client.
- Upload PDF files up to 10 MB; uploads are validated as PDFs before processing.

Question-paper and lab-report administration is restricted to users with the `ADMIN` role. Note mutations require a valid bearer token in the current API implementation.

## Technology stack

### Client

- React 19
- React Router 7
- Vite 8
- Tailwind CSS 4 with `@tailwindcss/vite`
- Lucide React icons
- JavaScript and JSX

### Server

- Node.js with native ES modules
- Express 5
- PostgreSQL via `pg`
- Prisma 7 with `@prisma/adapter-pg`
- JWT authentication with `jsonwebtoken`
- Password hashing with `bcryptjs`
- Multipart PDF uploads with `multer`
- Cloudinary for uploaded PDF storage
- CORS and `dotenv`
- `tsx` for starting the server and `nodemon` for development

## Architecture

The project is split into independently runnable client and server applications:

```text
StudyHub/
├── client/
│   ├── public/                 # Static public assets
│   └── src/
│       ├── components/         # Shared UI and admin components
│       ├── pages/              # Student, viewer, and admin login pages
│       ├── assets/             # Client assets
│       ├── App.jsx             # Client-side route definitions
│       ├── Home.jsx            # Home page
│       └── main.jsx            # React entry point
├── server/
│   ├── controllers/            # Request handlers and database operations
│   ├── middleware/             # JWT authorization and PDF upload validation
│   ├── routes/                 # Express API route modules
│   ├── services/               # Cloudinary integration
│   ├── prisma/
│   │   ├── schema.prisma       # PostgreSQL data model
│   │   └── migrations/         # Database migrations
│   ├── server.js               # Express and Prisma bootstrap
│   └── prisma7.config.ts       # Prisma CLI configuration
└── README.md
```

At runtime, the client calls the server using the API base URL configured by `VITE_API_URL`. If that variable is not set, the client uses `http://localhost:5000/api`.

## Screenshots

Add project screenshots to a `docs/screenshots/` directory and replace the placeholders below:

| View | Screenshot |
| --- | --- |
| Home page | ![StudyHub home page](docs/screenshots/home.png) |
| Resource listing | ![StudyHub resource listing](docs/screenshots/resource-listing.png) |
| PDF viewer | ![StudyHub PDF viewer](docs/screenshots/pdf-viewer.png) |
| Admin dashboard | ![StudyHub admin dashboard](docs/screenshots/admin-dashboard.png) |

## Live demo and repository

- Live demo: `https://learn-desk-ebon.vercel.app/`
- GitHub repository: `https://github.com/diwash-dev/StudyHub`


## Prerequisites

- Node.js compatible with the installed Vite, Express, and Prisma dependencies
- npm
- A PostgreSQL database
- A Cloudinary account for PDF uploads

## Local setup

Clone the repository and install dependencies separately for the two applications:

```bash
git clone https://github.com/your-org/your-studyhub-repository.git
cd StudyHub

cd server
npm install

cd ../client
npm install
```

Create the server environment file described below. Do not commit it. Then configure the client API URL if the server will not use its default local address.

### Server environment variables

Create `server/.env` using safe placeholder values:

```env
DATABASE_URL="postgresql://DB_USER:DB_PASSWORD@DB_HOST:5432/DB_NAME?schema=public"
JWT_SECRET="replace-with-a-long-random-secret"
PORT=5000

# Use either the three individual values or CLOUDINARY_URL.
CLOUDINARY_CLOUD_NAME="your-cloud-name"
CLOUDINARY_API_KEY="your-cloudinary-api-key"
CLOUDINARY_API_SECRET="your-cloudinary-api-secret"
# CLOUDINARY_URL="cloudinary://API_KEY:API_SECRET@CLOUD_NAME"
```

`DATABASE_URL`, `JWT_SECRET`, and `PORT` are read by the server. Cloudinary is configured using either `CLOUDINARY_CLOUD_NAME`, `CLOUDINARY_API_KEY`, and `CLOUDINARY_API_SECRET`, or a `CLOUDINARY_URL`. Use real values only in a local, ignored `.env` file or in the deployment provider's secret manager.

### Client environment variables

Optional: create `client/.env.local`:

```env
VITE_API_URL="http://localhost:5000/api"
```

Vite exposes variables prefixed with `VITE_` to the browser. Never place private credentials in the client environment.

## Database and Prisma

The Prisma datasource is PostgreSQL. The schema currently contains:

- `User` with `STUDENT` and `ADMIN` roles
- `Semester` and `Subject`
- `Note`
- `Project`
- `LabReport`
- `PastQuestion`
- `Syllabus`
- `Result`
- `Notice`
- `Event`

Question papers and lab reports use `draft` and `published` statuses. Lab reports also enforce a unique `(subjectId, experimentNo)` pair.

From `server/`, generate the Prisma client and apply the checked-in migrations:

```bash
npx prisma generate
npx prisma migrate deploy
```

For local schema development when creating or changing migrations, use:

```bash
npx prisma migrate dev --name describe-your-change
```

The server connects to PostgreSQL during startup and exits if the database connection cannot be established.

## Running the applications

Open two terminals.

### Terminal 1: API server

```bash
cd server
npm run dev
```

The development server listens on `PORT` (default `5000`) and exposes the API under `/api`.

For a non-watch start:

```bash
cd server
npm start
```

### Terminal 2: React client

```bash
cd client
npm run dev
```

Use the local URL printed by Vite. The client build and lint commands are:

```bash
cd client
npm run build
npm run lint
```

## API reference

The following routes are registered by the Express server. Unless marked public, routes require `Authorization: Bearer <JWT>`. Multipart create and update routes accept one PDF in the `file` field.

### Authentication

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| `POST` | `/api/auth/admin/login` | Public | Authenticate an admin with `email` and `password`; returns a one-day JWT. |

### Catalog and notes

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/api/catalog` | Public | Return semesters and their subjects. |
| `GET` | `/api/notes` | Public | List notes with semester and subject information. |
| `GET` | `/api/notes/:id` | Public | Return one note. |
| `GET` | `/api/notes/:id/download` | Public | Download a note PDF. |
| `POST` | `/api/notes` | Authenticated | Create a note from form fields and a PDF. |
| `PUT` | `/api/notes/:id` | Authenticated | Update note metadata and optionally replace its PDF. |
| `DELETE` | `/api/notes/:id` | Authenticated | Delete a note. |

### Question papers

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/api/question-papers` | Public | List published question papers. |
| `GET` | `/api/question-papers/:id/view` | Public | Redirect to a published PDF for viewing. |
| `GET` | `/api/question-papers/:id/download` | Public | Download a published question paper PDF. |
| `GET` | `/api/question-papers/admin` | Admin | List all question papers, including drafts. |
| `GET` | `/api/question-papers/admin/:id` | Admin | Return one question paper for administration. |
| `POST` | `/api/question-papers` | Admin | Create a question paper. |
| `PUT` | `/api/question-papers/:id` | Admin | Update a question paper and optionally replace its PDF. |
| `DELETE` | `/api/question-papers/:id` | Admin | Delete a question paper. |

Question-paper uploads validate semester/subject relationships, years from 1990 through 2100, paper types (`University`, `College`, `Mid-Term`, `Internal`, `Model`, `Practical`, or `Other`), and `draft`/`published` status.

### Lab reports

| Method | Endpoint | Access | Purpose |
| --- | --- | --- | --- |
| `GET` | `/api/lab-reports` | Public | List published lab reports. |
| `GET` | `/api/lab-reports/:id/view` | Public | Redirect to a published PDF for viewing. |
| `GET` | `/api/lab-reports/:id/download` | Public | Download a published lab report PDF. |
| `GET` | `/api/lab-reports/admin` | Admin | List all lab reports, including drafts. |
| `GET` | `/api/lab-reports/admin/:id` | Admin | Return one lab report for administration. |
| `POST` | `/api/lab-reports` | Admin | Create a lab report. |
| `PUT` | `/api/lab-reports/:id` | Admin | Update a lab report and optionally replace its PDF. |
| `DELETE` | `/api/lab-reports/:id` | Admin | Delete a lab report. |

Lab-report uploads validate the experiment number, title, semester/subject relationship, and `draft`/`published` status.

There are currently no registered Express routes for projects, syllabuses, results, notices, or events, even though corresponding Prisma models and some client UI are present.

## Deployment notes

### Client on Vercel

The client includes a `vercel.json` rewrite that sends routes to `index.html`, which supports the React Router client-side routes. Configure `VITE_API_URL` in the Vercel project settings to point to the deployed API base URL, including `/api`, then deploy the `client` directory with:

```bash
npm run build
```

### Server

The server is a Node/Express application that binds to `0.0.0.0` and reads the platform-provided `PORT`, making it suitable for a Node hosting service. No provider-specific server deployment configuration is included in this repository; configure the start command as `npm start` and provide all server environment variables through the host's secret settings.

Render is not configured in the repository, but its Node web service can be used for the server with the same start command and environment variables. Confirm the service's working directory is `server`.

### PostgreSQL and Supabase

The Prisma datasource and migration lock explicitly target PostgreSQL. A managed PostgreSQL provider can be used by supplying its connection string as `DATABASE_URL`; Supabase is one possible PostgreSQL provider, but no Supabase-specific integration is present in the code.

### Cloudinary

Cloudinary is directly integrated for PDF uploads and cleanup. Configure one of the supported Cloudinary credential formats in the server environment. Notes are stored in the `studyhub/notes` folder; question papers and lab reports use separate folders.

## Future improvements

Potential next steps, based on the current implementation:

- Add server routes and client integration for projects, syllabuses, results, notices, and events.
- Add automated tests for API controllers, authorization, uploads, and client flows.
- Add a seed script or documented admin provisioning workflow for the first administrator.
- Add API validation and consistent error handling across all resources.
- Add production observability, rate limiting, and stricter CORS configuration.
- Add pagination and search for larger resource collections.
- Add verified deployment URLs, screenshots, and CI checks.

## License

No license file is currently included in the repository. Until a license is added, all rights are reserved by the copyright holder. Add a project-approved `LICENSE` file before distributing StudyHub under an open-source license.

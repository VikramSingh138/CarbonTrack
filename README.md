# CarbonTrack

CarbonTrack is a web application for tracking personal and organizational sustainability activity. Users can record lifestyle and transport information, review carbon-footprint dashboards, explore sustainability recommendations, and participate in certificate and voucher workflows. Organization accounts can manage departments and offer incentives; administrator screens support user and certificate oversight.

This repository contains two separately deployed applications:

- `frontend/` - React single-page application built with Vite.
- `backend/` - Express REST API backed by MongoDB.

## Contents

- [Features and user flows](#features-and-user-flows)
- [Technology](#technology)
- [Repository layout](#repository-layout)
- [Prerequisites](#prerequisites)
- [Install and run locally](#install-and-run-locally)
- [Environment variables](#environment-variables)
- [Build and deploy](#build-and-deploy)
- [API overview](#api-overview)
- [Troubleshooting](#troubleshooting)

## Features and user flows

CarbonTrack provides role-oriented screens for three account types:

### Individual users

1. Create an account and sign in at `/signup` or `/login`.
2. Complete the sustainability profile form.
3. Visit the dashboard to review recorded user data, emissions visualizations, comparisons, and available vouchers.
4. Use Monitoring and Analysis for environmental charts, recommendations, and product-impact analysis.
5. Track certificate goals, upload proof, and download certificates when available.

The client stores the authentication token in browser `localStorage` and sends it as a Bearer token to protected API routes.

### Organization accounts

Organization users have an organization dashboard with department data, reporting, and voucher-management screens. The API supports department creation, deletion and JSON report export, as well as offering vouchers.

### Administrators

Administrator screens provide user and organization views and certificate-goal verification workflows.

Some dashboards include sample or locally generated visualization data. Integrations such as MongoDB, Gemini, and Cloudinary require the credentials described below to provide their corresponding functionality.

## Technology

### Frontend

- React 18
- Vite 5 and `@vitejs/plugin-react`
- React Router for client-side navigation
- Zustand for persisted authentication state
- Axios and Fetch for HTTP requests
- Tailwind CSS, CSS modules/styles, and Framer Motion
- Chart.js, `react-chartjs-2`, and Recharts for charts
- Google Generative AI SDK for browser-side Gemini features
- jsPDF and jsPDF AutoTable for document generation

### Backend

- Node.js ES modules (use Node 18.12+ or Node 20.9+; the `canvas` dependency declares this requirement)
- Express 4 and Mongoose 8
- MongoDB for users, departments, certificates, vouchers, and analysis history
- JSON Web Tokens and bcryptjs for authentication/password handling
- Multer for in-memory image and proof uploads
- Cloudinary for uploaded media
- Google Generative AI for server-side product image analysis

## Repository layout

```text
.
├── frontend/
│   ├── src/                 # React pages, layouts, components, and API clients
│   ├── package.json
│   ├── vite.config.js
│   └── vercel.json          # SPA route rewrite
└── backend/
    ├── config/              # Database helper (not used by current server entry point)
    ├── controllers/
    ├── middleware/
    ├── models/
    ├── routes/
    ├── package.json
    └── server.js            # Express entry point
```

There is no root `package.json`; install and run each application from its own directory.

## Prerequisites

- Node.js **18.12+** or **20.9+** and npm.
- A MongoDB database and connection URI.
- For the relevant integrations: a Google Gemini API key and Cloudinary credentials.

## Install and run locally

### 1. Configure the backend

Create `backend/.env`:

```dotenv
PORT=5000
MONGODB_URI=mongodb+srv://<username>:<password>@<cluster>/<database>
JWT_SECRET=replace-with-a-long-random-secret
API_KEY=your-google-generative-ai-key
CLOUDINARY_CLOUD_NAME=your-cloud-name
CLOUDINARY_API_KEY=your-cloudinary-api-key
CLOUDINARY_API_SECRET=your-cloudinary-api-secret
```

`MONGODB_URI` and `JWT_SECRET` are needed for the core database and authentication flows. `API_KEY` is used for backend Gemini image analysis. The three Cloudinary variables are needed for image and certificate-proof uploads. If an integration is not configured, its related feature will not work.

Install dependencies and start the API:

```bash
cd backend
npm ci
npm run dev
```

The development command uses Nodemon. To run without automatic restarts, use `npm start`. The API listens on `http://localhost:5000` unless `PORT` is set.

### 2. Configure the frontend

Create `frontend/.env.local`:

```dotenv
VITE_API_URL=http://localhost:5000/api
VITE_API_BASE_URL=http://localhost:5000/api
VITE_GEMINI_API_KEY=your-google-generative-ai-key
```

Both API URL variables are currently used by different parts of the app, so set them to the same API base URL, including `/api`. The Gemini key is used by the browser-side Monitoring and Analysis pages.

Install dependencies and start Vite in another terminal:

```bash
cd frontend
npm ci
npm run start
```

Open the URL Vite prints (usually `http://localhost:5173`). To make a production build locally:

```bash
npm run build
npm run preview -- --host 0.0.0.0
```

The build is written to `frontend/dist`; preview usually listens on port `4173`.

## Environment variables

Set frontend `VITE_` variables at build time. Vite embeds them in the generated JavaScript, where users can inspect them. Never put private server credentials in a frontend variable.

| Variable | Application | Required for / purpose |
| --- | --- | --- |
| `MONGODB_URI` | Backend | Required for the active server's MongoDB connection. |
| `JWT_SECRET` | Backend | Required for signing and verifying authentication tokens. Use a strong, private value. |
| `PORT` | Backend | Optional. Defaults to `5000`; set to the port provided by the deployment platform. |
| `API_KEY` | Backend | Google Generative AI key used in backend product-image analysis. |
| `CLOUDINARY_CLOUD_NAME` | Backend | Cloudinary account name used for uploads. |
| `CLOUDINARY_API_KEY` | Backend | Cloudinary API key used for uploads. |
| `CLOUDINARY_API_SECRET` | Backend | Private Cloudinary API secret used for uploads. |
| `VITE_API_URL` | Frontend | API base for auth, dashboard, certificate, and admin requests. Include `/api`. |
| `VITE_API_BASE_URL` | Frontend | API base for profile and organization requests. Include `/api`. |
| `VITE_GEMINI_API_KEY` | Frontend | Gemini key for direct browser-side calls in Monitoring and Analysis. This key is public in a client build. |
| `VITE_BACKEND_URL` | Vite config | Optional development proxy target in `vite.config.js`. Current page code uses the explicit API URLs above. The Vite config reads this from `process.env`; setting it only in a Vite `.env` file may not populate that config value. |

The repository also contains `backend/config/db.js`, which reads `MONGO_URI`, but the active `backend/server.js` does not import that helper and instead connects with `MONGODB_URI`. Set `MONGODB_URI` for the server as currently written.

`GCP_PROJECT_ID` and `GOOGLE_APPLICATION_CREDENTIALS` are read by a Vertex AI helper in the backend, but that helper currently has no call sites. They are not required by the active routes. A Supabase client file refers to `VITE_SUPABASE_URL` and `VITE_SUPABASE_ANON_KEY`, but it is not imported by the app and the Supabase package is not listed as a frontend dependency; these variables are not part of the current setup.

## Build and deploy

Deploy the frontend and backend as separate services.

### Frontend on Vercel or another static host

The repository includes `frontend/vercel.json`, which rewrites all paths to `/index.html` so React Router routes continue to work on refresh. Configure the frontend project with:

- **Root directory:** `frontend`
- **Install command:** `npm ci`
- **Build command:** `npm run build`
- **Output directory:** `dist`
- **Build environment:** set `VITE_API_URL`, `VITE_API_BASE_URL`, and `VITE_GEMINI_API_KEY`

Use the deployed backend URL for both API base variables, for example `https://api.example.com/api`. Frontend environment variables are compiled into the static build, so redeploy after changing them.

### Backend on a Node.js host

Configure a separate Node.js service with:

- **Root directory:** `backend`
- **Install command:** `npm ci`
- **Build command:** none (there is no backend build script)
- **Start command:** `npm start`
- **Environment:** `MONGODB_URI`, `JWT_SECRET`, `PORT`, and any integration credentials required by enabled features

The host must provide outbound access to MongoDB and to configured Google/Cloudinary services. The API currently uses `cors()` with its defaults: it allows requests from any origin and does not enable credentialed CORS requests. There is no CORS-origin environment variable in the current implementation.

## API overview

All routes are mounted under `/api`:

| Base path | Purpose |
| --- | --- |
| `/api/auth` | Registration, login, and authenticated profile lookup. |
| `/api/analysis` | Authenticated product image analysis and analysis history. Image uploads are limited to 10 MB and image MIME types. |
| `/api/certificates` | Certificate listing, proof upload, goal verification, and certificate download. |
| `/api` | User data, user comparison data, and department sample data. |
| `/api/vouchers` | List, offer, and claim vouchers. |
| `/api/departments` | List, create, delete, and export department data. |

Most routes require `Authorization: Bearer <token>`. The API has no separate health-check route defined in the current server.

## Troubleshooting

- **Browser calls go to localhost in production:** set both `VITE_API_URL` and `VITE_API_BASE_URL` to the deployed API origin plus `/api`, then rebuild/redeploy the frontend.
- **MongoDB connection errors:** check `MONGODB_URI`, database network access rules, and credentials. The active server logs a connection error but still starts listening, so a running process does not guarantee database connectivity.
- **Authentication fails or tokens are rejected:** ensure `JWT_SECRET` is set and remains consistent across backend restarts/deployments.
- **Gemini features fail:** check the relevant key: `VITE_GEMINI_API_KEY` for browser-side Monitoring/Analysis, or backend `API_KEY` for server-side image analysis.
- **Uploads fail:** check all three Cloudinary values and confirm the service is reachable.
- **Refreshing a nested frontend URL returns 404:** configure the static host to return `index.html` for application routes. The supplied Vercel rewrite handles this for Vercel deployments.

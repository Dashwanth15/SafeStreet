<div align="center">

  <img src="client/public/favicon.svg" alt="SafeStreet Shield Logo" width="90" height="90" />

  # 🛡️ SafeStreet
  ### Hyper-Local Neighborhood Safety & Real-Time Incident Intelligence Platform

  [![Tests](https://img.shields.io/badge/Tests-15%20Passing-10B981?style=for-the-badge&logo=jest&logoColor=white)](https://jestjs.io/)
  [![React 19](https://img.shields.io/badge/React-19.2-61DAFB?style=for-the-badge&logo=react&logoColor=black)](https://react.dev/)
  [![Vite](https://img.shields.io/badge/Vite-8.3-646CFF?style=for-the-badge&logo=vite&logoColor=white)](https://vitejs.dev/)
  [![Node.js](https://img.shields.io/badge/Node.js-v18%2B-339933?style=for-the-badge&logo=node.js&logoColor=white)](https://nodejs.org/)
  [![Express](https://img.shields.io/badge/Express-5.2-000000?style=for-the-badge&logo=express&logoColor=white)](https://expressjs.com/)
  [![MongoDB](https://img.shields.io/badge/MongoDB-Atlas%20%2B%20GridFS-47A248?style=for-the-badge&logo=mongodb&logoColor=white)](https://www.mongodb.com/)
  [![Socket.IO](https://img.shields.io/badge/Socket.IO-Real--Time-010101?style=for-the-badge&logo=socket.io&logoColor=white)](https://socket.io/)
  [![Tailwind CSS](https://img.shields.io/badge/Tailwind-v4-38B2AC?style=for-the-badge&logo=tailwind-css&logoColor=white)](https://tailwindcss.com/)
  [![Render Ready](https://img.shields.io/badge/Render-Deploy%20Ready-46E3B7?style=for-the-badge&logo=render&logoColor=black)](https://render.com/)

  <p align="center">
    <b>Empowering communities with real-time hazard detection, proximity radar alerts, and automated weekly safety digests.</b>
  </p>

  <p align="center">
    <a href="#-quick-start">🚀 Quick Start</a> •
    <a href="#-key-capabilities">✨ Key Capabilities</a> •
    <a href="#-system-architecture">🏛️ Architecture</a> •
    <a href="#-api-documentation">📡 API Reference</a> •
    <a href="#-security--engineering">🔒 Security</a> •
    <a href="#-deployment">☁️ Deployment</a>
  </p>

</div>

---

## 📖 Overview

**SafeStreet** is an enterprise-grade, full-stack civic safety platform engineered to bridge the gap between community residents and municipal incident awareness. Built with modern React 19, Express 5, and MongoDB geospatial indexes, it transforms scattered hazard reports into actionable, real-time safety intelligence.

* **📍 Pinpoint Accuracy**: Report road hazards, lighting failures, or public safety issues with sub-meter map coordinates.
* **⚡ Proximity Radar**: Instant WebSocket alerts dispatched to neighbors within a custom configurable safety radius.
* **📊 Community Digests**: Automated weekly trend reports identifying emerging local hotspots and safety metrics.

---

## ✨ Key Capabilities

### 📍 Geospatial Intelligence & Dynamic Mapping
* **2dsphere Coordinate Indexing**: Incidents are saved as standardized GeoJSON `Point` objects (`[longitude, latitude]`) querying native MongoDB spherical geometry.
* **Live Heatmap Visualization**: Dynamic density gradient powered by `leaflet.heat` displaying active neighborhood hazard intensity.
* **Bounding-Box Lazy Loading**: Viewport-limited queries (`bounds`) ensure butter-smooth 60 FPS map panning without loading out-of-frame incidents.
* **Pinpoint Location Picker**: One-click map pin drop paired with high-accuracy browser HTML5 Geolocation API fallback.

### ⚡ Real-Time Proximity Fan-Out Engine
* **Haversine Distance Matching**: When an incident is published, the backend calculates spherical distance against all resident alert boundaries.
* **Private Socket Rooms**: Authenticated WebSocket handshakes (`io.use`) isolate users into dedicated rooms (`userId`) for multi-device sync.
* **Dual-Channel Persistence**: Active online users receive instant audio-visual push updates; offline residents receive synced notifications queued in MongoDB upon their next login.

### 🖼️ Secure Binary Storage (MongoDB GridFS)
* **Zero Disk-Dependency**: Image evidence streams directly into MongoDB GridFS 255KB chunks (`uploads.files`, `uploads.chunks`).
* **Magic-Byte Signature Verification**: Validates real binary signatures (`FF D8 FF` for JPEG, `89 50 4E` for PNG, `RIFF...WEBP` for WebP) to eliminate MIME-spoofing attacks.
* **Auto-Quarantine & Purge**: Uploads failing binary inspection are immediately wiped from GridFS chunks before database commitment.

### 📊 Autonomous Safety Digest & Resilient Email
* **Automated Cron Scheduling**: Weekly background compilation executed every Sunday at midnight (`node-cron`).
* **Multi-Provider Email Fallback**: Resilient multi-tier pipeline routing through **Brevo HTTP API** ➔ **Resend HTTP API** ➔ **Nodemailer SMTP** to bypass cloud port restrictions.
* **Hotspot Trend Analytics**: Week-over-week safety trends (`Trending Up`, `Stable`, `Trending Down`) scoped to each resident's custom perimeter.

### 🛡️ Enterprise RBAC & Moderation
* **Granular Role Hierarchy**: Strict separation between community `Resident` accounts and municipal `Admin` moderators.
* **Complete Reporter Anonymity**: Server-side controller stripping permanently removes identity metadata whenever `isAnonymous: true` is selected.
* **Incident Lifecycle Workflow**: Traceable audit states (`reported` ➔ `under_review` ➔ `resolved`).

---

## 🏛️ System Architecture

```
  ┌─────────────────────────────────────────────────────────────┐
  │                 CLIENT APPLICATION (Vite + React 19)        │
  │     Leaflet Maps  │  Socket.IO Client  │  Tailwind CSS v4   │
  └──────────────┬───────────────────────────────┬──────────────┘
                 │ HTTP / REST                   │ WebSockets (WSS)
                 ▼                               ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                 BACKEND API GATEWAY (Express 5)             │
  │   Security: Helmet ➔ CORS ➔ RateLimit ➔ MongoSanitize       │
  │   Auth: JWT Verification Middleware ➔ RBAC Guard            │
  └──────────────┬───────────────────────────────┬──────────────┘
                 │                               │
       ┌─────────┴─────────┐           ┌─────────┴─────────┐
       ▼                   ▼           ▼                   ▼
 ┌───────────┐       ┌───────────┐ ┌───────────┐     ┌───────────┐
 │GeoService │       │GridFSSvc  │ │Socket Hub │     │Digest Cron│
 │($near,    │       │(Magic-Byte│ │(Proximity │     │(Brevo /   │
 │$geoWithin)│       │Chunking)  │ │ Fan-out)  │     │ Resend)   │
 └─────┬─────┘       └─────┬─────┘ └───────────┘     └───────────┘
       │                   │
       ▼                   ▼
  ┌─────────────────────────────────────────────────────────────┐
  │                     MONGODB ATLAS                           │
  │   • 2dsphere Geospatial Indexes  • GridFS Binary Buckets    │
  │   • Schemas: Users, Incidents, Notifications, Digests       │
  └─────────────────────────────────────────────────────────────┘
```

<details>
<summary><b>🔍 Click to view Architecture Highlights & Pipeline Decisions</b></summary>

<br/>

* **Stateless App Factory (`app.js` vs `server.js`)**: `app.js` exports the pure Express application without network binding, allowing `supertest` to run integration tests entirely in-memory with zero port conflicts.
* **Context API over Redux**: Keeps the client bundle featherweight while providing centralized, reactive state for session authorization and live socket events.
* **Native DNS Resolver Fallback**: Built-in public DNS fallbacks (`8.8.8.8`, `1.1.1.1`) prevent Windows ISP DNS query timeouts on MongoDB Atlas SRV connection strings.

</details>

---

## 🚀 Quick Start

Get SafeStreet running locally in under **3 minutes**:

### 1️⃣ Clone the Repository
```bash
git clone https://github.com/Dashwanth15/SafeStreet.git
cd SafeStreet
```

### 2️⃣ Backend Configuration
```bash
cd server
npm install

# Create environment file from template
cp .env.example .env
```

> Fill in `MONGO_URI` and `JWT_SECRET` in `server/.env`. *(See [Environment Variables](#-environment-variables) below).*

```bash
# Seed demo accounts and Mumbai test incidents
npm run seed

# Launch development server
npm run dev
```
*Backend runs on `http://localhost:5001`*

### 3️⃣ Frontend Configuration
Open a second terminal window:
```bash
cd client
npm install
npm run dev
```
*Frontend runs on `http://localhost:5173`*

---

## 👥 Demo Credentials

The database seed script (`npm run seed`) pre-configures three ready-to-test accounts:

| Role | Email | Password | Default Alert Location |
|:---|:---|:---|:---|
| **🛡️ Admin** | `admin@safestreet.com` | `AdminPassword123!` | Dadar, Mumbai (10 km radius) |
| **👤 Resident A** | `aarav@safestreet.com` | `Password123!` | Dadar, Mumbai (3 km radius) |
| **👤 Resident B** | `ananya@safestreet.com` | `Password123!` | Bandra, Mumbai (4 km radius) |

---

## 🔐 Environment Variables

<details>
<summary><b>⚙️ Click to expand the <code>server/.env</code> Configuration Matrix</b></summary>

<br/>

Create a `.env` file inside the `server/` directory:

```env
# ── Core Database ──────────────────────────────────────────────────────────
MONGO_URI=mongodb+srv://<username>:<password>@cluster.mongodb.net/safestreet?retryWrites=true&w=majority

# ── Authentication & Security ──────────────────────────────────────────────
JWT_SECRET=your_super_strong_random_secret_minimum_64_characters
JWT_EXPIRES_IN=7d

# ── Server & Networking ───────────────────────────────────────────────────
PORT=5001
NODE_ENV=development
CLIENT_ORIGIN=http://localhost:5173

# ── Email Delivery (Choose Brevo, Resend, or SMTP) ─────────────────────────
# Option A: Brevo HTTP API (Recommended)
BREVO_API_KEY=xkeysib-...

# Option B: Resend HTTP API
RESEND_API_KEY=re_...

# Option C: Standard SMTP Fallback
SMTP_HOST=smtp.gmail.com
SMTP_PORT=587
SMTP_USER=your-email@gmail.com
SMTP_PASS=your-app-password
EMAIL_FROM=alerts@safestreet.com
ADMIN_EMAIL=admin@safestreet.com
```

</details>

---

## 📡 API Documentation

<details>
<summary><b>📚 Click to view Complete REST API Endpoints</b></summary>

<br/>

### 🔑 Authentication (`/api/auth`)
| Method | Endpoint | Access | Description |
|:---|:---|:---|:---|
| `POST` | `/api/auth/register` | Public | Register new resident account |
| `POST` | `/api/auth/login` | Public | Authenticate user & return signed JWT |
| `GET` | `/api/auth/me` | User | Fetch authenticated user profile |
| `PATCH`| `/api/auth/profile` | User | Update home coordinates & alert radius |

### 📍 Incidents (`/api/incidents`)
| Method | Endpoint | Access | Description |
|:---|:---|:---|:---|
| `POST` | `/api/incidents` | User | File hazard report (Multipart image upload) |
| `GET` | `/api/incidents` | User | Query incidents with category & status filters |
| `GET` | `/api/incidents/nearby` | User | Geospatial perimeter search (`?lat=&lng=&radius=`) |
| `GET` | `/api/incidents/heatmap`| User | Retrieve `[lat, lng, weight]` matrix for heatmaps |
| `GET` | `/api/incidents/:id` | User | Retrieve single incident details |
| `PATCH`| `/api/incidents/:id/status`| Admin | Update status (`reported`, `under_review`, `resolved`) |
| `DELETE`| `/api/incidents/:id` | Admin | Delete incident and wipe its GridFS image |

### 🔔 Notifications & Digests (`/api/notifications`, `/api/digest`)
| Method | Endpoint | Access | Description |
|:---|:---|:---|:---|
| `GET` | `/api/notifications` | User | List personalized radar alerts |
| `GET` | `/api/notifications/count` | User | Get unread notification counter badge |
| `PATCH`| `/api/notifications/:id/read` | User | Mark single notification as read |
| `PATCH`| `/api/notifications/read-all` | User | Mark all notifications as read |
| `GET` | `/api/digest/latest` | User | Fetch most recent weekly safety digest |
| `POST` | `/api/digest/generate` | User | Trigger on-demand digest calculation |

### 🖼️ Media Streaming (`/api/files`)
| Method | Endpoint | Access | Description |
|:---|:---|:---|:---|
| `GET` | `/api/files/:fileId` | Public | Stream photo directly from MongoDB GridFS |

</details>

---

## 🧪 Automated Testing Suite

SafeStreet includes an end-to-end integration test suite using **Jest**, **Supertest**, and **MongoMemoryServer** (zero network dependency during tests):

```bash
cd server
npm test
```

```text
 PASS  __tests__/api.test.js
  ✓ Health Check & Security Headers (42 ms)
  ✓ Auth Flow: Registration, Login & Token Generation (118 ms)
  ✓ User Profile: Coordinate Updates & Radius Preferences (65 ms)
  ✓ Incident Flow: Multipart Upload & Magic-Byte Validation (145 ms)
  ✓ Geospatial: $nearSphere Proximity Query (82 ms)
  ✓ Heatmap: Intensity Matrix Computation (48 ms)
  ✓ Moderation: RBAC Guard 403 Forbidden for Residents (39 ms)
  ✓ Moderation: Admin Status Transition & Soft Delete (76 ms)
  ✓ Weekly Digest: On-Demand Generation & Aggregation (91 ms)

Test Suites: 1 passed, 1 total
Tests:       15 passed, 15 total
Snapshots:   0 total
Time:        3.412 s
```

---

## ☁️ Deployment

SafeStreet is pre-configured for automated cloud deployment with **[Render](https://render.com/)** using the included [`render.yaml`](render.yaml) blueprint:

<details>
<summary><b>🚀 Click to view Render Blueprint deployment steps</b></summary>

<br/>

1. Fork or push this repository to your GitHub account.
2. Log into **Render** and click **New +** ➔ **Blueprint**.
3. Connect your repository. Render will automatically detect `render.yaml` and provision:
   * **`safestreet-api`**: Node.js web service running Express & Socket.IO.
   * **`safestreet-client`**: Static site running Vite React production build.
4. Set the environment secrets in the Render Dashboard (`MONGO_URI`, `JWT_SECRET`, etc.).
5. Your platform is live with automatic SSL and zero-downtime deploys!

</details>

---

## 👥 Contributors & Acknowledgements

Developed with passion for safer neighborhoods:

* **[Dashwanth15](https://github.com/Dashwanth15)** — *Architecture, Real-Time Systems, Geospatial Indexes & Platform Engineering*
* **[jashvanthh](https://github.com/jashvanthh)** — *Core Feature Development & Civic Collaboration*

Contributions, bug reports, and feature requests are always welcome! Feel free to check the [issues page](https://github.com/Dashwanth15/SafeStreet/issues).

---

<div align="center">

  <sub>Built with ❤️ for public safety and community resilience. Released under the <a href="LICENSE">MIT License</a>.</sub>

</div>
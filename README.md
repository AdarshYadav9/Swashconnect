# SwasthyaConnect — Rural TeleHealth Access System

A modern full-stack telehealth platform built to provide remote healthcare access to rural communities. Patients can consult doctors via live video consultations, check medicine availability across pharmacies, manage digital health records, and receive AI-powered symptom assessments.

---

## Features

- **Live Video Consultations**: Powered by Jitsi Meet SDK with HD video, screen sharing, and in-call chat.
- **Role-Based Authentication**: Secure Patient and Doctor authentication via Firebase.
- **Appointment Management**: Streamlined scheduling, rescheduling, and status tracking.
- **Digital Health Records**: Secure storage and retrieval of prescriptions, vitals, and diagnoses.
- **Pharmacy & Medicine Search**: Real-time stock lookup and pricing across rural pharmacies.
- **AI Symptom Checker**: Intelligent health analysis and specialist recommendations using Google Gemini (`gemini-2.5-flash`).
- **Progressive Web App (PWA)**: Offline resilience and installable home-screen experience for low-connectivity environments.

---

## Tech Stack

| Layer | Technologies |
|---|---|
| **Frontend** | React 18, Vite, Tailwind CSS, React Router v6, Framer Motion, Lucide Icons |
| **Video Conferencing** | Jitsi Meet SDK (`@jitsi/react-sdk`) |
| **Authentication & Database** | Firebase Auth, Cloud Firestore (Firebase Admin SDK) |
| **Backend API** | Node.js, Express, Helmet, CORS, Rate Limiting, Winston Logger |
| **AI Intelligence** | Google Gemini API (`@google/generative-ai`) |
| **Deployment Target** | **Vercel** (Frontend SPA) & **Render** (Backend API) |

---

## Project Structure

```
SwasthyaConnect/
├── README.md                 # Main project overview & documentation
├── .gitignore                # Root gitignore excluding secrets & build outputs
├── frontend/                 # React + Vite application (Vercel target)
│   ├── index.html            # Single page entry point
│   ├── vite.config.js        # Vite bundler & PWA configuration
│   ├── tailwind.config.js    # Tailwind CSS styling configuration
│   ├── postcss.config.js     # PostCSS configuration
│   ├── vercel.json           # Vercel SPA routing & security headers
│   ├── package.json          # Frontend dependencies & scripts
│   ├── .env.example          # Frontend environment variable template
│   ├── public/               # Static assets, web manifest, service worker
│   └── src/
│       ├── App.jsx           # Root application router & layout
│       ├── main.jsx          # React DOM render entry
│       ├── index.css         # Global Tailwind stylesheets
│       ├── components/       # Reusable UI components & navigation
│       ├── pages/            # Views (Dashboard, Consultation, Records, etc.)
│       ├── services/         # Axios API client & endpoints
│       ├── firebase/         # Firebase client SDK initialization
│       └── lib/              # Utility functions & helpers
├── backend/                  # Node.js + Express API (Render target)
│   ├── index.js              # Express server entry point
│   ├── package.json          # Backend dependencies & scripts
│   ├── .env.example          # Backend environment variable template
│   ├── config/               # Firebase Admin SDK & logging setup
│   ├── controllers/          # Business logic handlers
│   ├── middleware/           # Auth, CORS, validation, rate limiting
│   └── routes/               # API route declarations
└── docs/                     # Detailed guides
    ├── DEPLOYMENT.md         # Step-by-step Vercel & Render deployment guide
    └── TESTING.md            # Local setup & comprehensive testing guide
```

---

## Getting Started

### Prerequisites

- **Node.js**: v18.0.0 or higher
- **npm**: v9.0.0 or higher
- **Firebase Project**: (Authentication & Cloud Firestore enabled)
- **Google Gemini API Key**: (for AI Symptom Checker)

---

### Local Installation & Setup

#### 1. Clone the repository
```bash
git clone <your-repository-url>
cd SwasthyaConnect
```

#### 2. Backend Setup
```bash
cd backend
npm install
cp .env.example .env
```
Edit `backend/.env` with your Firebase and Gemini credentials:
```env
PORT=5000
NODE_ENV=development
CORS_ORIGIN=http://localhost:5173
FIREBASE_PROJECT_ID=your-project-id
FIREBASE_CLIENT_EMAIL=your-service-account@your-project.iam.gserviceaccount.com
FIREBASE_PRIVATE_KEY="-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"
JWT_SECRET=your-secure-jwt-secret
GEMINI_API_KEY=your-gemini-api-key
GEMINI_MODEL=gemini-2.5-flash
```
Start the backend development server:
```bash
npm run dev
```
Backend API will be running at `http://localhost:5000`.

#### 3. Frontend Setup
In a new terminal window:
```bash
cd frontend
npm install
cp .env.example .env
```
Edit `frontend/.env`:
```env
VITE_API_BASE_URL=http://localhost:5000/api
VITE_FIREBASE_API_KEY=your-firebase-api-key
VITE_FIREBASE_AUTH_DOMAIN=your-project.firebaseapp.com
VITE_FIREBASE_PROJECT_ID=your-project-id
VITE_FIREBASE_STORAGE_BUCKET=your-project.appspot.com
VITE_FIREBASE_MESSAGING_SENDER_ID=000000000000
VITE_FIREBASE_APP_ID=1:000000000000:web:xxxxxxxxxxxx
VITE_JITSI_DOMAIN=meet.jit.si
```
Start the frontend development server:
```bash
npm run dev
```
Open `http://localhost:5173` in your browser.

---

## Production Deployment

SwasthyaConnect is configured for deployment with **Vercel** for the frontend and **Render** for the backend API:

- **Frontend (Vercel)**:
  - **Root Directory**: `frontend`
  - **Build Command**: `npm run build:prod`
  - **Output Directory**: `dist`
- **Backend (Render)**:
  - **Root Directory**: `backend`
  - **Build Command**: `npm install`
  - **Start Command**: `npm start`

For detailed step-by-step instructions, environment variable configurations, and Firebase setup, refer to [docs/DEPLOYMENT.md](docs/DEPLOYMENT.md).

---

## Testing & Verification

For local testing procedures, video call debugging steps, and API endpoint verification, refer to [docs/TESTING.md](docs/TESTING.md).

```bash
# Build test frontend
cd frontend
npm run build:prod

# Lint frontend
npm run lint
```

---

## API Reference

| Method | Endpoint | Description |
|---|---|---|
| `GET` | `/api/health` | Service health status check |
| `GET` | `/api/users` | List registered users |
| `GET` | `/api/users/doctors` | List verified doctors |
| `POST` | `/api/users` | Create/sync user profile |
| `GET` | `/api/appointments` | Retrieve appointments list |
| `POST` | `/api/appointments` | Book new appointment |
| `PUT` | `/api/appointments/:id` | Update appointment details |
| `DELETE` | `/api/appointments/:id/cancel` | Cancel existing appointment |
| `GET` | `/api/records` | Retrieve digital health records |
| `POST` | `/api/records` | Create new health record entry |
| `GET` | `/api/medicines` | List medicines catalog |
| `GET` | `/api/medicines/search?query=` | Search medicines across pharmacies |
| `POST` | `/api/symptom-check` | AI symptom analysis via Gemini |
| `GET` | `/api/health-tips` | Fetch health recommendations |

---

## License

This project is licensed under the MIT License.

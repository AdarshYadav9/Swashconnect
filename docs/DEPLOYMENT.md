# Deployment Guide — SwasthyaConnect

This guide provides end-to-end instructions for deploying **SwasthyaConnect** to production with **Vercel** (Frontend) and **Render** (Backend API).

---

## Architecture Overview

```
┌─────────────────────────────────────────────────────────────┐
│                      Client Browser                         │
└──────────────┬───────────────────────────────┬──────────────┘
               │ HTTPS                         │ HTTPS / WSS
               ▼                               ▼
┌──────────────────────────────┐ ┌────────────────────────────┐
│      Frontend (Vercel)       │ │     Backend API (Render)   │
│   React 18 + Vite (SPA)      │ │     Node.js + Express      │
│   Root Directory: frontend   │ │     Root Directory: backend│
└──────────────┬───────────────┘ └─────────────┬──────────────┘
               │                               │
               ├───────────────────────────────┤
               ▼                               ▼
┌──────────────────────────────┐ ┌────────────────────────────┐
│     Firebase Auth & DB       │ │   Google Gemini 2.5 Flash  │
│  Client SDK & Admin SDK      │ │   AI Symptom Checker & Tips│
└──────────────────────────────┘ └────────────────────────────┘
```

---

## 1. Backend Deployment (Render)

### Step 1.1: Create a Web Service on Render
1. Log in to [Render Dashboard](https://dashboard.render.com/).
2. Click **New +** → **Web Service**.
3. Connect your GitHub repository (`SwasthyaConnect`).
4. Configure service settings:
   - **Name**: `swasthya-connect-api` (or preferred name)
   - **Region**: Select the region closest to your users (e.g., Singapore, Frankfurt, Oregon)
   - **Branch**: `main`
   - **Root Directory**: `backend`
   - **Runtime**: `Node`
   - **Build Command**: `npm install`
   - **Start Command**: `npm start`
   - **Plan Type**: `Free` or `Starter`

### Step 1.2: Configure Backend Environment Variables
In the Render Web Service settings, navigate to **Environment** and add the following variables:

| Variable Name | Example Value | Description |
|---|---|---|
| `NODE_ENV` | `production` | Production environment mode |
| `PORT` | `10000` | Render default port (injected automatically) |
| `CORS_ORIGIN` | `https://your-frontend.vercel.app` | Allowed frontend URL (comma-separated if multiple) |
| `FIREBASE_PROJECT_ID` | `your-firebase-project-id` | Firebase project ID |
| `FIREBASE_CLIENT_EMAIL` | `firebase-adminsdk-xxx@your-project.iam.gserviceaccount.com` | Firebase service account client email |
| `FIREBASE_PRIVATE_KEY` | `"-----BEGIN PRIVATE KEY-----\n...\n-----END PRIVATE KEY-----\n"` | Service account private key (include quotes & escaped newlines) |
| `JWT_SECRET` | `generate-a-strong-random-secret-key` | Random 32+ character string |
| `GEMINI_API_KEY` | `AIzaSy...` | Google AI Studio API Key |
| `GEMINI_MODEL` | `gemini-2.5-flash` | Gemini model version |

> **Private Key Formatting Tip for Render**: If pasting the private key directly with actual line breaks, wrap it in double quotes or replace newlines with `\n`. The backend code automatically handles escaped `\n` characters.

### Step 1.3: Deploy and Verify Backend
1. Click **Create Web Service**.
2. Wait for the build to complete.
3. Test your Render backend URL:
   ```bash
   curl https://swasthya-connect-api.onrender.com/api/health
   ```
   Expected response:
   ```json
   {
     "status": "ok",
     "message": "Rural TeleHealth API is running",
     "timestamp": "..."
   }
   ```

---

## 2. Frontend Deployment (Vercel)

### Step 2.1: Import Project to Vercel
1. Log in to [Vercel](https://vercel.com/).
2. Click **Add New...** → **Project**.
3. Import your GitHub repository (`SwasthyaConnect`).

### Step 2.2: Configure Vercel Project Settings
In the configuration screen:
- **Framework Preset**: `Vite`
- **Root Directory**: Click *Edit* and select `frontend`
- **Build and Output Settings**:
  - **Build Command**: Override → `npm run build:prod`
  - **Output Directory**: Override → `dist`
  - **Install Command**: Override → `npm install`

### Step 2.3: Configure Frontend Environment Variables
Add the following environment variables in Vercel:

| Variable Name | Example Value | Description |
|---|---|---|
| `VITE_API_BASE_URL` | `https://swasthya-connect-api.onrender.com/api` | Render backend API URL (with `/api` suffix) |
| `VITE_FIREBASE_API_KEY` | `AIzaSy...` | Firebase Web API Key |
| `VITE_FIREBASE_AUTH_DOMAIN` | `your-project.firebaseapp.com` | Firebase Auth Domain |
| `VITE_FIREBASE_PROJECT_ID` | `your-project-id` | Firebase Project ID |
| `VITE_FIREBASE_STORAGE_BUCKET` | `your-project.appspot.com` | Firebase Storage Bucket |
| `VITE_FIREBASE_MESSAGING_SENDER_ID` | `123456789012` | Firebase Sender ID |
| `VITE_FIREBASE_APP_ID` | `1:123456789012:web:abcdef` | Firebase Web App ID |
| `VITE_JITSI_DOMAIN` | `meet.jit.si` | Jitsi Meet video domain |

### Step 2.4: Deploy Frontend
1. Click **Deploy**.
2. Vercel will run `npm run build:prod` and deploy the static assets to their global edge network.
3. Once deployed, note down your production URL (e.g., `https://swasthya-connect.vercel.app`).

### Step 2.5: Sync CORS on Render Backend
1. Go back to Render → your Web Service → **Environment**.
2. Update `CORS_ORIGIN` to include your new Vercel domain:
   ```env
   CORS_ORIGIN=https://swasthya-connect.vercel.app
   ```
3. Save changes; Render will redeploy with the updated CORS policy.

---

## 3. Firebase Project Configuration

### 3.1: Enable Authentication
1. Go to [Firebase Console](https://console.firebase.google.com/).
2. Select **Build** → **Authentication** → **Get Started**.
3. Under **Sign-in method**, enable **Email/Password**.
4. In **Settings** → **Authorized domains**, add your Vercel deployment domain (e.g., `swasthya-connect.vercel.app`).

### 3.2: Enable Cloud Firestore
1. Select **Build** → **Firestore Database** → **Create database**.
2. Choose a location closest to your users.
3. Start in **Production mode**.
4. Apply the following Firestore Security Rules:

```javascript
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    // Users collection: users can read/write their own profile
    match /users/{userId} {
      allow read, write: if request.auth != null && request.auth.uid == userId;
    }
    // Appointments: authenticated users can access appointments
    match /appointments/{appointmentId} {
      allow read, write: if request.auth != null;
    }
    // Health records: authenticated users can read; doctors can write
    match /health_records/{recordId} {
      allow read: if request.auth != null;
      allow write: if request.auth != null && request.auth.token.role == 'doctor';
    }
    // Medicines: public read, pharmacies can update stock
    match /medicines/{medicineId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.role == 'pharmacy';
    }
  }
}
```

### 3.3: Generate Firebase Service Account Key (for Backend)
1. Go to **Project settings** (gear icon) → **Service accounts**.
2. Under **Firebase Admin SDK**, select **Node.js**.
3. Click **Generate new private key** and confirm download.
4. Extract `project_id`, `client_email`, and `private_key` into your Render backend environment variables.

---

## 4. Google Gemini AI API Configuration

1. Visit [Google AI Studio](https://aistudio.google.com/).
2. Click **Get API key** → **Create API key**.
3. Copy the generated API key.
4. Add to Render backend environment variables as `GEMINI_API_KEY`.
5. Set `GEMINI_MODEL=gemini-2.5-flash`.

---

## 5. Deployment Verification Checklist

- [ ] Backend `/api/health` returns `{"status":"ok"}`.
- [ ] Frontend loads with all UI components and styling intact.
- [ ] User signup and login (Patient & Doctor) successfully creates/authenticates Firebase user.
- [ ] Appointment creation and retrieval functions through `/api/appointments`.
- [ ] AI Symptom Checker successfully queries `/api/symptom-check` with Gemini response.
- [ ] Video consultation initializes Jitsi Meet frame with camera and microphone permissions.
- [ ] PWA service worker registers successfully for offline capabilities.

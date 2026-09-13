# Testing Guide — SwasthyaConnect

This document details procedures for testing SwasthyaConnect locally and in staging environments.

---

## 1. Local Development Setup

### Prerequisites
- Node.js 18+ (tested on Node.js 18, 20, 22)
- npm 9+
- Firebase project credentials (or built-in demo mode)
- Google Gemini API key (optional for local mock testing)

### Installation & Starting Services

#### Terminal 1 — Backend Server
```bash
cd backend
npm install
npm run dev
```
- Server starts on: `http://localhost:5000` (or `PORT` from `.env`)
- Health Check: `http://localhost:5000/api/health`

#### Terminal 2 — Frontend Application
```bash
cd frontend
npm install
npm run dev
```
- Frontend starts on: `http://localhost:5173`

---

## 2. Automated & Build Testing

### Frontend Build Test (Production Mode)
```bash
cd frontend
npm run build:prod
```
Expected output:
- Vite generates optimized assets in `frontend/dist/`
- Zero syntax or bundling errors
- Generates PWA service worker and manifest

### Frontend Linting
```bash
cd frontend
npm run lint
```

---

## 3. Backend API Verification

You can verify all backend endpoints using `curl` or Postman:

### 3.1 Health Check
```bash
curl http://localhost:5000/api/health
```
**Response:**
```json
{
  "status": "ok",
  "message": "Rural TeleHealth API is running",
  "timestamp": "2026-09-14T..."
}
```

### 3.2 Health Tips Endpoint
```bash
curl http://localhost:5000/api/health-tips
```

### 3.3 Medicine Search Endpoint
```bash
curl "http://localhost:5000/api/medicines/search?query=paracetamol"
```

---

## 4. Video Consultation Testing (Jitsi Meet)

SwasthyaConnect uses Jitsi Meet for peer-to-peer and group telehealth video sessions. Follow these steps to verify video conferencing between two users:

### Step 4.1: Dedicated Video Test Page
1. Open `http://localhost:5173/simple-test` in **Browser Window 1** (e.g. Chrome).
2. Enter your name: `"Patient John"`.
3. Click **Copy Room ID** to copy the generated session key.
4. Click **Join Room** and allow browser camera & microphone access.
5. In **Browser Window 2** (Incognito mode or a different browser like Firefox):
   - Navigate to `http://localhost:5173/simple-test`.
   - Enter name: `"Dr. Sharma"`.
   - Paste the EXACT same Room ID.
   - Click **Join Room** and allow camera/mic access.
6. **Expected Outcome**:
   - Both windows show live video feeds of each participant.
   - Window 1 indicates participant count updated.
   - Audio and video controls (mute/unmute, camera toggle, screen sharing) respond instantly.

### Step 4.2: Video Debugging Checklist
If video does not connect:
1. **Camera/Microphone Blocked**: Click the lock/camera icon in the browser address bar → select "Always Allow".
2. **Room ID Mismatch**: Verify both browser sessions are connected to the exact same room ID string.
3. **Firewall / VPN**: Jitsi Meet requires UDP/WSS outbound access to `meet.jit.si`. Disable strict VPNs if connection times out.
4. **Debug Route**: Visit `http://localhost:5173/debug-video/test123` for real-time diagnostic checks on WebRTC capabilities and media device enumeration.

---

## 5. End-to-End Feature Verification

| Feature | Test Procedure | Expected Result |
|---|---|---|
| **Patient Registration** | Go to `/patient-signup`, fill details, submit | Account created, redirected to Patient Dashboard |
| **Doctor Registration** | Go to `/doctor-signup`, fill details, submit | Account created, redirected to Doctor Dashboard |
| **Login Selection** | Go to `/login`, select role, enter credentials | Authenticated and redirected based on user role |
| **Appointment Booking** | Navigate to `/book-appointment`, select doctor, date & time, submit | Appointment saved and displayed in dashboard list |
| **AI Symptom Checker** | Navigate to `/symptom-checker`, enter symptoms (e.g., "fever and headache"), submit | Structured triage advice and specialist recommendation rendered |
| **Medicine Search** | Navigate to `/medicines`, search for common medicine name | List of pharmacies with stock availability and price |
| **Health Records** | Navigate to `/health-records`, view or upload prescriptions/vitals | Medical history cards rendered with diagnostic notes |
| **PWA Offline Mode** | In DevTools → Application → Service Workers, toggle Offline, refresh | App shell loads from cache with offline indicator |

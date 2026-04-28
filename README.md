# Lifeline - Hyperlocal Emergency Response MVP

Lifeline is a rapid MVP that connects people in emergencies with nearby volunteers using smart, rule-based prioritization.

## Tech Stack (chosen for fast setup)

- Mobile app: Expo (React Native)
- Backend API: Node.js + Express
- Realtime alerts: Socket.IO
- Storage: in-memory (MVP, zero setup)
- Location math: simple Haversine distance calculation

This stack avoids heavy cloud setup so you can run everything locally quickly.

## Project Structure

```text
lifeline/
  README.md
  backend/
    package.json
    .env.example
    src/
      config/
        env.js
      data/
        store.js
      routes/
        volunteers.js
        emergencies.js
      services/
        matching.js
      utils/
        distance.js
      socket.js
      server.js
  mobile/
    package.json
    app.json
    babel.config.js
    .env.example
    App.js
    src/
      config/
        api.js
      screens/
        HomeScreen.js
        RequestHelpScreen.js
        VolunteerRegisterScreen.js
        VolunteerDashboardScreen.js
      services/
        socket.js
```

## Core Features Implemented

- Big red SOS trigger with emergency type selector
- Auto-location for users and volunteers
- Volunteer registration with skills and availability
- Smart alerting that scores volunteers by:
  - Distance to emergency
  - Skill relevance
  - Availability
- Priority logic:
  - For `heart_issue`, nearby `doctor`/`first_aid` volunteers are heavily prioritized
- Realtime alert feed for available volunteers

## Quick Start (Run Locally)

### Prerequisites

- Node.js 18+ and npm
- Expo Go app on Android/iOS (for phone testing)
- Both laptop and phone on the same Wi-Fi network (for LAN mode)

### 1) Start Backend API

1. Open terminal 1:
   - `cd lifeline/backend`
2. Install dependencies:
   - `npm install`
3. Create env file:
   - Copy `.env.example` to `.env`
4. Start backend:
   - `npm run dev`

Backend runs on `http://localhost:4000` by default.

### 2) Start Mobile App (Expo)

1. Open terminal 2:
   - `cd lifeline/mobile`
2. Install dependencies:
   - `npm install`
3. Create env file:
   - Copy `.env.example` to `.env`
4. Start Expo in LAN mode:
   - `npx expo start --host lan -c`

### 3) Open the App

- Android phone (Expo Go): scan QR shown in terminal
- Android emulator: press `a`
- Web browser: press `w` (or run `npm run web`)
- iOS simulator (macOS only): press `i`

If Expo shows `127.0.0.1`, restart with:
- `npx expo start --host lan -c`

## Environment Variables

### Backend (`backend/.env`)

```env
PORT=4000
CLIENT_ORIGIN=*
```

### Mobile (`mobile/.env`)

```env
EXPO_PUBLIC_API_BASE_URL=http://localhost:4000
```

For physical devices, replace `localhost` with your computer LAN IP:

```env
EXPO_PUBLIC_API_BASE_URL=http://192.168.x.x:4000
```

To find LAN IP on Windows:
- `ipconfig` -> use Wi-Fi adapter IPv4 address.

Important:
- Do not use VMware adapter IPs.
- Restart Expo after changing `.env`.

## Run On Web?

Yes, you can run this project on web.

- Command: `npm run web` (inside `mobile`)
- This opens the Expo web build in your browser.

Notes:
- Core screens and API calls work on web for demo/testing.
- Device-specific behavior (native permissions, some sensor/platform APIs) may behave differently than Android.

## API Endpoints

- `GET /health` - health check
- `POST /api/volunteers/register` - register/update volunteer profile
- `POST /api/volunteers/availability` - toggle volunteer availability
- `GET /api/volunteers` - list volunteers
- `POST /api/emergencies` - create emergency + trigger smart matching
- `GET /api/emergencies` - list emergencies

## Example Request Payloads

### Register volunteer

```json
{
  "name": "Asha",
  "phone": "+911111111111",
  "skills": ["first_aid", "nurse"],
  "available": true,
  "location": { "lat": 28.6139, "lng": 77.2090 }
}
```

### Trigger SOS

```json
{
  "reporterName": "Rahul",
  "type": "heart_issue",
  "notes": "Chest pain and dizziness",
  "location": { "lat": 28.6125, "lng": 77.2102 }
}
```

## Matching Logic (Rule-Based)

Each volunteer gets a score:

- Distance score: up to 60 points (closer = higher)
- Skill score: up to 40 points based on emergency type
- Heart issue boost: +50 points for `doctor` or `first_aid`
- Availability required (`available: true`)

Top scored volunteers are returned and pushed live via Socket.IO.

## MVP Notes

- This MVP uses in-memory storage; data resets when server restarts.
- Add database (Postgres/Supabase) later for persistence.
- Add push notifications (Expo Notifications / FCM) for production.
- Realtime alerts currently work in-app (Socket.IO), not as SMS.

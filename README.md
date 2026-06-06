# Women Safety App (AngelWatch)

A React + Capacitor personal safety app for women, with a shake-triggered SOS, live location sharing, a curated emergency helpline directory, and a WearOS companion that mirrors alerts to a paired smartwatch.

## Overview

This repository contains a hybrid mobile safety application (packaged as an installable PWA and as a native Android app via Capacitor) built around a single goal: let a user raise an emergency alert as fast as possible and get their location in front of the right people.

The app detects a device shake using the accelerometer and, without requiring the user to unlock the phone and tap through menus, captures a photo, grabs the current GPS position, and logs an alert record. A separate live-location screen lets a user share their current position and turn-by-turn route to a Discord channel or a Telegram chat with one tap. A searchable directory of Indian emergency helpline numbers (police, ambulance, women's helpline, disaster management, etc.) and a safety/self-defense tips reference are included alongside standard account features (email/password and Google sign-in via Firebase, trusted-contacts list, profile, recording history, and settings).

The codebase includes internal references (`vite.config.js`, and comments referencing "Angel Polytechnic College, Vashi") indicating it originated as a student/hackathon project named `Hackathon_Agnel`.

## Features

- **Shake-based SOS** — `GestureSOS` listens to the device accelerometer (`@capacitor/motion`), detects a shake pattern, and automatically captures a photo (`@capacitor/camera`) and the current GPS position (`@capacitor/geolocation`), writing an alert record (message, photo URI, contact list, timestamp) to Firebase Realtime Database.
- **Live location sharing** — a Mapbox GL map (`react-map-gl`) shows the user's position, lets them pick a destination, fetches a driving route from the Mapbox Directions API, and can push the current location plus turn-by-turn directions to a Discord webhook or a Telegram bot chat.
- **Alert Bell** — plays an audible alarm and invokes a Firebase callable Cloud Function (`triggerAlert`) to notify the backend.
- **Emergency contacts** — add and persist up to 5 trusted phone numbers (stored in `localStorage`).
- **Recording history** — lists locally stored SOS video recordings with in-browser playback, download, and delete.
- **Helpline directory** — a searchable list of Indian emergency numbers (National Emergency, Police, Fire, Ambulance, Women's Helpline, Child Helpline, and more) with tap-to-call links.
- **Safety & self-defense tips** — a static reference page of situational-awareness and self-defense guidance.
- **Guided onboarding** — a sequence of "landing feature" screens (`LF*` components) introduces Gesture SOS, Live Location, Emergency Contacts, Hidden Camera, and Alert Bell before the user reaches account creation.
- **Authentication & profile** — Firebase Authentication with email/password and Google sign-in, plus a profile form and settings/help sections.
- **Installable PWA** — configured through `vite-plugin-pwa` with a service worker that caches static assets and API responses.
- **WearOS companion** — a native Android Wear module (`android/womensafetywear`) listens for alert broadcasts over the Google Play Services Wearable Data API and displays the alert text on a paired watch.

## Tech Stack

| Category | Technologies |
|---|---|
| Frontend | React 18, Vite, React Router, Tailwind CSS, Framer Motion, animate.css, Lucide React, react-icons, Heroicons |
| Mobile shell | Capacitor 6 (Camera, Geolocation, Motion, Device, Local Notifications, Push Notifications, Toast plugins), targeting Android |
| Maps & routing | Mapbox GL / `react-map-gl` (map rendering and the Mapbox Directions API) |
| Backend / Cloud | Firebase (Authentication, Firestore, Realtime Database, Cloud Functions, Hosting) |
| Native companion | Android (Java), Google Play Services Wearable Data API, for a WearOS companion app |
| DevOps / Tooling | ESLint 9, PostCSS + Autoprefixer, Gradle (Android build), Firebase CLI |

## Architecture

```
React SPA (Vite build) ──┬── Firebase Auth / Firestore / Realtime Database
                          ├── Mapbox Directions API (routing)
                          ├── Discord webhook / Telegram Bot API (direct client-side alert fan-out)
                          └── Firebase Cloud Function "location-function"
                                  (HTTPS endpoint: receives {latitude, longitude}, relays to Discord)

Capacitor wraps the built web bundle (dist/) into a native Android app (android/app),
bridging Camera, Geolocation, Motion and Notification plugins to native APIs.

android/womensafetywear (WearOS module) listens for Wearable DataClient events
and renders the alert text on a paired smartwatch, independent of the phone UI.
```

Alerting is intentionally fanned out across multiple independent channels (Firebase Realtime Database, a Discord webhook, and the Telegram Bot API) directly from the client, rather than funneled through a single backend — so a trusted contact can still be reached if one channel is unavailable.

## Project Structure

```
women-safety/
├── src/
│   ├── components/        # ~25 screens: onboarding (LF*), SOS, location sharing,
│   │                       #   contacts, auth, profile, settings, helpline, safety tips
│   ├── router/AppRouter.jsx  # Route table for all screens
│   ├── firebase.js          # Firebase app/auth/Firestore/Realtime DB initialization
│   └── assets/               # Images, alarm/buzzer audio, sample SOS recordings
├── location-function/     # Firebase Cloud Function: relays {latitude, longitude} to a Discord webhook
├── android/                # Capacitor Android project
│   ├── app/                # Main Android app (applicationId: com.women_safety_app.www)
│   ├── womensafetywear/    # WearOS companion module (Java, Wearable Data API listener)
│   └── wearos/             # Additional WearOS module scaffold
├── public/                 # Static assets and PWA icon
├── dist/                   # Vite production build output (generated)
├── capacitor.config.json   # Capacitor app id / web directory
├── firebase.json / .firebaserc  # Firebase Hosting & Functions configuration
└── app/, build.gradle, settings.gradle, gradlew  # A Gradle-init "Hello World" Kotlin
                                                    # scaffold at the repo root, unrelated
                                                    # to the Android app under android/
```

## Getting Started

### Prerequisites

- Node.js 22 (declared in `package.json` `engines`)
- npm
- A Firebase project with Authentication, Firestore, and Realtime Database enabled (to match the configuration in `src/firebase.js`)
- Firebase CLI, to deploy the `location-function` Cloud Function and Hosting config
- Android Studio and a JDK, to build the Capacitor Android app and the WearOS companion modules

The project does not use `.env` files or `process.env` — the Firebase config and third-party service credentials (Mapbox, Discord, Telegram, WhatsApp Cloud API) are embedded directly in the relevant component source files rather than loaded from environment variables.

### Run locally

```bash
npm install
npm run dev
```

`npm run dev` runs `vite --host`, exposing the dev server on the local network so it can be opened from a phone browser during testing.

### Build

```bash
npm run build      # Production build to dist/
npm run preview    # Preview the production build
```

### Android app

```bash
npx cap sync android   # Copy the web build into the Capacitor Android project
npx cap open android    # Open android/ in Android Studio
```

Build and run from Android Studio, or from the `android/` directory with `./gradlew assembleDebug`. The `womensafetywear` and `wearos` Gradle modules build alongside the main app.

### Cloud Function (location-function)

```bash
cd location-function
npm install
firebase deploy --only functions:location-function
```

## Usage

A new user is taken through onboarding screens explaining Gesture SOS, Live Location, Emergency Contacts, Hidden Camera, and the Alert Bell, then to account creation or sign-in (email/password or Google). From the home screen, the user can:

- Trigger an SOS by shaking the device, or via the SOS/Alert Bell controls
- View their live location on the map and share it (with route) to Discord or Telegram
- Open the helpline directory to call a national or state emergency number
- Manage trusted emergency contacts, review past SOS recordings, read safety tips, and adjust settings from the sidebar navigation

## API Documentation

The repository exposes one HTTP endpoint, implemented as a Firebase Cloud Function in `location-function/index.js`:

| Method | Path | Body | Behavior |
|---|---|---|---|
| POST | `/` (deployed function URL / Hosting rewrite) | `{ "latitude": number, "longitude": number }` | Validates that both coordinates are present, posts a formatted "User location" message to a Discord webhook, and returns `200` on success, `400` for missing coordinates, or `500` on failure. |

Note: the client also calls a Firebase callable function named `triggerAlert` (see `src/components/IAlertBell.jsx` and `src/components/HomeMain.jsx`), but no matching function implementation exists anywhere in this repository.

## Design Decisions

- **Shake detection instead of a visible-only trigger** — `GestureSOS` compares consecutive accelerometer readings across all three axes against a threshold and counts consecutive shakes, so an SOS can be raised without unlocking the phone or navigating any menu.
- **Multi-channel alert fan-out** — rather than relying on a single notification path, alerts and live location are pushed to Firebase Realtime Database, a Discord webhook, and the Telegram Bot API in parallel from the client.
- **Native WearOS companion via the Data Layer API** — `android/womensafetywear` is a separate Android module using `Wearable.getDataClient(...)` to receive alert data pushed from the phone and display it on a paired watch, independent of whether the phone's screen is visible.
- **Client-first persistence for non-critical data** — emergency contacts, profile details, and recording history are kept in `localStorage` rather than round-tripped through Firestore, so those screens work without waiting on network reads/writes.
- **Single codebase for web, PWA, and Android** — the same React/Vite source is built once, served as an installable PWA (`vite-plugin-pwa`) and wrapped by Capacitor into the native Android app under `android/app`.

## Future Improvements

- Move the Firebase config and third-party credentials (Mapbox token, Discord webhook URL, Telegram bot token, WhatsApp Cloud API token in `LocationSharing.jsx`) out of source files and into environment-based configuration.
- Implement the `triggerAlert` callable Cloud Function referenced by the Alert Bell feature; it is currently called from the client but not defined in `location-function` or anywhere else in the repo.
- Correct the `axois` dependency name in `location-function/package.json` (the code imports `axios`, but the declared dependency is misspelled).
- Remove dependencies that are declared in `package.json` but not imported anywhere in `src/` (`leaflet`, `react-leaflet`, `ol`, `@ffmpeg/ffmpeg`).
- Reconcile `firebase.json`, which declares four Cloud Functions codebases (`functions`, `src` as `locationfunctions`, `locationfunction`, `location-function`) though only `location-function` exists in this repository.

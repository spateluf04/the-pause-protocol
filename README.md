# The Pause Protocol

**Pause · reflect · proceed.** A mobile app that puts a short, mindful pause between you and the apps that eat your time.

When you open a distracting app such as Instagram, TikTok or YouTube, an iOS Shortcuts automation sends you to The Pause Protocol first. You see a countdown and a quote. When the timer ends, you choose: go back to something better, or continue to the app. If you continue, that app is unlocked for a cooldown period you set.

Built with Expo (React Native) and Firebase.

## Features

- **Pause screen:** a countdown (3–300 seconds, default 10) with a random quote. If you leave the app and come back, the timer starts over, so you can't skip the pause by switching away.
- **Continue with a cooldown:** after the pause, "Continue" opens the target app and unlocks it for a cooldown you choose (1–1440 minutes, default 5).
- **Manual pause:** start a pause yourself from the home screen at any time.
- **Accounts:** email/password sign-up and login with Firebase Authentication. Each user's cooldown setting is saved in Firestore.
- **Quote library:** 100 starter quotes are stored in Firestore across three categories: mindfulness, focus and wellness. Admins can add, edit and delete quotes from the Manage Quotes screen.
- **Guided setup:** a step-by-step screen in the app that walks through installing the shortcut and creating automations.

## How it works

```
Open Instagram ──► iOS Personal Automation ("Instagram" is opened)
                        │
                        ▼
               "The Pause Protocol" shortcut
                        │  opens  pauseprotocol://pause?appName=Instagram
                        ▼
               Pause screen: countdown + quote
                        │
           ┌────────────┴─────────────┐
           ▼                          ▼
   Back to The Pause Protocol    Continue
                                     │ runs the shortcut with "CONTINUE|Instagram|<cooldown>"
                                     ▼
                         Shortcut records the cooldown and opens Instagram
```

The app and the shortcut communicate through URLs:

- **Shortcut → app:** the `pauseprotocol://pause` deep link opens the pause screen. It accepts these query parameters:
  - `appName`: the app being paused.
  - `seconds`: the pause length, clamped to 3–300.
  - `t`: an optional nonce that forces the timer to restart.
- **App → shortcut:** `shortcuts://run-shortcut?name=The Pause Protocol&input=text&text=CONTINUE|<app>|<minutes>`.

The shortcut keeps cooldowns in a small `cooldowns.json` file in the user's iCloud Drive → Shortcuts folder. That file contains only app names and timestamps.

> **Platform note:** app interception relies on iOS Shortcuts automations, so it is iOS-only. The app itself, including the manual pause, accounts and quotes, runs on iOS, Android and web.

## Tech stack

| Area | Tools |
|---|---|
| App | Expo SDK 54, React Native 0.81, React 19, TypeScript |
| Navigation | Expo Router (file-based routing) |
| Backend | Firebase Authentication, Cloud Firestore |
| Scripts | Node.js and the `firebase-admin` SDK (quote seeding) |

## Getting started

### Prerequisites

- Node.js 18 or newer
- The [Expo Go](https://expo.dev/go) app, or an iOS or Android simulator
- A Firebase project of your own

### 1. Install

```bash
git clone <this-repo-url>
cd the-pause-protocol
npm install
```

### 2. Connect your own Firebase project

The repo is wired to the original team's Firebase project. To run your own copy:

1. In the [Firebase console](https://console.firebase.google.com/), create a project and add a **Web app**.
2. Enable **Authentication → Email/Password** and create a **Cloud Firestore** database.
3. Replace the `firebaseConfig` object in [`services/firebase.ts`](services/firebase.ts) with your project's config.
4. Add Firestore security rules (see [Security](#security) below).

> Firebase web config values, including `apiKey`, are public identifiers by design. Firestore security rules are what protect your data, not the secrecy of these values.

### 3. Seed the quote library (optional)

1. In the Firebase console, open **Project settings → Service accounts → Generate new private key**.
2. Save the downloaded file as `serviceAccount.json` in the project root. It is gitignored. **Never commit it.**
3. Run:

```bash
node scripts/seedQuotes.js
```

### 4. Run

```bash
npx expo start      # then press i (iOS), a (Android) or w (web), or scan the QR code with Expo Go
```

### 5. Set up the shortcut (iOS)

Follow **Setup Instructions** in the app. In short:

1. Install "The Pause Protocol" shortcut, then run it once from the Shortcuts app and approve its permission prompts.
2. Go to **Shortcuts → Automation → New Personal Automation → App**. Pick a distracting app, choose **Is Opened → Run Immediately**, and turn off **Notify When Run**.
3. Add a **Text** action containing the app's exact name, then **Run Shortcut → The Pause Protocol** with that text as input.
4. Repeat for each app you want to pause, and set your cooldown under **Profile**.

Instagram, TikTok, YouTube, X, Snapchat, Reddit and Facebook work out of the box. To support any other app, add its URL scheme to the `AppSchemes` dictionary inside the shortcut.

> The shortcut link in [`app/(app)/setup.tsx`](app/(app)/setup.tsx) is a placeholder (`SHORTCUT_URL`). Point it at your own hosted copy of the shortcut.

## Project structure

```
app/
  index.tsx            Entry point; redirects to login
  pause.tsx            Pause screen (deep-link target)
  (auth)/              Login and registration
  (app)/
    home.tsx           Home: profile, setup, manual pause, admin entry point
    profile.tsx        Account info and cooldown setting
    setup.tsx          Shortcut setup guide
    adminQuotes.tsx    Quote management (admins only)
components/            Shared UI components
hooks/useUserRole.ts   Reads the signed-in user's role from Firestore
services/
  firebase.ts          Firebase initialization
  quoteService.ts      Quote CRUD
  settingsService.ts   Cooldown setting (clamped to 1–1440 minutes)
scripts/seedQuotes.js  Seeds Firestore with the starter quotes
```

### Data model

| Collection | Document | Fields |
|---|---|---|
| `users` | `{uid}` | `email`, `cooldownMinutes`, `role` (`"admin"` for admins) |
| `quotes` | auto ID | `text`, `author`, `category` |

## Security

The app checks admin access on the client, so **Firestore security rules must enforce it on the server**. At minimum, your rules should:

- let signed-in users read quotes and read and write their own `users/{uid}` document,
- **block users from setting or changing their own `role` field**, and
- allow writes to `quotes` only from admins.

Assign admins outside the app, for example by setting `role: "admin"` in the Firebase console or with an Admin SDK script. The in-app admin code from the original course build has been disabled.

## Team

Built as a software engineering group project by:

- Samir Patel ([@spateluf04](https://github.com/spateluf04))
- Dipak Mehta ([@Dmehta06](https://github.com/Dmehta06))
- Kush ([@kushp163](https://github.com/kushp163))

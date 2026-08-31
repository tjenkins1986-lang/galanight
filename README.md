# Gala Night

A tiny travel memory book for two. Every trip gets logged with a few
highlight questions, and you can flip back through past trips any time.

Static HTML/JS app backed by Firebase Auth (sign-in) and Firestore
(storage). No build step, no framework.

## Setup

### 1. Create a Firebase project

1. Go to the [Firebase console](https://console.firebase.google.com/) and create a new project.
2. In **Build → Authentication → Sign-in method**, enable the **Email/Password** provider.
3. In **Build → Authentication → Users**, manually add two users (you and your wife) with email + password. There's no public sign-up page in this app on purpose — accounts are created by hand in the console.
4. In **Build → Firestore Database**, create a database (production mode is fine).
5. In **Project settings → General → Your apps**, add a **Web app** and copy the `firebaseConfig` object it gives you.

### 2. Add your config

Paste your config into [`js/firebase-config.js`](js/firebase-config.js), replacing the placeholder values:

```js
export const firebaseConfig = {
  apiKey: "...",
  authDomain: "...",
  projectId: "...",
  storageBucket: "...",
  messagingSenderId: "...",
  appId: "...",
};
```

### 3. Deploy the Firestore security rules

[`firestore.rules`](firestore.rules) restricts all reads/writes to signed-in users. Deploy it with the [Firebase CLI](https://firebase.google.com/docs/cli):

```bash
npm install -g firebase-tools
firebase login
firebase init firestore   # point it at your existing project, keep the default rules file name
firebase deploy --only firestore:rules
```

(Or paste the contents of `firestore.rules` into the console's Firestore **Rules** tab directly.)

### 4. Run it locally

No build step needed — just serve the folder statically, e.g.:

```bash
python3 -m http.server 8000
```

Then open `http://localhost:8000/login.html`.

### 5. Deploy to GitHub Pages

1. Push this repo to GitHub (already done if you're reading this from the repo).
2. In the repo's **Settings → Pages**, set the source to the `main` branch, root folder.
3. Once published, in the Firebase console under **Authentication → Settings → Authorized domains**, add your `*.github.io` domain so sign-in works from the deployed site.

## How it works

- `login.html` — sign in with the email/password accounts you created in step 1.
- `index.html` — lists all trips, most recent first. Links to each trip's detail page.
- `add.html` — form to log a new trip: name, location, start/end dates, and the six highlight questions.
- `entry.html` — read-only view of one trip's memory.

Every trip is stored as a document in the Firestore `memories` collection with these fields:

| Field | Type |
| --- | --- |
| `tripName` | string |
| `location` | string |
| `startDate` / `endDate` | string (`YYYY-MM-DD`) |
| `ate` / `drank` / `saw` / `did` / `wishedDone` / `highlight` | string |
| `createdAt` | Firestore timestamp |
| `createdBy` | string (uid of whoever added it) |

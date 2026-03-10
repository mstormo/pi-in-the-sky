# Pi in the Sky — Firebase Setup

The global leaderboard requires a free Firebase project. Setup takes ~3 minutes.

## 1. Create a Firebase Project

1. Go to [Firebase Console](https://console.firebase.google.com/)
2. Click **Add project**
3. Name it (e.g., `pi-in-the-sky`)
4. Disable Google Analytics (not needed) → **Create project**

## 2. Enable Firestore

1. In the Firebase console, click **Build → Firestore Database**
2. Click **Create database**
3. Choose **Start in production mode** → pick a region → **Enable**

## 3. Set Firestore Security Rules

In **Firestore → Rules**, replace the default rules with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /scores/{score} {
      // Anyone can read scores
      allow read: if true;

      // Anyone can create a score (with validation)
      allow create: if request.resource.data.keys().hasAll(['name', 'mode', 'score', 'timestamp'])
                    && request.resource.data.name is string
                    && request.resource.data.name.size() > 0
                    && request.resource.data.name.size() <= 40
                    && request.resource.data.mode in ['sequential', 'chunks', 'fill', 'speed']
                    && request.resource.data.score is number
                    && request.resource.data.score >= 0
                    && request.resource.data.score <= 10000;

      // No one can modify or delete scores
      allow update, delete: if false;
    }
  }
}
```

Click **Publish**.

## 4. Get Your Web App Config

1. In the Firebase console, click the **gear icon → Project settings**
2. Under **Your apps**, click the **web icon** (`</>`)
3. Register an app name (e.g., `pi-web`) — skip Firebase Hosting
4. Copy the `firebaseConfig` object

## 5. Paste Config into index.html

Open `index.html` and find this section near the top of the `<script>`:

```js
var FIREBASE_CONFIG = null;
```

Replace `null` with your config:

```js
var FIREBASE_CONFIG = {
  apiKey: "AIzaSy...",
  authDomain: "pi-in-the-sky-xxxxx.firebaseapp.com",
  projectId: "pi-in-the-sky-xxxxx",
  storageBucket: "pi-in-the-sky-xxxxx.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abcdef123456"
};
```

## 6. Deploy

Commit and push to GitHub. The leaderboard will be live.

## Notes

- The Firebase config is safe to include in client-side code — Firestore security rules protect the data
- The free Spark plan allows 50K reads + 20K writes per day — plenty for a family game
- To clear fake/spam scores: delete them in the Firebase Console → Firestore → scores collection

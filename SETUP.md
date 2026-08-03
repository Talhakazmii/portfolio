# Firebase Setup — Portfolio CMS

Your site now has an admin page (`admin.html`) where you can log in and edit your bio, skills, and projects. The public `index.html` reads that content from Firebase automatically — until you configure Firebase, it just shows the built-in default content, so nothing breaks in the meantime.

## 1. Create a Firebase project

1. Go to https://console.firebase.google.com and sign in with your Google account.
2. Click **Add project**, name it (e.g. `talha-portfolio`), and finish the wizard (Google Analytics is optional — you can skip it).

## 2. Register a web app

1. In the project overview, click the **`</>`** (web) icon to add a web app.
2. Give it a nickname (e.g. `portfolio-site`) — you don't need Firebase Hosting.
3. Firebase will show a `firebaseConfig` object. Copy the values into `firebase-config.js` (the `apiKey`, `authDomain`, `projectId`, `storageBucket`, `messagingSenderId`, `appId` fields).

## 3. Enable Authentication

1. In the left sidebar, go to **Build → Authentication → Get started**.
2. Under **Sign-in method**, enable **Email/Password**.
3. Go to the **Users** tab → **Add user** → enter your email (`talhakazmi301@gmail.com`) and a password you'll use to log into `admin.html`.

## 4. Enable Firestore

1. Go to **Build → Firestore Database → Create database**.
2. Choose **Start in production mode**, pick a region close to you, click **Create**.
3. Go to the **Rules** tab and replace the contents with:

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {
    match /portfolio/{docId} {
      allow read: if true;
      allow write: if request.auth != null && request.auth.token.email == 'talhakazmi301@gmail.com';
    }
  }
}
```

4. Click **Publish**.

This means: anyone can view your portfolio content, but only your logged-in account can edit it.

## 5. Fill in `firebase-config.js`

Open `firebase-config.js` and paste in the config values from step 2. Save the file.

## 6. Try it out

1. Open `admin.html` in your browser, log in with the email/password from step 3.
2. Click **Load starter content** to pre-fill the form with your current bio/skills/projects.
3. Edit anything you like, then click **Save Changes**.
4. Open `index.html` — it should now show your updated content (refresh if it was already open).

## 7. Deploy to GitHub Pages

Push `index.html`, `admin.html`, and `firebase-config.js` together to your repo, then enable Pages (Settings → Pages → source: main branch). Your API key being visible in the JS is normal and expected for Firebase web apps — real security comes from the Firestore rules above, not from hiding the key.

## Notes

- Only the email set in step 3 can save changes — anyone else who finds `admin.html` will be able to see the login screen but can't write data (blocked by both auth and Firestore rules).
- If you ever want to add a second admin, add another email check with `||` in the rules, and create another user in step 3.

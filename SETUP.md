# Firebase Setup — Portfolio CMS

Your site now has an admin page (`admin.html`) where you can log in and edit your bio, skills, experience, certifications, and projects. The public `index.html` reads that content from Firebase automatically. While it's loading it shows a skeleton loading animation, and any section with no data yet shows a "No ... Added Yet" message instead of fake placeholder content — so nothing looks broken before you've configured Firebase or added your first entries.

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

## 5. Set up Cloudinary (for your profile photo and certification uploads)

File uploads (profile photo, certification files) go through Cloudinary instead of Firebase Storage — Firebase changed Cloud Storage to require a paid Blaze plan (with a credit card on file) as of February 2026, while Cloudinary's free tier needs no card at all.

1. Sign up free at https://cloudinary.com — no credit card required.
2. Your **Cloud Name** is shown right on the Cloudinary Dashboard homepage after signup.
3. Create an **unsigned** upload preset (required so the browser can upload directly without a backend or secret key):
   - Go to **Settings** (gear icon, top right) → **Upload** tab → **Upload presets** → **Add upload preset**.
   - Set **Signing Mode** to **Unsigned**.
   - (Optional) Under "Folder", you can leave it blank — the app already organizes uploads into `portfolio/profile` and `portfolio/certifications` folders automatically.
   - Click **Save**, then copy the preset's name (shown in the preset list).
4. Open `cloudinary-config.js` and fill in `CLOUDINARY_CLOUD_NAME` and `CLOUDINARY_UPLOAD_PRESET` with the values from steps 2 and 3.

Because the upload preset is unsigned, anyone who inspects your site's JS could technically upload files to your Cloudinary account too — that's expected and normal for this kind of setup (Cloudinary's free tier is generous, and you can always regenerate/delete the preset if it's ever abused). Firestore itself still stays locked down to only your login.

## 6. Fill in `firebase-config.js`

Open `firebase-config.js` and paste in the config values from step 2. Save the file.

## 7. Try it out

1. Open `admin.html` in your browser, log in with the email/password from step 3.
2. Click **Load starter content** to pre-fill the form with your current bio/skills/projects.
3. Upload a profile photo, and/or add an experience entry or certification (choose a file, then click **Upload** — you'll see a live progress bar).
4. Click **Save Changes**.
5. Open `index.html` — it should now show your updated content (refresh if it was already open).

## 8. Deploy to GitHub Pages

Push `index.html`, `admin.html`, `firebase-config.js`, and `cloudinary-config.js` together to your repo, then enable Pages (Settings → Pages → source: main branch). Your API key being visible in the JS is normal and expected for Firebase web apps — real security comes from the Firestore rules above, not from hiding the key.

## Notes

- Only the email set in step 3 can save changes to your content — anyone else who finds `admin.html` will be able to see the login screen but can't write data (blocked by both auth and Firestore rules).
- If you ever want to add a second admin, add another email check with `||` in the Firestore rules, and create another user in step 3.
- Uploaded files live in Cloudinary, not Firestore — Firestore only stores the resulting URLs.

# Score Update Console v2

Here is a comprehensive guide to understanding the **Score Update Console v2** web application and setting up Firebase to get it up and running.

---

## 🏏 Project Overview
The **Score Update Console v2** is a React.js based web application designed to act as an admin console for live updating cricket scores. Judging by some of the hardcoded logic, it seems specifically tailored for a match context between two teams: "MCG" and "RCG" (likely Mahinda College vs Richmond College).

The app allows an authenticated user to perform duties like setting up a match, updating current innings, tracking ball-by-ball scores, managing wickets, ending overs, and mapping new players dynamically.

## 🏗️ Architecture & Technology Stack
*   **Core Framework**: React 18, bootstrapped using `create-react-app`.
*   **Routing**: `react-router-dom` v6 for client-side routing.
*   **State Management**: React Contexts (`AuthContext` and `InningContext`).
*   **UI/Styling**: Material-UI (MUI `v5.11`) alongside Emotion for styled components.
*   **Backend & Real-Time Database**: Firebase (Authentication and Firestore).

## 📁 Directory Structure
All core logic exists inside the `/src` folder. Here's what each sub-folder is responsible for:
*   **`src/firebase/config.js`**: Contains the initialization file connecting the React app to your Firebase project.
*   **`src/context/`**: Contains global state providers.
    *   `AuthContext.js`: Manages the currently logged-in user state.
    *   `InningContext.js`: Sets up a real-time listener to Firestore that actively pushes changes from the cloud to the app (for live team stats, current batsmen, active bowler, and balls/score).
*   **`src/pages/`**: Holds the main views linked to routes in `App.js`.
    *   `/login`: The authentication gate.
    *   `/` (Home/Dashboard): The main live scoring dashboard interface.
    *   `/setup`, `/players`, `/innings`, `/wicket`, `/over`: Dedicated views to perform scoring actions.
*   **`src/hooks/`**: Custom hooks acting as controllers that communicate with Firestore. For instance, `useMatchSetup.js` initializes a match between two players and a bowler. `useAddScore.js` pushes the runs scored per ball, and `useAddWicket.js` handles bowler stats and out statuses.
*   **`src/components/`**: Reusable generic UI elements like the Dashboard wrapper and Menu Items.

---

## 🔥 How to Setup Firebase for this App
The code explicitly relies on specific parts of Firebase. Follow these steps to set it up:

### Step 1: Create a Project
1. Go to the [Firebase Console](https://console.firebase.google.com/) and create a new project.
2. Turn off Google Analytics if you don't need it for testing.

### Step 2: Enable Firebase Authentication
Because the app routes are protected by `useAuthContext`, you must login to see the console:
1. In the Firebase Console, go to **Build > Authentication**.
2. Click **Get Started**, navigate to the **Sign-in method** tab, and enable **Email/Password**.
3. Go to the **Users** tab and **Add User**. Create an email and password directly there—you will use these exact credentials to log into the web app's `/login` screen.

### Step 3: Enable Firestore Database & Structure
The custom hooks interact with a specific database structure.
1. In the Firebase Console, go to **Build > Firestore Database** and click **Create database**.
2. Select **Start in test mode** (this disables security rules for 30 days so you can test without permission errors). Choose the closest region.
3. According to the current logic (namely in `useMatchSetup` and `InningContext`), the application requires the existence of a specific document to work properly. We must create this manually right away:
   *   Click **Start collection**.
   *   Name the Collection: **`main`**.
   *   Create a Document with the ID: **`live`**.
   *   If it asks you to add a field right away, just add a dummy string field (e.g., `Field: test, Type: string, Value: test`) and hit **Save**. The custom hooks (`useMatchSetup`) will overwrite this document with the actual properties later, but the listener crashes if the document itself doesn't exist.

The application automatically creates other collections such as `batting` and `bowling` via the app as actions occur.

### Step 4: Configure the Codebase
1. In the Firebase Console, click the **Gear Icon (Project Settings)** at the top left and scroll to the bottom.
2. Under "Your apps", select the **Web icon (`</>`)** to register a web app. Give it a name and register.
3. Copy the `firebaseConfig` block provided.
4. Go back to your code: open `src/firebase/config.js`. You will notice commented-out dummy configuration. Replace it with your new setup:

```javascript
import { initializeApp } from 'firebase/app';
import { getFirestore, serverTimestamp } from 'firebase/firestore';
import { getAuth } from 'firebase/auth';

const firebaseConfig = {
    apiKey: "YOUR_API_KEY",
    authDomain: "YOUR_PROJECT.firebaseapp.com",
    projectId: "YOUR_PROJECT",
    storageBucket: "YOUR_PROJECT.appspot.com",
    messagingSenderId: "123456789",
    appId: "YOUR_APP_ID",
    measurementId: "YOUR_MEASUREMENT_ID"
};

// init firebase
initializeApp(firebaseConfig);

// init service
const db = getFirestore();
const auth = getAuth();
const timestamp = serverTimestamp();

export { db, auth, timestamp };
```

### Step 5: Start the App
Now that Firebase is mapped to the web app, run the following commands in your terminal to view the project:
1. Install the missing npm packages: `npm install`
2. Start the application: `npm start`

You will immediately be redirected to the login page. Login with the user you created in Step 2, navigate to the `/setup` tab if necessary to configure players, and the dashboard should light up!

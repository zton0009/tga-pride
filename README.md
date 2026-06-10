# 🌈 Pride Community Voice Tracker

A single-file, real-time collaborative response tracker for Pride Month events. Participants enter their initials and tick boxes — all responses sync live across every device connected to the same Firebase project.

Hosted as a static file on GitHub Pages. No server, no build step.

---

## Quick start

1. Clone or download this repo
2. Open `pride-tracker-firebase.html` in a browser
3. Enter your Firebase credentials in the setup modal (see [Firebase Setup](#firebase-setup) below)
4. Push to GitHub and enable Pages — done

---

## What to customise

All configuration lives at the top of the `<script>` tag inside `pride-tracker-firebase.html`. You do not need to touch anything else.

### Number of response rows

```js
const ROWS = 15; // slots per column
const COLS = 2;  // side-by-side columns
```

`ROWS × COLS` = total number of people who can respond. The default is `15 × 2 = 30`. Increase `ROWS` to allow more responses; add a third column by setting `COLS = 3` (note: this will make the table very wide on smaller screens).

---

### Questions

Each question is an object in the `QUESTIONS` array:

```js
const QUESTIONS = [
  {
    id: 1,                        // unique number — do not repeat
    icon: "ti-heart",             // Tabler icon class (see icons.getbootstrap.com/tabler)
    q: "Your question text here", // the question shown in the header bar
    opts: ["Option A", "Option B", "Option C"], // checkbox column labels (2–7 recommended)
    hc: "#D4537E",  // header background colour (hex)
    tc: "#72243E"   // column label text colour (hex — usually a darker shade of hc)
  },
  // ... more questions
];
```

**To add a question:** copy an existing object, paste it at the end of the array, and change `id`, `q`, `opts`, `hc`, and `tc`. Make sure to add a comma after the previous object.

**To remove a question:** delete its object from the array (including the trailing comma on the object before it).

**To reorder questions:** cut and paste the objects into the order you want.

**Choosing colours:** each question gets its own colour. Pick a mid-tone for `hc` (used as the header background and filled checkbox colour) and a darker version of the same hue for `tc` (used for the angled column labels). Free tool: [coolors.co](https://coolors.co).

**Choosing icons:** any icon from [Tabler Icons](https://tabler.io/icons) works. Click an icon, copy the class name (e.g. `ti-flame`), and paste it into the `icon` field. The `ti-` prefix is required.

---

### Page title

Change the browser tab title at the top of the file:

```html
<title>Pride Community Voice Tracker</title>
```

Change the large rainbow heading inside the `QUESTIONS` section — search for:

```js
e('div', { className: 'pt-title' }, '🌈 Happy Pride Month 🌈'),
```

And replace the string with your event name.

---

### Firebase database path

By default all data is stored at `pride-tracker/v1` in your Realtime Database. If you run multiple events from the same Firebase project, change this path so they don't share data:

```js
const ref = db.ref('pride-tracker/v1'); // change 'v1' to 'event-2026' etc.
```

Search for `db.ref(` in the script to find it.

---

## Firebase setup

Firebase Realtime Database is a free hosted JSON store that pushes updates to all connected clients instantly. The free Spark plan is more than enough for an event tracker.

### Step 1 — Create a Firebase project

1. Go to [console.firebase.google.com](https://console.firebase.google.com)
2. Click **Add project**
3. Enter a project name (e.g. `pride-tracker-2026`)
4. Disable Google Analytics if you don't need it, then click **Create project**

---

### Step 2 — Create a Realtime Database

1. In the left sidebar, under **Build**, click **Realtime Database**
2. Click **Create database**
3. Choose a database location (pick the region closest to your event)
4. When asked for security rules, select **Start in test mode** — this allows open read/write, which is fine for a temporary event tracker. You can lock it down later (see [Security rules](#security-rules))
5. Click **Enable**
6. Copy the database URL shown at the top of the page — it looks like:
   ```
   https://your-project-default-rtdb.asia-southeast1.firebasedatabase.app
   ```
   You'll need this in Step 4.

---

### Step 3 — Register a web app

1. Click the ⚙ gear icon next to **Project Overview** in the top-left → **Project settings**
2. Scroll down to **Your apps** and click the **</>** (Web) icon
3. Give the app a nickname (e.g. `pride-tracker-web`) — you do not need Firebase Hosting
4. Click **Register app**
5. Firebase will show you a config block like this:

```js
const firebaseConfig = {
  apiKey: "AIzaSyABC123...",
  authDomain: "your-project.firebaseapp.com",
  databaseURL: "https://your-project-default-rtdb.asia-southeast1.firebasedatabase.app",
  projectId: "your-project",
  storageBucket: "your-project.appspot.com",
  messagingSenderId: "123456789",
  appId: "1:123456789:web:abc123"
};
```

You only need three values from this:

| Field in modal | Value from Firebase config |
|---|---|
| **API Key** | `apiKey` |
| **Project ID** | `projectId` |
| **Database URL** | `databaseURL` |

---

### Step 4 — Connect the tracker

Open the HTML file in a browser (or visit your GitHub Pages URL). The setup modal appears automatically on first load.

Enter the three values from Step 3 and click **⚡ Connect**.

The tracker stores these in your browser's `localStorage` under the key `pt_firebase_config`, so you won't be asked again on that device. To reset or change the config, click the **⚙ Config** button in the sync status bar.

---

### Security rules

The default test-mode rules allow anyone with the database URL to read and write. For a public event, this is usually fine. To lock things down after the event or restrict to an event window, update your rules in **Realtime Database → Rules**:

**Open (default test mode — expires after 30 days):**
```json
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
```

**Time-locked to your event window:**
```json
{
  "rules": {
    ".read": "now > 1749081600000 && now < 1749340800000",
    ".write": "now > 1749081600000 && now < 1749340800000"
  }
}
```
Replace the timestamps with your event start and end in Unix milliseconds. Use [unixtimestamp.com](https://www.unixtimestamp.com) to convert dates.

**Read-only after the event (results visible but no new entries):**
```json
{
  "rules": {
    ".read": true,
    ".write": false
  }
}
```

---

### Resetting data between sessions

To clear all responses and start fresh (e.g. for a second session on the same day), go to **Realtime Database → Data** in the Firebase console, find the `pride-tracker` node, hover over it, and click the **×** (delete) button.

Alternatively, change the `db.ref()` path in the script (e.g. from `v1` to `v2`) — this starts a clean slate without deleting historical data.

---

## GitHub Pages hosting

1. Push `pride-tracker-firebase.html` to a GitHub repository (public or private with Pages enabled)
2. Go to **Settings → Pages**
3. Under **Source**, select **Deploy from a branch** → choose `main` (or `master`) and `/ (root)`
4. Click **Save**
5. Your tracker will be live at `https://yourusername.github.io/your-repo-name/pride-tracker-firebase.html` within a minute or two

> **Tip:** Rename the file to `index.html` so the URL is just `https://yourusername.github.io/your-repo-name/`

---

## How sync works

- On load, the app subscribes to the Firebase path with `ref.on('value', ...)` — this fires immediately with all existing data, then again whenever any client writes a change
- Checkbox toggles and initials updates are debounced by 400 ms before writing, so rapid clicks don't create unnecessary writes
- The sync status pill in the header shows the current state: **Offline**, **Connecting…**, **Live · syncing**, **Saving…**, or **Sync error**
- If Firebase is unreachable (no credentials, network error, or bad rules), the tracker falls back to local-only mode and still works normally — data just won't sync

---

## Free tier limits

Firebase Spark (free) limits that apply to this tracker:

| Limit | Free allowance | Typical event usage |
|---|---|---|
| Simultaneous connections | 100 | Well within for most events |
| GB stored | 1 GB | Tracker data is a few KB |
| GB downloaded per month | 10 GB | Each page load is ~5 KB |

You would need an event with thousands of simultaneous users before approaching any limits.

---

## Troubleshooting

**"Sync error" status** — Check your database rules allow read/write. In the Firebase console go to Realtime Database → Rules and confirm they are not in locked mode (the 30-day test period may have expired).

**Config modal keeps appearing** — The browser's `localStorage` may be blocked (e.g. in a private/incognito window or with strict privacy settings). Enter credentials each session, or the tracker will work in offline mode.

**Changes from one device don't appear on another** — Confirm both devices entered the same Project ID and Database URL. Even a trailing slash difference will create separate database paths.

**All data disappeared** — Check if the test-mode rules expired (30 days) and locked the database. Update the rules to re-enable access.

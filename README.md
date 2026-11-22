[README.md](https://github.com/user-attachments/files/35229015/README.md)
# Well-Era — Anti-Gravity Watch Website

Landing page for the Well-Era watch brand, split into a **frontend** and a **backend**.
The backend sends form submissions to a **Google Form**, so every signup shows up in your form's Responses tab (or a linked Google Sheet).

```
well-era/
├── frontend/              # Static website (HTML / CSS / JS)
│   ├── index.html
│   ├── style.css
│   ├── script.js
│   └── images/
├── backend/               # Node.js + Express API
│   ├── src/
│   │   ├── server.js      # Routes, validation, rate limiting, serves frontend
│   │   └── googleForm.js  # Google Form integration
│   ├── .env.example       # Config template
│   └── package.json
└── README.md
```

## How the Google Form link works

| Where | What happens |
|-------|--------------|
| Footer newsletter form ("Join") | Frontend sends the email to `POST /api/subscribe`. The backend validates it and forwards it to your Google Form. |
| "Reserve Yours Now" button | Frontend calls `GET /api/form-link`. The backend returns your Google Form's public link, which opens in a new tab. |

## Setup

### 1. Prepare your Google Form
1. Create a form at [forms.google.com](https://forms.google.com) with one **Short answer** question for **Email**.
2. Click **Send → link (🔗)** and copy the link. It looks like  
   `https://docs.google.com/forms/d/e/1FAIpQLSc.../viewform`  
   The part between `/d/e/` and `/viewform` is your **`GOOGLE_FORM_ID`**.
3. Find the **entry id** of the Email question:
   - Open the form's public link, right-click → **Inspect**, and search the page for `entry.`  
     — or use **⋮ → Get pre-filled link**, type any value in the Email field, click **Get link**, and read the `entry.1234567890=...` part of the URL.
   - That `entry.1234567890` is your **`GOOGLE_FORM_EMAIL_ENTRY`**.
4. In the form's settings, make sure **"Limit to 1 response"** and **"Require sign-in"** are **off**, otherwise the backend can't submit.

### 2. Configure the backend
```bash
cd backend
cp .env.example .env
```
Edit `.env`:
```
PORT=3000
CORS_ORIGIN=http://localhost:3000
GOOGLE_FORM_ID=<your form id>
GOOGLE_FORM_EMAIL_ENTRY=entry.<your email field id>
```

### 3. Install and run
Requires **Node.js 18+**.
```bash
cd backend
npm install
npm start
```
Open **http://localhost:3000** — the backend also serves the frontend, so this is all you need.

## API

| Method | Route | Description |
|--------|-------|-------------|
| `GET`  | `/api/health` | Server status and whether the Google Form is configured |
| `POST` | `/api/subscribe` | Body: `{ "email": "you@example.com" }` → saved to the Google Form |
| `GET`  | `/api/form-link` | Returns `{ "url": "<public Google Form link>" }` |

`POST /api/subscribe` is rate-limited to 5 requests per minute per IP.

## Hosting frontend and backend separately

1. Deploy `backend/` to a Node host (Render, Railway, Fly.io, etc.) and set the environment variables from `.env.example` there.
2. In `frontend/script.js`, set `API_BASE_URL` to your backend URL:
   ```js
   const API_BASE_URL = 'https://your-backend.example.com';
   ```
3. Set `CORS_ORIGIN` on the backend to the URL where the frontend is hosted.
4. Deploy `frontend/` to any static host (Netlify, Vercel, GitHub Pages, ...).



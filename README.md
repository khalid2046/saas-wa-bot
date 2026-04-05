# WhatsApp SaaS Bot Platform

A multi-tenant SaaS platform where business owners sign up, connect their WhatsApp number by scanning a QR code, and run automated bots for their customers.

**Two bot services:**
- 🔑 **CDK Key Activation** — activates ChatGPT Plus subscriptions via OpenAI session tokens
- 💳 **NayaPay Purchase** — lets customers order ChatGPT plans and receive email receipts

---

## Project Structure

```
saas-whatsapp-bot/
├── artifacts/
│   ├── api-server/        ← Node.js + TypeScript backend
│   │   └── src/
│   │       ├── index.ts       — Express server entry point
│   │       ├── db.ts          — PostgreSQL database layer
│   │       ├── wa-manager.ts  — WhatsApp session manager (Baileys)
│   │       ├── handler.ts     — Bot conversation engine
│   │       ├── cdk.ts         — CDK API integration
│   │       ├── gmail.ts       — Email receipt sender
│   │       └── platform.ts    — REST API routes
│   └── platform/          ← React frontend dashboard
│       └── src/
│           ├── pages/         — Dashboard, Customers, Activations, Purchases, Messages, Settings
│           ├── components/    — Layout/sidebar
│           ├── context/       — Auth context
│           └── lib/api.ts     — Axios API client
├── render.yaml            ← One-click Render deployment
├── package.json           ← pnpm workspace root
└── pnpm-workspace.yaml
```

---

## ⚡ Quick Deploy to Render (Recommended)

### Step 1 — Push to GitHub

1. Create a new repository on [github.com](https://github.com)
2. Push this entire folder to it:
   ```bash
   git init
   git add .
   git commit -m "Initial commit"
   git remote add origin https://github.com/YOUR_USERNAME/YOUR_REPO.git
   git push -u origin main
   ```

### Step 2 — Deploy on Render

1. Go to [render.com](https://render.com) and sign up (free)
2. Click **"New"** → **"Blueprint"**
3. Connect your GitHub account and select your repository
4. Render will read `render.yaml` and automatically create:
   - A **Web Service** for the app
   - A **PostgreSQL database**
5. Before clicking Deploy, you must fill in these environment variables:

| Variable | Value | Where to get it |
|---|---|---|
| `CDK_API_BASE` | Your CDK API URL | e.g. `https://keys.ovh/api/v1` |
| `CDK_API_KEY` | Your CDK API key | From your CDK dashboard |

6. Click **"Apply"** — Render will build and deploy everything automatically.

> `JWT_SECRET` and `DATABASE_URL` are generated automatically by Render.

### Step 3 — Access your platform

Once deployed, Render gives you a URL like `https://wa-bot-platform.onrender.com`.

- Open it in your browser → you'll see the **Login page**
- Click **"Create account"** to register your first tenant
- Go to **Dashboard** → click **"Connect WhatsApp"** → scan the QR code
- Your bot is live! ✅

---

## 🖥️ Running Locally (Development)

### Prerequisites
- Node.js 18+
- pnpm (`npm install -g pnpm`)
- PostgreSQL database (local or cloud)

### Setup

1. **Clone the repo:**
   ```bash
   git clone https://github.com/YOUR_USERNAME/YOUR_REPO.git
   cd saas-whatsapp-bot
   ```

2. **Install dependencies:**
   ```bash
   pnpm install
   ```

3. **Create environment file:**
   ```bash
   cp .env.example .env
   ```
   Then edit `.env` with your values (see below).

4. **Start development servers:**
   ```bash
   pnpm dev
   ```
   - Backend runs on: `http://localhost:3000`
   - Frontend runs on: `http://localhost:5173` (with proxy to backend)

---

## 🔐 Environment Variables

Create a `.env` file in the root of `artifacts/api-server/`:

```env
# Required
DATABASE_URL=postgresql://user:password@localhost:5432/wabotplatform
JWT_SECRET=your-very-long-random-secret-here
CDK_API_BASE=https://keys.ovh/api/v1
CDK_API_KEY=your_cdk_api_key_here

# Optional (tenants set this per-account in Settings page)
PORT=3000
```

---

## 🤖 How the Bot Works

### Main Menu
Any message from a new user (or sending `*`) shows:
```
👋 Welcome! Please choose an option:

1️⃣  CDK Key Activation
2️⃣  NayaPay Purchase

Reply with 1 or 2
```

### CDK Key Activation Flow
```
User → 1
Bot  → "Send your CDK key (16 chars)"
User → AB12CD34EF56GH78
Bot  → "Key valid! Now send your OpenAI session JSON"
User → {"accessToken":"eyJ...", "user":{...}}
Bot  → "🎉 Activated! Email: user@gmail.com | Plan: Plus"
```

### NayaPay Purchase Flow
```
User → 2
Bot  → "Choose plan: 1=Plus 1mo / 2=Plus 12mo / 3=Go 12mo"
User → 1
Bot  → "How many?"
User → 2
Bot  → "Order: Plus 1mo × 2 = Rs. 5,600. Confirm? yes/no"
User → yes
Bot  → "✅ Purchase confirmed! Receipt sent to email."
```

---

## 🛠️ Admin Dashboard Pages

| Page | Description |
|---|---|
| **Dashboard** | Bot connection status, QR code, stats summary |
| **Customers** | All WhatsApp users who messaged the bot |
| **Activations** | CDK key activation history |
| **Purchases** | NayaPay purchase orders and revenue |
| **Bot Messages** | Customise all bot message templates |
| **Settings** | Gmail credentials for sending receipts |

---

## 📱 Multi-Tenant Architecture

Each tenant gets:
- Their own WhatsApp session (stored in `wa-auth/{tenantId}/`)
- Isolated database rows (all queries filtered by `tenant_id`)
- Independent bot message templates
- Their own Gmail credentials
- Separate customer, activation, and purchase records

No tenant can ever see another tenant's data.

---

## 🔧 Important Notes

- **WhatsApp sessions persist** across server restarts — tenants only re-scan QR if their session is explicitly deleted or they get logged out from WhatsApp
- **Conversation state is in-memory** — if the server restarts mid-conversation, the user will be asked to start over (they just send `*`)
- **Free WhatsApp** — uses Baileys (open source), not the official WhatsApp Business API, so no monthly fees
- **QR code** in the dashboard uses a free QR API (`api.qrserver.com`) to render the image

---

## 📦 Tech Stack

| Layer | Technology |
|---|---|
| Backend | Node.js + TypeScript + Express |
| WhatsApp | Baileys (`@whiskeysockets/baileys`) |
| Database | PostgreSQL via `node-postgres` |
| Frontend | React + TypeScript + Vite + Tailwind CSS |
| Auth | JWT (jsonwebtoken) |
| Email | Nodemailer + Gmail SMTP |
| Build | esbuild (backend) + Vite (frontend) |
| Package manager | pnpm workspaces |
| Hosting | Render.com |

---

## ❓ Troubleshooting

**QR code not showing?**
- Click "Connect WhatsApp" button on the Dashboard
- Wait 3-5 seconds for QR to generate
- If it still doesn't appear, click "New QR"

**Bot not responding?**
- Make sure the WhatsApp session shows "Connected" on the Dashboard
- Check Render logs for errors
- The bot only responds to direct messages (not groups)

**Activation failing?**
- The OpenAI session token expires quickly — get a fresh one from `chat.openai.com/api/auth/session`
- Make sure to copy the **full** JSON response

**Email receipts not sending?**
- Go to Settings and enter your Gmail address and App Password
- Regular Gmail password won't work — you need an App Password
- Instructions: [myaccount.google.com/apppasswords](https://myaccount.google.com/apppasswords)

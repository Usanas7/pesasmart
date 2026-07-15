# PesaSmart

**A USSD-based governance and transparency platform for informal Ikimina (rotating savings groups) in Rwanda.**

PesaSmart is **non-custodial** — it never holds, moves, or processes money. Members continue to contribute exactly as they always have (mobile money or cash). What PesaSmart provides is the missing *governance layer*: a shared, verifiable record that every member can check from any phone, in English or Kinyarwanda.

**Live demo:** https://pesasmart.vercel.app
**Demo video (5 min):** https://youtu.be/PKt4HbL8M58 

---

## The Problem

Ikimina are trust-based rotating savings groups. Money is typically tracked in a paper notebook held by one organiser. This creates three recurring governance failures:

1. **Unrecorded contributions** — a member pays, but it is never written down.
2. **Contested rotation order** — disagreement over whose turn it is to receive the payout.
3. **Unresolvable disputes** — no shared record, so disagreements become word-against-word.

The failure is not a *payment* problem. It is an **information asymmetry** problem: only the organiser can see the ledger.

## The Solution

PesaSmart puts the notebook on the table. Every member can dial a USSD code from **any phone** (no smartphone or internet required) and see the group's live state. Organisers manage the group through a web dashboard. Both views read from the same database, so members and organisers always see the same truth.

---

## Core Features

| # | Feature | How it works |
|---|---------|--------------|
| 1 | **Rotation support** | Groups are created with a contribution amount, frequency, cycle length and start date. Members are assigned rotation positions and scheduled payout dates. |
| 2 | **Dispute resolution** | A member raises a dispute over USSD, selects the issue type, and (for payment disputes) submits their MoMo transaction ID. The system issues a reference number (REF#0012) and notifies the organiser by SMS. |
| 3 | **Transparency** | Any member can view: their own status (position, amount owed, payout date), who has paid, the full rotation order with the current turn marked, and the number of open disputes. |
| 4 | **Membership changes** | Members request to exit or update their phone number over USSD. The organiser approves or rejects on the web, and the change is applied and broadcast to the group. |
| 5 | **SMS notifications** | Event-triggered SMS (dispute raised, dispute resolved, contribution confirmed, membership decision) plus organiser-initiated group broadcasts. |

**Bilingual:** the entire USSD interface is available in **English and Kinyarwanda**, selected on the first screen.

---

## Architecture

```
   Member (any phone)                    Organiser (browser)
          │                                      │
     USSD *384*6772#                    pesasmart.vercel.app
          │                                      │
   Africa's Talking                        React + MUI
          │                                      │
          └──────────► Node.js / Express ◄───────┘
                              │
                       PostgreSQL (Neon)
```

| Layer | Technology |
|-------|-----------|
| USSD & SMS gateway | Africa's Talking (sandbox) |
| Backend | Node.js, Express |
| Database | PostgreSQL (Neon) |
| Frontend | React (Vite), Material UI |
| Auth | JWT, bcrypt-hashed PINs |
| Deployment | Render (backend), Vercel (frontend) |
| CI | GitHub Actions |

---

## Repositories

| Repo | Purpose | Link |
|------|---------|------|
| `pesasmart` | This hub (overview + docs) | https://github.com/Usanas7/pesasmart |
| `pesasmart-backend` | Node/Express API, USSD handler, database | https://github.com/Usanas7/pesasmart-backend |
| `pesasmart-frontend` | React + MUI organiser dashboard | https://github.com/Usanas7/pesasmart-frontend |

---

## Try the Live Demo

### 1. Organiser web dashboard

Open **https://pesasmart.vercel.app**

- Create an account (phone number + 5-digit PIN), or sign in.
- Create a group, add members, mark contributions and payouts, resolve disputes, approve membership requests, export a CSV, and send SMS broadcasts.

### 2. Member USSD interface

USSD is served through the **Africa's Talking sandbox simulator**:

1. Go to https://developers.africastalking.com and open the **sandbox simulator**.
2. Enter a phone number that has been added as a member of a group (via the web dashboard).
3. Dial **`*384*6772#`**
4. Choose a language (1 = English, 2 = Kinyarwanda), then navigate the menu.

> **Note on live handsets:** the application code is production-ready, but running USSD on real phones requires a paid short-code agreement with a mobile network operator or aggregator. That is a commercial/regulatory step outside the scope of this project. The simulator exercises exactly the same backend.

---

## Running Locally

### Prerequisites

- Node.js 18+
- A PostgreSQL database (this project uses [Neon](https://neon.tech))
- An [Africa's Talking](https://africastalking.com) sandbox account

### 1. Backend

```bash
git clone https://github.com/Usanas7/pesasmart-backend.git
cd pesasmart-backend
npm install
```

Create a `.env` file in the project root:

```env
DATABASE_URL=postgresql://user:password@host/dbname?sslmode=require
AT_USERNAME=sandbox
AT_API_KEY=your_africastalking_api_key
JWT_SECRET=a_long_random_secret_string
PORT=3000
```

Start the server:

```bash
node index.js
```

The API runs at `http://localhost:3000`. Visit it in a browser — you should see `PesaSmart API is running`.

### 2. Database schema

Run the following in your PostgreSQL console to create the tables:

```sql
CREATE TABLE users (
  user_id SERIAL PRIMARY KEY,
  full_name VARCHAR(100) NOT NULL,
  phone_number VARCHAR(20) UNIQUE NOT NULL,
  pin VARCHAR(255),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE ikimina_groups (
  group_id SERIAL PRIMARY KEY,
  name VARCHAR(100) NOT NULL,
  contribution_amount NUMERIC NOT NULL,
  frequency VARCHAR(20) NOT NULL,
  cycle_length INT NOT NULL,
  start_date DATE,
  created_by INT REFERENCES users(user_id),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE ikimina_members (
  member_id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(user_id),
  group_id INT REFERENCES ikimina_groups(group_id),
  rotation_order INT,
  contribution_status VARCHAR(20) DEFAULT 'pending',
  payout_received BOOLEAN DEFAULT FALSE,
  status VARCHAR(20) DEFAULT 'active'
);

CREATE TABLE contribution_disputes (
  dispute_id SERIAL PRIMARY KEY,
  group_id INT REFERENCES ikimina_groups(group_id),
  member_id INT REFERENCES ikimina_members(member_id),
  disputed_week INT,
  momo_txid VARCHAR(50),
  dispute_type VARCHAR(30),
  status VARCHAR(20) DEFAULT 'open',
  raised_at TIMESTAMP DEFAULT NOW(),
  resolved_at TIMESTAMP
);

CREATE TABLE membership_changes (
  change_id SERIAL PRIMARY KEY,
  group_id INT REFERENCES ikimina_groups(group_id),
  affected_user INT REFERENCES users(user_id),
  change_type VARCHAR(20),
  status VARCHAR(20) DEFAULT 'pending',
  details VARCHAR(100),
  created_at TIMESTAMP DEFAULT NOW()
);

CREATE TABLE sms_notifications (
  notification_id SERIAL PRIMARY KEY,
  user_id INT REFERENCES users(user_id),
  message TEXT,
  status VARCHAR(20)
);
```

### 3. Frontend

```bash
git clone https://github.com/Usanas7/pesasmart-frontend.git
cd pesasmart-frontend
npm install
```

Create a `.env` file:

```env
VITE_API_URL=http://localhost:3000
```

Start the dev server:

```bash
npm run dev
```

Open `http://localhost:5173`.

### 4. Connect USSD

In your Africa's Talking sandbox dashboard, create a USSD channel and set the callback URL to:

```
https://<your-backend-url>/ussd
```

For local testing, expose your local server with a tunnel (e.g. `ngrok http 3000`) and use that URL.

---

## Testing

| Strategy | What was tested |
|----------|-----------------|
| **Functional testing** | Every USSD path and every organiser action was exercised end-to-end against the live database, in both English and Kinyarwanda. |
| **Input validation testing** | Forms and API endpoints were tested with invalid data: empty fields, non-numeric phone numbers, PINs of the wrong length, negative contribution amounts, and non-existent group codes. Each is rejected with a clear message rather than an error. |
| **Integration testing** | Cross-surface flows were verified: a dispute raised over USSD appears on the organiser's dashboard; an approval on the web is reflected in the member's next USSD session and triggers an SMS. |
| **Authorisation testing** | Protected API routes were called without a token and with an expired token; both are rejected with 401 and the client is returned to sign-in. |
| **Responsive / cross-device testing** | The dashboard was tested on desktop and mobile viewports; the layout collapses to a hamburger menu on small screens. USSD was tested through the Africa's Talking simulator. |

---

## Honest Limitations

These are deliberate scope boundaries:

- **Simulator, not live handsets.** Live USSD requires a paid mobile-operator short code — a commercial step beyond this project's scope. The code is production-ready.
- **Transaction IDs are recorded, not verified.** Because PesaSmart is non-custodial, it cannot independently confirm a mobile-money payment. It records the member's evidence and surfaces it to the organiser; the human resolution process remains with the group. This is stated to the member on screen.
- **USSD session timeouts** are imposed by the mobile network (typically 90–180 seconds), not by the application.
- **Rotation is not renumbered after a mid-cycle exit.** This is intentional: renumbering would silently change other members' scheduled payout dates without their consent, and a mid-cycle exit involves a real financial settlement between people that software should not paper over. PesaSmart's role is to record the exit, notify the group, and make the obligation undeniable.
- **One group per member** in the current USSD flow. Multi-group membership is identified as future work.

## Future Work

- Automatic escalation SMS for disputes unresolved after 48 hours
- Live mobile-operator USSD integration

---

## Author

**Christelle Usanase**
BSc Software Engineering, African Leadership University
Supervisor: Bernard Odartei Lamptey

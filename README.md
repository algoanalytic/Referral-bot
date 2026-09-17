# Telegram Referral Bot with SQLite Storage

Production-ready Telegram Referral Bot built with Python, [`python-telegram-bot`](https://github.com/python-telegram-bot/python-telegram-bot) (async v20+), and asynchronous SQLite ([`aiosqlite`](https://github.com/omnilib/aiosqlite)).

---

## 🌟 Key Features

1. **Relational Database Architecture (`database.py`)**:
   - **`users` Table**: Ties one individual person to all their referrals (solves the multi-USDOT carrier issue).
   - **`referrals` Table**: Full status lifecycle (`Submitted` → `SM Got It` → `Joined` → `Pending` → `Got Bonus`).
   - **`bonuses` Table**: Decoupled payout tracking with automatic `paid_at` timestamps for cash and service credits.
   - **Automatic Migrations**: Safe, idempotent startup checks that auto-alter existing databases.

2. **One-Time Registration Gating**:
   - Gated access for "Refer Someone" and "My Balance".
   - Collects verified Full Name, US Phone Number (validated with friendly re-prompts), and USDOT Number (skippable).
   - Seamless transition: once registered, immediately continues to the user's intended action without extra clicks.

3. **Telegram Deep-Link Attribution**:
   - Every user receives a unique referral code: `ref_<telegram_id>`.
   - Links: `https://t.me/<bot_username>?start=ref_<telegram_id>`.
   - Automatic attribution: if an invited friend opens the bot and later submits referrals, the channel post includes the full attribution chain.

4. **PTB `allow_reentry=True` & Smooth Flow**:
   - After referral submission, users can immediately tap **[👥 Refer Another]** or **[📊 My Balance]** without conversation lockup.

5. **Authoritative DB Writes & Best-Effort Channel Posting**:
   - Referral data is saved to SQLite first.
   - Channel post to `TELEGRAM_CHANNEL_ID` is best-effort: if channel permissions or bot setup is missing, the submission is still safely preserved in SQLite.

---

## 🚀 Quick Start Guide

### 1. Open Workspace
Open the project directory in Antigravity IDE or your terminal:
```powershell
cd C:\Users\User\.gemini\antigravity-ide\scratch\telegram-referral-bot
```

### 2. Set Up Virtual Environment & Dependencies
```powershell
python -m venv .venv
.\.venv\Scripts\Activate.ps1
pip install -r requirements.txt
```

### 3. Configure Environment Variables
Copy `.env.example` to `.env`:
```powershell
Copy-Item .env.example .env
```
Open `.env` and fill in:
- `TELEGRAM_BOT_TOKEN`: From [@BotFather](https://t.me/BotFather)
- `TELEGRAM_CHANNEL_ID`: Channel ID (e.g. `-100xxxxxxxxxx` or `@your_channel`) where the bot is an Administrator
- `DB_PATH`: `data/bot.db` (default)

### 4. Run the Test Suite
Verify that the database layer, migrations, and bonus logic work:
```powershell
python test_db.py
```

### 5. Launch the Bot
```powershell
python bot.py
# Or using the virtual environment:
.\.venv\Scripts\python.exe bot.py
```

---

## 📊 Referral Lifecycle Reference

```
[Submitted] ────────▶ [SM Got It] ────────▶ [Joined] ────────▶ [Pending] ────────▶ [Got Bonus]
   (New)            (Manager called)       (Onboarded)       (Payout ready)       (Paid out)
```

To update referral statuses or issue bonuses programmatically from admin scripts or an admin dashboard, use the helpers in `database.py`:
- `await update_referral_status(referral_id, "SM Got It")`
- `await update_referral_status(referral_id, "Joined")`
- `await create_bonus(referral_id, referrer_id, amount=250.0, payout_type="cash")`
- `await update_bonus_status(bonus_id, "Paid")`


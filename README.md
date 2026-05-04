# SNAM Auction Bot

A lightweight Python bot that monitors [SNAM's public Jarvis portal](https://jarvis.snam.it) for new **within-day (infragiornaliera) storage capacity auction results** and delivers instant Telegram alerts when fresh data is detected.

---

## How it works

```
┌─────────────────────┐      ┌──────────────────────┐      ┌─────────────────┐
│  SNAM Jarvis Portal │─────▶│  Headless Chrome      │─────▶│  Excel Report   │
│  (public data)      │      │  (Selenium download)  │      │  (.xlsx)        │
└─────────────────────┘      └──────────────────────┘      └────────┬────────┘
                                                                     │
                                                              pandas parse
                                                                     │
                                                            ┌────────▼────────┐
                                                            │ Signature check  │
                                                            │ (snam_last_sig)  │
                                                            └────────┬────────┘
                                                           changed?  │
                                                                ┌────▼────┐
                                                                │Telegram │
                                                                │ Alert   │
                                                                └─────────┘
```

1. **Download** — A headless Chrome session navigates to the portal and clicks the download button inside the shadow DOM of the `<ds-button>` component.
2. **Parse** — The Excel sheet is scanned for the real header row and the most recent auction timestamp is extracted.
3. **Deduplicate** — A signature `timestamp_rows_offered_requested` is compared against the last persisted signature. No alert is sent if data is unchanged.
4. **Notify** — A Markdown-formatted Telegram message is sent listing each auction row (injection/withdrawal, capacity type, price, quantities).

---

## Sample Telegram alert

```
📊 SNAM AUCTION RESULTS
🕐 31/03/2026 23:00

--- WITHDRAWAL / WITHINDAY ---
Type:      FIRM
Price:     0.2700 €cent/kWh
Offer:     24.157.178 kWh/d
Request:   9.600.000 kWh/d
Allocated: 4.800.000 kWh/d
```

---

## Requirements

| Dependency | Purpose |
|---|---|
| `selenium` | Headless browser automation |
| `webdriver-manager` | Auto-manages ChromeDriver binaries |
| `pandas` + `openpyxl` | Excel parsing |
| `requests` | Telegram API calls |
| `python-dotenv` | Environment variable loading |
| `truststore` | System CA store for SSL verification |

Install all dependencies:

```bash
pip install -r requirements.txt
```

> **Chrome required** — a compatible version of Google Chrome (or Chromium) must be installed. `webdriver-manager` handles the matching ChromeDriver automatically.

---

## Setup

### 1. Clone the repository

```bash
git clone https://github.com/your-username/snam-auction-bot.git
cd snam-auction-bot
```

### 2. Install dependencies

```bash
pip install -r requirements.txt
```

### 3. Configure environment variables

Copy the example file and fill in your values:

```bash
cp .env.example .env
```

```ini
# .env
TELEGRAM_TOKEN=your_bot_token_here
TELEGRAM_CHAT_ID=your_chat_id_here
```

- **TELEGRAM_TOKEN** — obtain from [@BotFather](https://t.me/BotFather) on Telegram.
- **TELEGRAM_CHAT_ID** — your user, group, or channel ID. Use [@userinfobot](https://t.me/userinfobot) to find it.

### 4. Run once

```bash
python snam_bot.py
```

---

## Scheduled execution

The bot is designed for **single-run execution** (no internal loop). Schedule it externally to match the portal's update cadence.

**Linux / macOS — cron** (every 10 minutes):
```cron
*/10 * * * * /usr/bin/python3 /path/to/snam_bot.py >> /path/to/bot_logs/cron.log 2>&1
```

**Windows — Task Scheduler:**
- Trigger: repeat every 10 minutes
- Action: `python C:\path\to\snam_bot.py`

---

## Project structure

```
snam-auction-bot/
├── snam_bot.py            # Main bot script
├── snam_latest_data.xlsx  # Most recent downloaded report (git-ignored)
├── snam_last_sig.txt      # Persisted deduplication signature (git-ignored)
├── bot_logs/              # Rotating log files (git-ignored)
├── .env                   # Secrets — never commit this (git-ignored)
├── .env.example           # Template for environment variables
├── requirements.txt       # Python dependencies
└── .gitignore
```

---

## Data source

Auction data is sourced from SNAM's public Jarvis portal:

> **Capacità di Stoccaggio — Esiti Aste Short Term**  
> Thermal year: 2025/2026  
> Product: Infragiornaliera (within-day)  
> https://jarvis.snam.it

To track a different thermal year or product, update the `PORTAL_URL` constant in `snam_bot.py`.

---

## Logging

Logs are written to `bot_logs/snam_bot.log` with rotation (5 MB per file, 3 backups). Console output mirrors the same messages.

---

## Disclaimer

This project is an independent tool and is not affiliated with or endorsed by SNAM S.p.A. It accesses only publicly available data. Use responsibly and in accordance with SNAM's terms of service.

# CyberCoinvvs BNB Giveaway â€” Render deployment package

This package contains the responsive Telegram Mini App website, Express API, PostgreSQL schema/bootstrap, Telegram bot webhook, admin CSV export, and post-close random draw endpoint.

## Important launch blockers

The official rules and privacy notice are drafts, not legal advice. Before public launch, complete the organizer legal identity/address, applicable law, minimum age, winner claim window, post-giveaway retention/deletion date, and verify the token contract's actual token name, symbol, decimals, balance, transfer restrictions, and prize denomination on BNB Smart Chain.

The published prize is currently worded as **100,000 units of an ERC-20 token with displayed symbol BNB** at `0x854A21266A02cd2f4c31D3FB6f8A2271CF9DE8d8`. Do not describe it as 100,000 native BNB unless that is actually what is funded. Verify the contract on a reputable BNB Smart Chain explorer before launch.

The application does not custody funds, request seed phrases/private keys, connect wallets, sign transactions, or transfer tokens. Prize delivery is a separate organizer action.

## Deploy on Render

1. Create a private GitHub repository and upload the contents of this folder.
2. In Render, choose **New â†’ Blueprint** and select the repository. Render reads `render.yaml` and provisions the web service and PostgreSQL database.
3. In the Render service environment, set:
   - `TELEGRAM_BOT_TOKEN`: the token from BotFather (secret; never put it in HTML or Git).
   - `TELEGRAM_BOT_USERNAME`: bot username without `@`.
   - `APP_ORIGIN`: exact public Render URL, e.g. `https://your-service.onrender.com`.
   - Confirm `DATABASE_URL`, `ADMIN_TOKEN`, and `TELEGRAM_WEBHOOK_SECRET` were created from the Blueprint. Keep both generated secrets private.
4. Deploy and check `https://YOUR-SERVICE/health`.
5. Configure the bot webhook from a trusted terminal, replacing the placeholders. Use the same webhook secret as Render:
   ```bash
   curl -X POST "https://api.telegram.org/bot<YOUR_BOT_TOKEN>/setWebhook" \
     -d "url=https://YOUR-SERVICE.onrender.com/telegram/webhook" \
     -d "secret_token=<YOUR_TELEGRAM_WEBHOOK_SECRET>"
   ```
   Never paste the bot token into a public chat, source file, GitHub issue, or screenshot.
6. In BotFather, set the Mini App menu button to your deployed URL (Bot Settings â†’ Menu Button, or use `/setmenubutton`). Set the URL to `https://YOUR-SERVICE.onrender.com`.
7. Test `/start`, `/help`, `/rules`, then open the Mini App from Telegram and submit a test entry. The server rejects entries without valid Telegram-signed `initData`.
8. Before public launch, remove test entries from the database using an authenticated database session, and verify the prize/official rules.

## API endpoints

- `GET /health` â€” health check.
- `GET /api/config` â€” public giveaway configuration.
- `POST /api/entries` â€” entry submission; validates Telegram Mini App `initData` server-side and enforces one entry per Telegram user ID.
- `POST /telegram/webhook` â€” Telegram bot webhook, authenticated with Telegram's secret header.
- `GET /admin/entries.csv` â€” admin CSV export; requires `Authorization: Bearer <ADMIN_TOKEN>`.
- `POST /admin/draw` â€” one-time random selection endpoint, disabled until after the published close time; requires the same admin bearer token.
- `GET /rules` and `GET /privacy` â€” draft legal pages.

## Security notes

- Telegram Bot token, admin token, and webhook secret are server-side environment variables only.
- Do not expose `/admin` endpoints or tokens in the front end.
- Telegram `initDataUnsafe` is used only to prefill a display field; server authorization relies on the raw `initData` signature.
- Database uniqueness on `telegram_user_id` prevents duplicate entries even under concurrent submissions.
- Rate limiting and request-size limits are enabled.
- For production, keep Render access restricted, enable MFA on Render/GitHub, rotate secrets if exposed, and take database backups.
- The built-in retention job deletes entry records at `RETENTION_END_AT` only after a winner has been selected. Set this to the actual disclosed deletion date; do not schedule deletion before winner verification/delivery is complete.

## Local run

Requires Node.js 20+ and PostgreSQL. Copy `.env.example` to `.env`, fill values, then:
```bash
npm install

npm start
```

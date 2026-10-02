# Trusted BAZAAR Telegram SMM Bot

## Required Render Environment Variables
- BOT_TOKEN
- SMM_API_KEY
- ADMIN_ID
- DATABASE_URL
- SMM_API_URL=https://my.smmsun.com/api/v2
- USD_TO_BDT (optional initial rate, for example `122.50`; the owner can set it from Admin Panel)

Start command: `node bot.js`

## Service prices in BDT and USD
Set the exchange rate in Owner Admin Panel → `💱 USD → BDT`. It is stored in PostgreSQL and takes precedence over the optional initial `USD_TO_BDT` environment variable. Customer service lists and order selection show both ৳ and $ per 1,000. `Set Price`, `Increase Price`, and `Decrease Price` accept BDT per 1,000; the USD figure is an approximate display conversion. If a service has no custom price, its provider USD rate is converted to BDT using this rate. Orders and customer balances are charged in BDT. Set a rate before customers browse services or order.

## Render Free web service and Telegram updates
This version registers a Telegram HTTPS webhook instead of long polling. Render supplies `RENDER_EXTERNAL_URL` automatically for a Web Service. If hosting elsewhere, set `PUBLIC_URL=https://your-public-domain.example`; the URL must be publicly reachable over HTTPS. Keep one deployed bot instance for each bot token. Do not configure a second webhook or polling worker for the same bot. The webhook uses Telegram's secret-token header.

Render Free web services spin down after idle time. A Telegram message sends an incoming webhook request that wakes the service, but the first reply after sleep can be delayed while Render starts it. Check Render logs for `Telegram webhook registered successfully.` and `Startup failed` if messages still fail. PostgreSQL remains the store for balances and orders. Render Free PostgreSQL expires after 30 days unless upgraded.

## IMPORTANT: Persistent data
Customer balances, deposits, orders, services, prices, payment numbers, users, and referral data are stored in PostgreSQL (`DATABASE_URL`). The bot now refuses to start if `DATABASE_URL` is missing, so a redeploy cannot silently start with empty temporary data.

Keep the **same PostgreSQL DATABASE_URL** across every deploy. Do not create a new database or replace the URL when redeploying.

The bot loads the saved database before accepting Telegram updates and preserves existing user/referral fields when a user sends a message.

## Order behavior
Orders are submitted server-side to the configured SMM API using `SMM_API_KEY`. The customer is not redirected to the provider website.

## Customer language and support
- On first use, each customer chooses বাংলা or English. They can switch later from the `🌐 Language / ভাষা` button; the choice is saved with their profile.
- The Customer Panel has a `🆘 Support` button. The owner can manage agents from Admin Panel → `🆘 Support Agents`.
- Add an agent as `@telegram_username | Display Name`; customers can tap an agent button to open Telegram chat.

## Customer service categories
- Customer Panel → `📋 Services` and `🛒 New Order` first show platform categories such as Facebook, TikTok, Instagram, YouTube, Telegram, and other supported platforms.
- Services are grouped from the provider's category/name. Unrecognized platforms appear under `Other Services`.
- Choosing a category opens only the services assigned to that platform; `New Order` then lets the customer choose a service from that category.

Do not put BOT_TOKEN or SMM_API_KEY in source code. Keep them in Render Environment Variables.

## Customer account & admin customer history
- Customer Panel → `👤 Account Details` shows User ID, username, name, current balance, total orders, total order value, completed order count, last seen, and joined time when available.
- Admin Panel → `👤 Customer Details` lists customers and lets the admin open an individual customer profile.
- Each customer profile includes order history with Order ID, service name/ID, quantity, cost, status, provider order ID, link, and order time.
- New orders also save the service name so future admin history is easier to read.

## Admin and manager access
- Owner Admin Panel → `➖ Customer Balance`: enter `CustomerID Amount`, review the current and resulting BDT balance, then confirm. Example: `123456789 20` removes ৳20 of an accidental over-credit. The bot refuses unknown customers, zero/invalid amounts and deductions larger than the current balance. Only the owner can confirm; a correction record with owner ID, before/after balances, amount and timestamp is stored in PostgreSQL. The customer receives a correction notice. This changes the wallet balance only; it does not rewrite the original approved deposit request or manager payment summary.
- Set `ADMIN_ID` to the owner's numeric Telegram User ID. The owner opens the panel with `/admin` and sees `👥 Manage Managers`.
- Use `➕ Add Manager`, enter a manager's numeric Telegram User ID, and ask them to open `/admin`. They can get their ID with `/myid`.
- Each manager has separate bKash, Nagad, and Binance numbers; the owner has a separate set. Customers see which account belongs to the owner or a manager when adding balance.
- A payment request goes only to the owner of the selected account. Only that owner/manager can approve or reject it; manager requests cannot be approved by another manager or by the owner.
- Managers cannot access service/pricing controls, customer history, or manager controls. The owner can view approved and pending payment totals for each manager under `💰 Manager Payment Summary`.
- Manager accounts, their payment numbers, and payment records are stored in PostgreSQL. Removing a manager stops access; historical collection totals remain in the report.


## Added: Hindi and manager special prices
- Customer language picker now includes বাংলা, English, हिन्दी.
- Owner: Admin Panel → ⭐ Set Manager Price. Enter `ServiceID Price` in BDT per 1,000, e.g. `979 100`. Remove a special price with `979 remove`. Add the service to the regular service ID list first.
- All active managers share the special price list. Only managers see ⭐ Manager Specials and can open/order from it. Owner controls prices but does not get manager ordering access.
- Only services with an explicit special price appear in this list. Regular customer service prices are unchanged.
- Special orders use the manager's existing BDT wallet and existing order/status workflow. Access and current price are checked again before submission.
- Manager prices and Hindi language choices persist in the existing PostgreSQL database. Existing records are preserved. Keep the same DATABASE_URL.

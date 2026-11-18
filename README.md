# APEX SIGNAL AI — Chart/API Test Build

This build preserves the supplied APEX SIGNAL AI interface and adds a separate **Chart & API Diagnostics** section.

## What is included

- **Open Chart / API Test** button opens a separate chart section; Back returns to the main dashboard.
- **Connect TradoWix API** uses the supplied PyTradoWix client.
- Credentials are read only from server-side environment variables (`TRADOWIX_EMAIL`, `TRADOWIX_PASSWORD`). The browser never receives them.
- Live and OTC instrument lists are loaded dynamically from the API's instrument metadata (`isOtc`).
- Pair selection is populated from the provider's actual instrument list.
- 30-second and 1-minute historical candle requests, with a maximum request of 1,000 candles.
- Candlestick chart and live tick polling; new incoming ticks update the current candle on the chart.
- API authentication, instrument count, candle count, and latest tick status are displayed separately.
- **No order-placement route or automatic trade execution is exposed by this web app.**

## Local run (only for a developer; not required on your phone)

Python 3.11 or newer is required by the supplied SDK.

```bash
pip install -r requirements.txt
TRADOWIX_EMAIL='your-account-email' TRADOWIX_PASSWORD='your-new-password' uvicorn app:app --host 0.0.0.0 --port 8000
```

Then open `http://localhost:8000`.

## Cloud deployment

1. Upload the contents of this folder to a **private** GitHub repository.
2. Create a new Web Service on Render and connect that repository.
3. Runtime: Python. Build command: `pip install -r requirements.txt`.
4. Start command: `uvicorn app:app --host 0.0.0.0 --port $PORT`.
5. In Render's Environment settings, add `TRADOWIX_EMAIL` and `TRADOWIX_PASSWORD` as private environment variables. Do not put real credentials in GitHub, this ZIP, chat, or frontend code.
6. Deploy, open the service URL, click **OPEN CHART / API TEST**, then **CONNECT TRADOWIX API**.
7. If authentication succeeds, choose LIVE or OTC, select an API-listed pair, choose 30s or 1m, and click **LOAD 1,000 CANDLES**.

## Important limits

- The chart's Connect button performs a real login attempt only after deployment with valid private environment variables. This package cannot verify account authentication from this development environment.
- OTC/live classification and pair availability depend on the `isOtc`, active/open flags, and instrument list actually returned by TradoWix.
- The provider may return fewer than 1,000 candles. The chart reports the exact number received rather than pretending all 1,000 arrived.
- A live tick subscription is requested after history is fetched. Whether the provider supplies ticks for a selected symbol must be confirmed in the deployed chart diagnostics.
- The chart is a lightweight in-app OHLC canvas; it does not use a third-party chart CDN.
- The existing main-screen signal button remains a demo placeholder at this stage. Do not treat its random sample output as a real signal. The real-data analysis engine is a later integration step.
- No win-rate or prediction accuracy is claimed until historical testing is implemented and validated.

## API endpoints

- `GET /api/health` — service health
- `POST /api/connect` — authenticate using server environment variables
- `GET /api/status` — authentication and instrument status
- `GET /api/assets?market=live|otc|all` — API instrument list
- `GET /api/chart?symbol=...&timeframe=30|60&limit=1000` — historical candles and tick subscription
- `GET /api/ticks/{symbol}` — latest tick for the selected symbol
- `POST /api/disconnect` — close the current API session

This application intentionally has no trade execution endpoint.

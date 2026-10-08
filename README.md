# VayuDrishti-SerpApi

VayuDrishti's entry for the SerpApi India Hackathon 2026 by Anubhav Chakraborty. A weather dashboard with India-scoped learned rainfall correction and location-based weather news.

## What existed before the hackathon

The VayuDrishti weather dashboard and rainfall models pre-existed this entry. This repository copies the complete runnable application from `vayudrishti-arch/vayudrishti-arch.github.io`. The hackathon addition is **Live alerts & news**, described below. Weather forecasts come from Open-Meteo/GFS, not SerpApi. SerpApi does not train or correct the rainfall model.

## Live alerts & news (SerpApi)

After a named place is loaded, the app requests recent weather-related reporting for that place. SerpApi's **Google News API** (`engine=google_news`, `gl=in`, `hl=en`) searches the place and country with weather, alert, advisory, warning, rain and flood terms, restricted to the past 14 days. Results can include regional stories, not exclusively city-specific reporting.

The card shows up to eight linked headlines, with publisher and date. News reports add context to the forecast, but they are **not official warnings**. For safety decisions use IMD or local authorities. VayuDrishti issues no automated warning or all-clear.

### Request flow and key security

1. The browser sends the selected place name and country to the Cloudflare Worker in `serpapi-proxy/worker.js`.
2. The Worker builds the search query and calls `https://serpapi.com/search.json`.
3. `SERPAPI_KEY` is a Cloudflare secret. It is never embedded in the frontend or committed to this repository.
4. The Worker returns only the selected result fields: title, source, date, link and snippet. The current UI displays headline, source and date.

Results are cached per place/country/day for about six hours. When the `COUNTERS` KV binding is configured, the Worker limits searches to 230 per month and 20 uncached requests per IP per hour. These are usage safeguards, not guaranteed accounting under concurrent requests. CORS accepts the original Pages origin; localhost is accepted when `ALLOW_LOCAL=1` is configured. If news is unavailable, empty, over quota or too slow, the card stays hidden and the forecast still works.

## Run locally on Windows

Download this whole repository with **Code > Download ZIP** and extract it, or clone it with Git. Open Command Prompt in the folder containing `app` and `README.md`.

```bat
py -m http.server 8000 --bind 127.0.0.1
```

If `py` is unavailable but Python is installed, use `python` instead. No pip packages or Node installation are needed to run the static frontend. Keep the server window open, then visit:

```text
http://localhost:8000/app/
```

Internet access is needed for weather/geocoding requests and the deployed news proxy. No SerpApi key is needed to try this frontend because it points to the existing deployed Worker, subject to availability and quota. Search Pune, choose **Pune, Maharashtra, India**, then click **Load forecast**. Check that **Live alerts & news** appears. If a first request times out, wait and try Load forecast once more.

The rainfall models are embedded in `app/index.html`. `app/accuracy.json` contains historical evaluation data. `app/lead1-model.json` is an optional promoted-model override; its absence is expected. Historical charts replay archived forecasts, not predictions published live on those dates.

## Deploy your own news proxy

Use `serpapi-proxy/worker.js` and its `wrangler.toml`. Create a Cloudflare Worker and a KV namespace bound as `COUNTERS`. Store your own key with `wrangler secret put SERPAPI_KEY`, never in a source file. Configure `ALLOW_LOCAL=1` for local testing; add your own production origin to `ORIGINS`. Change the Worker's name as needed and point `NEWS_PROXY` in `app/index.html` to your deployed Worker. Use your own SerpApi account and respect its search limits.

The existing deployed Worker is named `vayudrishti-serpapi`. This repository does not include its secrets, account credentials or production KV identifiers.

## Data, privacy and limitations

Forecast and geocoding requests go directly to Open-Meteo. The selected place name and country are sent to the Worker and SerpApi for news. No device location is requested and forecasts are not automatically polled. Theme preference is stored locally.

Weather data: Open-Meteo, CC BY 4.0. Location data: GeoNames via Open-Meteo. Learned rainfall correction is limited to the Indian training scope. ERA5 is model-based reanalysis, not local rain-gauge truth. The app is a research product, not an official weather service. A code license does not replace source-data terms.

The copied nightly workflow is separate research maintenance, not needed for the demo. Automated promotion is off in the source snapshot.

## Build disclosure

This project was built with AI assistance for code, integration and documentation. The existing weather app and the SerpApi addition are distinguished above; the entry does not claim that all weather-model work was created during this hackathon.

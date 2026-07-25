# Aakash (आकाश) — The Sky Is the Interface

A premium animated weather app for Android where the **entire screen atmosphere changes with live conditions** — rain streaks fall during rain, snow drifts on cold days, the sun sets into a golden dusk in real time, and a crescent moon rises at night. Built with Jetpack Compose, powered by Open-Meteo, designed end-to-end from research to shipped code.

> **Why a weather app?** Weather is the most saturated category on the Play Store — which is exactly why I picked it. The goal wasn't market share; it was proving I could match top-tier product polish (Apple Weather, CARROT) with AI-assisted execution, solo, in days. This repo documents the whole pipeline: market research → design system → phased build spec → code review loop → shipped v1.1.

---

## ✨ What it does

- **Live atmosphere engine** — a single `WeatherKind` enum (mapped from WMO condition codes) drives the background gradient, particle system, and hero scene. 8 weather states × day/dusk/night phases, crossfading in 1200ms.
- **Time-correct skies** — a local `DayPhase` state machine (DAWN / DAY / DUSK / NIGHT) computed from sunrise/sunset means the sky transitions to golden hour and night **in real time while the app is open, without a network call**.
- **Air quality as a first-class citizen** — a full-width AQI card sits *above* the standard stats. In Delhi NCR, air quality determines your day more than temperature does; the app reflects that priority.
- **Live precipitation radar** — MapLibre + Carto Dark Matter dark cartography with RainViewer tiles: animated past-2h playback with a timeline scrubber, an Apple-style temperature-bubble location marker anchored to the map, intensity legend, and an Open-Meteo minutely-15 "next 2 hours" nowcast strip.
- **7-day forecast** with expandable day cards, sunrise/sunset, and precipitation probability.
- **Offline resilience** — last successful response cached; airplane-mode launch shows data with an "Offline" banner.
- **Haptics everywhere** (user-toggleable), pull-to-refresh, animated unit switching, themed splash.

## 🌧️ The Atmosphere Engine (the differentiator)

All weather effects are **hand-rolled Compose Canvas particle systems** — no Lottie, no GIFs, no video:

| Condition | Effect |
|---|---|
| Rain / Drizzle / Storm | 90–280 pre-allocated drops, delta-time physics via `withFrameNanos`, zero per-frame allocations |
| Storm | + double-flash lightning overlay on a random 3–8s interval |
| Snow | 140 flakes with sine-wave drift |
| Clear day | Radial-gradient sun with 12 rotating rays (60s period) |
| Clear night | Crescent moon + 20 twinkling stars |
| Dusk / Dawn | Warm gradient wash, sun rendered low at 78% screen height |
| Fog / Overcast | Drifting translucent bands |

## 🏗️ Architecture

```
UI (Compose + Material 3)
  └── ViewModels (StateFlow + sealed UiState: Loading / Success / Error)
        └── Repositories (Weather, Location) — parallel fetches via async/awaitAll
              └── Retrofit + Gson → Open-Meteo (forecast, geocoding, air quality)
              └── DataStore (settings, saved cities, offline cache)
```

**Deliberate choices:** manual DI via `AppContainer` (no Hilt — simpler, more debuggable generated code), Canvas over Lottie (no asset pipeline, full control), Open-Meteo over OpenWeatherMap (no API key, WMO codes map 1:1 to the theming enum, CC BY 4.0).

## 🛠️ How it was built

1. **Research** — competitive scan of CARROT, Apple Weather, (Not Boring) Weather, Overdrop; framework bake-off (Flutter vs KMP vs native).
2. **Design** — full 5-screen design system in Google Stitch (indigo `#0B1026 → #1B2A52`, glassmorphism, 96sp ultra-light numerals), iterated through a senior-review loop that produced a 4-variant "one layout, four atmospheres" hero system.
3. **Build** — Google AI Studio, driven by a Master Prompt contract (pinned stack, quality bars, phased delivery) + 4 phase prompts.
4. **Review** — every build zip code-reviewed: caught a dependency-bloat P0 (Firebase/Room/Moshi boilerplate), a WMO-mapping P1 (merged cloud codes), and a real user-reported bug (noon sun at 7 PM → root cause: binary `is_day` + stale fetches → fix: the `DayPhase` state machine).

## 🗺️ Roadmap

- **Next:** GPS auto-location with graceful permission handling; notification rain alerts.
- **Deliberately cut so far:** forecast radar on the map (free radar tiles are past-only — the nowcast strip covers the forward-looking need with data instead), widgets, ambient sounds, Wear OS. A prioritized not-doing list is a feature.

## 📱 Running it

Open in Android Studio (Ladybug+), sync, run. No API keys required — all data sources are free tiers.

- Weather, geocoding & air quality: [Open-Meteo](https://open-meteo.com/) (CC BY 4.0)
- Radar: [RainViewer](https://www.rainviewer.com/) · Map © CARTO © OpenStreetMap contributors

---

**Anubhav Kumar** — Product & Growth Associate → APM. Part of a portfolio of shipped product experiments: [Khyaal](https://github.com/anubhavk42/khyaal) · [SAAR](https://github.com/anubhavk42/saar) · [NutriLens AI](https://github.com/anubhavk42/nutrilens-ai) · [Disha](https://github.com/anubhavk42/disha-career-app)

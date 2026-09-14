# Aakash (आकाश) — The Sky Is the Interface

A premium animated weather app for Android where the **entire screen atmosphere changes with live conditions** — rain streaks fall during rain, snow drifts on cold days, the sun sets into a golden dusk in real time, and a crescent moon rises at night. Built with Jetpack Compose, powered by Open-Meteo, designed end-to-end from research to shipped code.

**▶️ [Try the live demo](https://appetize.io/app/b_gfn567ycsmtelj6zzl7tp7pnzq)** — run the app in your browser, no install needed.

> **Why a weather app?** Weather is the most saturated category on the Play Store — which is exactly why I picked it. The goal wasn't market share; it was proving I could match top-tier product polish (Apple Weather, CARROT) with AI-assisted execution, solo, in days.

## The problem

Most weather apps are functionally identical — the same seven-day forecast card in a slightly different font. Nobody opens a weather app expecting to feel anything; it's a glance-and-close utility. Meanwhile, for a lot of Indian users — especially in Delhi NCR — air quality affects daily life more directly than temperature does, and most global weather apps still treat AQI as an afterthought buried below the fold, if it's shown at all.

## Who it's for

- **Primary:** People who want checking the weather to feel like looking outside, not reading a spreadsheet — and who live somewhere AQI is a daily, practical concern.
- **Secondary:** Anyone comparing weather apps on pure design/polish, since this is built to match the visual bar of Apple Weather and CARROT.
- **Explicitly not for:** Users who need enterprise-grade meteorological precision, hyperlocal severe-storm alerting, or professional forecasting tools — this is a consumer daily-glance app, not a weather API product.

## The key decision

Two bets, stacked: **hand-rolled visual atmosphere (not a template) is what makes a commodity category feel worth opening daily** — and **AQI deserves equal billing with temperature**, not a secondary stat, because for the target user it often matters more day-to-day.

## The trade-off

The hard choice was building every weather effect as a **hand-rolled Compose Canvas particle system** instead of using Lottie or pre-made animation assets. The alternative rejected: drop in existing animated weather asset packs, which would have shipped faster with far less risk.

What that cost: significantly more build time and code to maintain, and real bugs that only show up with custom state machines — a noon-sun-at-7-PM bug traced back to a binary `is_day` flag plus stale fetches, fixed by building a dedicated `DayPhase` state machine. The payoff was full creative control over 8 weather states × 3 day-phases with zero asset pipeline — nothing pre-made could have delivered "the sky visibly changing in real time while the app is open."

A second trade-off: Open-Meteo over OpenWeatherMap — no API key required and WMO codes map 1:1 to the theming enum, at the cost of the broader enterprise support and coverage a paid provider offers.

## What's in v1

- **Live atmosphere engine** — a single `WeatherKind` enum (mapped from WMO condition codes) drives the background gradient, particle system, and hero scene. 8 weather states × day/dusk/night phases, crossfading in 1200ms.
- **Time-correct skies** — a local `DayPhase` state machine (DAWN / DAY / DUSK / NIGHT) computed from sunrise/sunset means the sky transitions to golden hour and night in real time while the app is open, without a network call.
- **Air quality as a first-class citizen** — a full-width AQI card sits above the standard stats.
- **Live precipitation radar** — MapLibre + Carto Dark Matter dark cartography with RainViewer tiles: animated past-2h playback with a timeline scrubber, an Apple-style temperature-bubble location marker, intensity legend, and an Open-Meteo minutely-15 "next 2 hours" nowcast strip.
- **7-day forecast** with expandable day cards, sunrise/sunset, and precipitation probability.
- **Offline resilience** — last successful response cached; airplane-mode launch shows data with an "Offline" banner.
- Haptics everywhere (user-toggleable), pull-to-refresh, animated unit switching, themed splash.

## What's deliberately not in v1

- **Forward-looking radar on the map** — free radar tiles are past-only; the nowcast strip covers the forward-looking need with real data instead of building a paid-tier dependency for it.
- **Home-screen widgets** — cut to keep v1 scoped to the in-app experience; a natural v2 candidate once the core engine is proven.
- **Ambient sounds** — considered, cut as a nice-to-have that doesn't change the core value proposition.
- **Wear OS support** — out of scope for a solo v1 build; would need its own design pass, not a port.
- **GPS auto-location** — not yet built; currently relies on manual city search/selection, with graceful auto-location handling planned next.

## The Atmosphere Engine (the differentiator)

All weather effects are hand-rolled Compose Canvas particle systems — no Lottie, no GIFs, no video:

| Condition | Effect |
|---|---|
| Rain / Drizzle / Storm | 90–280 pre-allocated drops, delta-time physics via `withFrameNanos`, zero per-frame allocations |
| Storm | + double-flash lightning overlay on a random 3–8s interval |
| Snow | 140 flakes with sine-wave drift |
| Clear day | Radial-gradient sun with 12 rotating rays (60s period) |
| Clear night | Crescent moon + 20 twinkling stars |
| Dusk / Dawn | Warm gradient wash, sun rendered low at 78% screen height |
| Fog / Overcast | Drifting translucent bands |

## Architecture

```
UI (Compose + Material 3)
  └── ViewModels (StateFlow + sealed UiState: Loading / Success / Error)
        └── Repositories (Weather, Location) — parallel fetches via async/awaitAll
              └── Retrofit + Gson → Open-Meteo (forecast, geocoding, air quality)
              └── DataStore (settings, saved cities, offline cache)
```

**Deliberate choices:** manual DI via `AppContainer` (no Hilt — simpler, more debuggable generated code), Canvas over Lottie (no asset pipeline, full control), Open-Meteo over OpenWeatherMap (no API key, WMO codes map 1:1 to the theming enum, CC BY 4.0).

## How it was built

1. **Research** — competitive scan of CARROT, Apple Weather, (Not Boring) Weather, Overdrop; framework bake-off (Flutter vs KMP vs native).
2. **Design** — full 5-screen design system in Google Stitch (indigo `#0B1026 → #1B2A52`, glassmorphism, 96sp ultra-light numerals), iterated through a senior-review loop that produced a 4-variant "one layout, four atmospheres" hero system.
3. **Build** — Google AI Studio, driven by a Master Prompt contract (pinned stack, quality bars, phased delivery) + 4 phase prompts.
4. **Review** — every build zip code-reviewed: caught a dependency-bloat P0 (Firebase/Room/Moshi boilerplate), a WMO-mapping P1 (merged cloud codes), and a real user-reported bug (noon sun at 7 PM → root cause: binary `is_day` + stale fetches → fix: the `DayPhase` state machine).

## How I would measure it

**North star: % of users who open the app at least once daily over a 7-day window.** Weather is an inherently high-frequency utility category — if the atmosphere engine and AQI-first design aren't driving daily habitual opens more than a generic forecast app would, the core bet hasn't paid off.

Supporting metrics:

- **% of sessions that interact with the radar or AQI card** — tests whether the differentiating features are actually used, not just admired once and ignored.
- **Average session length** — a hand-rolled atmosphere engine invites lingering versus a quick-glance utility; this checks whether that's actually happening.
- **Crash-free session rate** — custom particle systems and hand-rolled state machines (like `DayPhase`) carry real stability risk that a template-based app wouldn't have; this is the honest check on whether the differentiator is also reliable.

## Known limits

An honest list of what's missing or unverified:

- **No GPS auto-location yet** — city selection is currently manual.
- **Radar is past-looking only** — no forward-predicted precipitation, by design (see trade-off above), but worth stating plainly.
- **No automated tests** — verification has been manual, phone-in-hand testing.
- **Performance across device tiers is untested** — the hand-rolled particle systems have only been verified on the devices used during development, not across low-end or older hardware.

## Running it

Open in Android Studio (Ladybug+), sync, run. No API keys required — all data sources are free tiers.

- Weather, geocoding & air quality: [Open-Meteo](https://open-meteo.com/) (CC BY 4.0)
- Radar: [RainViewer](https://www.rainviewer.com/) · Map © CARTO © OpenStreetMap contributors

## Development note

This project was built using AI-assisted development with [Claude Code](https://claude.com/claude-code), Anthropic's agentic coding tool.

---

**Anubhav Kapoor** — Product & Growth Associate → APM. Part of a portfolio of shipped product experiments: [Khyaal](https://github.com/anubhavk42/khyaal) · [SAAR](https://github.com/anubhavk42/saar) · [NutriLens AI](https://github.com/anubhavk42/nutrilens-ai) · [Disha](https://github.com/anubhavk42/disha-career-app)

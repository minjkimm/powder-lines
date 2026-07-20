# Powder Lines · Tirol

A prototype that scores backcountry ski runs for powder quality, per segment, gated by avalanche danger. Covers 8 touring regions in Tirol and Vorarlberg with live OSM routes, live and historical weather, and a region leaderboard.

**Live app:** open `index.html` via GitHub Pages.

## What it does

Each ski-tour route is split into segments. Every segment gets a 0 to 100 powder score from new snow, temperature trend, wind loading and scouring, sun aging, freezing level, and slope angle. Separately, an avalanche danger gate can veto any segment: High danger at that elevation, or Considerable danger on wind-loaded aspects, draws the segment red regardless of its score. The deepest pocket and the wind slab are usually the same place. Deep does not mean safe.

Three weather modes: Live (Open-Meteo forecast), Replay (pick any past date, ERA5 archive), Simulate (sliders).

## Warning

**This is a prototype, not a forecast.** Routes are community OSM data. Aspect is estimated from the fall line of the drawn route, not true surface aspect. Avalanche danger is manual user input until the Euregio bulletin integration ships. Nothing here overrides the official forecast. Plan your day with [avalanche.report](https://avalanche.report) and your own judgment.

## Data sources

- Routes: OpenStreetMap via Overpass API (`piste:type=skitour`)
- Elevation: Open-Meteo elevation API
- Weather: Open-Meteo forecast API (live) and ERA5 archive (replay)
- Basemaps: CARTO light, OpenTopoMap

## Roadmap

Euregio avalanche.report CAAML feed (Nov), true DEM surface aspect, GPX import for personal run libraries, rule backtesting against the bulletin archive.

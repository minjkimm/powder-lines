# Powder Lines · Tirol

A prototype that answers one question: **which backcountry ski line skis deep and is worth skinning, given honest avalanche constraints?** It scores runs per segment from open data, covers 8 touring regions in Tirol and Vorarlberg, and never lets snow quality override safety.

**Live app:** https://minjkimm.github.io/powder-lines/

## Product framing

The user is someone willing to drive to the mountains after a storm who does not know every valley by heart. Her decision sequence is: which region → which line → is it safe → how do I navigate it. The UI mirrors that sequence: a region leaderboard first, then a ranked run list with real place names ("Praxmar → Zischgeles"), then the map, then a GPX she can take offline. Every design decision below serves that sequence.

## The "worth skinning" scoring model

Every route is split into ~180m segments, because a run is not one place: a single 1000m descent crosses elevation bands, aspects, and slope angles, and the snow differs on each. Scoring the whole run with one number would average away exactly the information a skier needs ("blower up top, rain crust exit"). Each segment gets a 0–100 score built additively:

- **Snow base (0–55 pts):** linear in 48h new snow, saturating at 60cm. Rationale: beyond ~60cm, more snow stops improving the skiing and starts increasing hazard and trail-breaking cost.
- **Temperature trend (+15 / +5 / −12):** cooling during a storm means light snow on top of dense ("right-side-up"), warming means the reverse ("upside-down"). Asymmetric penalty because upside-down snow ruins skiing faster than cooling improves it.
- **Aging and sun:** score decays per day since snowfall, slower on shaded northerly aspects, plus an extra penalty for sunny days on south-facing slopes. The model knows powder has a shelf life and that the sun is the main destroyer.
- **Freezing level:** −25 if the segment top sits below the freezing level, −10 within 150m of it. Rain on snow is disqualifying, not a minor deduction.
- **Wind (the double-edged term):** windward aspects lose up to 28 pts (scoured), lee aspects *gain* up to 16 (loaded deeper) but are flagged `loaded`. This is deliberate: the bonus and the flag travel together, feeding the safety gate below.
- **Slope shape:** below 15° the score is halved (too flat to ski powder), above 45° penalized (sluffs and scouring).

Human-readable thresholds: 75+ "Call in sick," 55+ "Worth the skin," 40+ "Fine, not special," under 40 "Save your legs." The labels force the model to commit to a verdict a user can act on, rather than hiding behind a number.

## Safety by design

The core principle: **the deepest pocket and the wind-slab problem are the same place. Deep does not mean safe.** The system encodes this as an architectural separation:

1. **Score and gate are independent computations.** The score says how good the snow is; the gate says whether you may want it. A segment at avalanche danger High (4+) at its elevation band, or Considerable (3) on a wind-loaded aspect, is gated: drawn red, flagged in every verdict, regardless of score. The gate cannot be outbid by a high score, and the tempting number is never presented as a recommendation.
2. **The wind bonus feeds the gate.** The same computation that rewards lee loading marks the segment as loaded, which is what the Considerable-level gate keys on. The model cannot praise a slope without simultaneously reporting why it is dangerous.
3. **Danger is never invented.** Danger ratings are user-entered (per elevation band) until the official Euregio CAAML feed is integrated. The app states this in the UI rather than estimating danger from weather, because a wrong estimate that looks authoritative is worse than an honest manual input.
4. **Regional forecast, per-segment application.** No forecast on Earth is slope-specific. Danger stays regional (by aspect and elevation band) and is applied to segments; the app never claims more precision than its sources have.
5. **Honest degradation ladder.** Live OSM routes → built-in starter set (clearly labeled "approx," with no GPX export, because a hand-drawn line must not masquerade as a track) → an explicit empty state that quotes its own diagnostic. Status pills always show which layers are live, simulated, or fallback. The user always knows what she is looking at.
6. **The disclaimer is a feature.** The app declares itself a prototype and defers to the official avalanche.report in the UI, permanently visible, not buried in an about page.

## Data sources (all open)

- Routes, peaks, villages: OpenStreetMap via Overpass API (`piste:type=skitour`), mirrors raced in parallel
- Elevation: Open-Meteo elevation API (per-segment slope and fall-line aspect)
- Weather: Open-Meteo forecast (live mode) and ERA5 archive (replay mode: score any past date, all 8 zones in one call)
- Basemaps: CARTO light, OpenTopoMap

## Known limitations

Aspect is estimated from the fall line of the drawn route, not true surface aspect (DEM-based aspect is roadmap). ERA5 replay is model reanalysis at km-scale grid, regional truth rather than couloir truth. Danger input is manual until bulletin integration ships (November). Rule weights are heuristics pending backtesting against the Euregio bulletin archive.

## Warning

**This is a prototype, not a forecast.** Nothing here overrides the official bulletin. Plan with [avalanche.report](https://avalanche.report) and your own judgment.

## Roadmap

Euregio CAAML bulletin feed (live and archived, replacing manual danger), true DEM surface aspect, GPX import to score a user's personal run library, mobile layout pass, rule backtesting against bulletin archives, US expansion study (PNW and Colorado).

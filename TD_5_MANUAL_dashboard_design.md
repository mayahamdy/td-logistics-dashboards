# Student manual — Dashboard design: three roles, three KPIs, one logistics network

**Create a file called `REFLECTIONS.md` in your repo now, before you start.** Every
question in this document marked **Reflection** goes in it, numbered. The consolidated
list is repeated at the end of this document so you can check you haven't missed one.

**Everything below happens in a web browser on github.com. No local install
required.**

---

## The scenario — read this before anything else

One logistics network, three existing warehouses (Paris, Lyon, Lille), serving 20
real French cities. **Three different people look at this exact same network and
need three completely different dashboards**, because they're making three
completely different decisions:

| Dashboard | Who | Decision |
|---|---|---|
| `co2.html` | Sustainability/CSR manager | Which specific routes should be prioritized for CO2 reduction (consolidation, rail shift)? |
| `cost.html` | Logistics/operations manager | For each warehouse, is the cost problem transport or storage — because the fix is different for each? |
| `network.html` | Strategic/network planning manager | Should we open a warehouse in Bordeaux or Marseille, or close Lille? |

**This is CM1's "decisions hiding in plain sight" argument made as concrete as this
course gets it**: same underlying data, three real stakeholders, three genuinely
different charts — and for every chart, this document states *why* that chart and
not another, not just what it shows. That argument — not the HTML, not the JSON —
is the actual point of this TD.

> **Reflection 1.** Before you build anything: which of the three stakeholders above
> do you predict will disagree most sharply with the other two about what the
> network should actually do next — open a warehouse, close one, or change nothing?
> Write 1-2 sentences saying who, and why. You'll come back to this after Step 10.

## What you're building

```
td-logistics-dashboards/
  data/
    cities.csv                      # provided: 20 real French cities, coordinates, population
  scripts/
    compute_kpis.py                  # computes all three dashboards' data from one network
  docs/                              # GitHub Pages source
    index.html                        # landing page, links to the three dashboards
    co2.html
    cost.html
    network.html
    js/
      france-map-background.js        # shared: real France outline + a Chart.js background plugin
    data/
      co2.json                        # generated + committed
      cost.json                       # generated + committed
      network.json                    # generated + committed
  .github/workflows/
    compute.yml
  README.md
  REFLECTIONS.md                      # your answers, created above
```

**On maps:** two of the three dashboards (CO2, network) plot cities by real
longitude/latitude — Step 7 adds a real, simplified outline of France as a shallow
background behind those points, so "where" reads as an actual map instead of bare
coordinate axes. `cost.html` deliberately has no map at all — see Step 9's own
argument for why.

---

## Pipeline stage 1 — Data collection, storage, and provenance (CM3 / CM6)

*This is the acquisition-and-provenance discipline from CM3, and the honest-proxy /
"a number without its source is not trustworthy" discipline from CM6, applied before
you write a single line of analysis code.*

**The data and the assumptions — read before you trust a single number.**

**Cities:** 20 real French cities with real coordinates (city centers, adequate
precision for this exercise, same convention as the EU capitals/Smart Mobility
projects) and **approximate municipal population figures — treat these as
illustrative and re-verify against INSEE's current publication before citing them
professionally**, the exact same discipline `TD_2_DELIVERY_MANUAL_smart_mobility_lemans.md`
applies to its own commune population data.

**Demand is a population proxy, not real shipment data — stated explicitly, not
hidden in a formula.** There is no public dataset of real freight volumes between
these specific cities available to this course, so `compute_kpis.py` estimates each
city's annual shipped tonnage as `population / 1000 × 50` — an illustrative
assumption (50 tonnes/year per 1,000 residents), not a researched logistics
statistic. Same honesty requirement as Smart Mobility's "commune population as a
demand proxy" — a viewer of these dashboards needs to know this, so every dashboard
below states it.

**CO2 factor:** `0.10 kg CO2e per tonne-km`. Real-world cited figures for French/
European road freight span roughly **0.047 kg CO2e/tonne-km** (French Ministry of
Ecology's methodological guide, 2018, long-distance semi-trailer) up to
**~0.12 kg CO2e/tonne-km** (commonly cited range depending on vehicle type, load
factor, and whether the empty-return leg is included — see ADEME's Base Carbone/Base
Empreinte methodology). `0.10` is a round illustrative figure inside that real,
cited range — **check ADEME's current Base Empreinte figure before using this number
in anything you'd actually defend**, the same "verify before citing professionally"
discipline the EU capitals project applied to its own DEFRA emissions factor.

**Cost figures (`0.12 EUR/tonne-km` transport, `40 EUR/tonne/year` storage):**
**explicitly not researched, not sourced, illustrative only.** Unlike CO2, there is
no single authoritative government-published transport cost figure — real transport
pricing is commercial and highly variable. Both constants are clearly named and
placed at the top of the script specifically so they're easy to replace with real
figures if you have access to them.

> **Reflection 2.** Of the three assumptions above (population-as-demand, the CO2
> factor, the cost figures), which would you be **least** comfortable defending if a
> real client pushed back on it in a meeting? Name the one and say why in 1-2
> sentences — one of them has a real, cited, government-published range behind it,
> and the other two don't. That difference should show up in your answer.

### Step 1 — Create the repository

Public repo (GitHub Pages requires it), e.g. `td-logistics-dashboards`.

### Step 2 — Allow the workflow to write to the repo

**Settings → Actions → General → Workflow permissions → "Read and write
permissions" → Save.**

### Step 3 — Add the reference data (`data/cities.csv`)

This is your raw, collected dataset — the thing every KPI in this TD is ultimately
computed from. **Add file → Create new file** → `data/cities.csv` → paste:

```csv
city,lat,lon,population,role
Paris,48.8566,2.3522,2102650,existing_warehouse
Marseille,43.2965,5.3698,873076,candidate_warehouse
Lyon,45.7640,4.8357,522250,existing_warehouse
Toulouse,43.6047,1.4442,493465,customer
Nice,43.7102,7.2620,342637,customer
Nantes,47.2184,-1.5536,320732,customer
Montpellier,43.6108,3.8767,302454,customer
Strasbourg,48.5734,7.7521,291313,customer
Bordeaux,44.8378,-0.5792,261804,candidate_warehouse
Lille,50.6292,3.0573,233098,existing_warehouse
Rennes,48.1173,-1.6778,220488,customer
Reims,49.2583,4.0317,182460,customer
Le Havre,49.4944,0.1079,170147,customer
Saint-Etienne,45.4397,4.3872,171924,customer
Toulon,43.1242,5.9280,176198,customer
Grenoble,45.1885,5.7245,158552,customer
Dijon,47.3220,5.0415,156920,customer
Angers,47.4784,-0.5632,154508,customer
Le Mans,48.0061,0.1996,143847,customer
Clermont-Ferrand,45.7772,3.0870,143886,customer
```

Note that Marseille and Bordeaux are `candidate_warehouse`, not `customer` — they're
real cities with real demand of their own (served by whichever existing warehouse is
currently nearest), *and* they're the two locations under consideration for a new
warehouse in Step 10's network scenarios.

---

## Pipeline stage 2 — Turning one dataset into three KPI constructs (CM2)

*CM2's central argument is that a KPI is a construct you choose, not a neutral fact
that falls out of the data. This stage is that argument made literal: one script,
one dataset, and three genuinely different definitions of "what matters" computed
from it.*

### Step 4 — Add the computation script (`scripts/compute_kpis.py`)

```python
import csv, json, math, time

# -- Assumptions, stated explicitly (per this course's standing discipline: a
# number without its source/status is not trustworthy) --
TONNES_PER_1000_RESIDENTS_PER_YEAR = 50   # ILLUSTRATIVE PROXY, not a researched
                                            # logistics statistic -- see manual.
CO2_FACTOR_KG_PER_TONNE_KM = 0.10          # within the commonly-cited range for
                                            # French/European loaded-HGV road freight
                                            # (~0.047-0.12 kg CO2e/tonne-km depending
                                            # on load factor and source) -- verify
                                            # against ADEME's current Base Empreinte
                                            # figure before citing professionally.
COST_PER_TONNE_KM_EUR = 0.12               # ILLUSTRATIVE assumption, not sourced --
                                            # real transport cost is commercial/
                                            # variable, no single authoritative figure.
STORAGE_COST_PER_TONNE_YEAR_EUR = 40       # ILLUSTRATIVE assumption.

BASELINE_WAREHOUSES = ["Paris", "Lyon", "Lille"]
SCENARIOS = {
    "baseline (Paris, Lyon, Lille)": ["Paris", "Lyon", "Lille"],
    "+ Bordeaux": ["Paris", "Lyon", "Lille", "Bordeaux"],
    "+ Marseille": ["Paris", "Lyon", "Lille", "Marseille"],
    "- Lille (close it)": ["Paris", "Lyon"],
}

def haversine_km(lat1, lon1, lat2, lon2):
    r = 6371
    phi1, phi2 = math.radians(lat1), math.radians(lat2)
    dphi = math.radians(lat2 - lat1)
    dlambda = math.radians(lon2 - lon1)
    a = math.sin(dphi / 2) ** 2 + math.cos(phi1) * math.cos(phi2) * math.sin(dlambda / 2) ** 2
    return 2 * r * math.asin(math.sqrt(a))

def load_cities(path="data/cities.csv"):
    with open(path) as f:
        rows = list(csv.DictReader(f))
    for r in rows:
        r["lat"] = float(r["lat"])
        r["lon"] = float(r["lon"])
        r["population"] = int(r["population"])
        r["demand_tonnes"] = round(r["population"] / 1000 * TONNES_PER_1000_RESIDENTS_PER_YEAR, 1)
    return rows

def nearest_warehouse(city, warehouse_cities):
    best_wh, best_dist = None, None
    for wh in warehouse_cities:
        d = haversine_km(city["lat"], city["lon"], wh["lat"], wh["lon"])
        if best_dist is None or d < best_dist:
            best_dist, best_wh = d, wh["city"]
    return best_wh, round(best_dist, 1)

def assign_all(cities, warehouse_names):
    warehouse_cities = [c for c in cities if c["city"] in warehouse_names]
    links = []
    for c in cities:
        wh, dist = nearest_warehouse(c, warehouse_cities)
        links.append({**c, "assigned_warehouse": wh, "distance_km": dist})
    return links

def compute_co2(links):
    for l in links:
        l["co2_kg"] = round(l["distance_km"] * l["demand_tonnes"] * CO2_FACTOR_KG_PER_TONNE_KM, 1)
    by_warehouse = {}
    for l in links:
        by_warehouse.setdefault(l["assigned_warehouse"], {"co2_kg": 0.0, "tonnes": 0.0})
        by_warehouse[l["assigned_warehouse"]]["co2_kg"] += l["co2_kg"]
        by_warehouse[l["assigned_warehouse"]]["tonnes"] += l["demand_tonnes"]
    warehouse_summary = [
        {"warehouse": wh, "total_co2_kg": round(v["co2_kg"], 0),
         "co2_kg_per_tonne": round(v["co2_kg"] / v["tonnes"], 2) if v["tonnes"] else 0}
        for wh, v in by_warehouse.items()
    ]
    top_links = sorted(links, key=lambda l: l["co2_kg"], reverse=True)[:10]
    total_co2 = sum(l["co2_kg"] for l in links)
    total_tonnes = sum(l["demand_tonnes"] for l in links)
    return {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "assumptions": {
            "co2_factor_kg_per_tonne_km": CO2_FACTOR_KG_PER_TONNE_KM,
            "tonnes_per_1000_residents_per_year": TONNES_PER_1000_RESIDENTS_PER_YEAR,
        },
        "total_co2_kg": round(total_co2, 0),
        "co2_kg_per_tonne_network_avg": round(total_co2 / total_tonnes, 2) if total_tonnes else 0,
        "by_warehouse": warehouse_summary,
        "top_links": [
            {"city": l["city"], "warehouse": l["assigned_warehouse"],
             "distance_km": l["distance_km"], "co2_kg": l["co2_kg"]}
            for l in top_links
        ],
        "all_cities": [
            {"city": l["city"], "lat": l["lat"], "lon": l["lon"],
             "warehouse": l["assigned_warehouse"], "co2_kg": l["co2_kg"],
             "is_warehouse": l["city"] in BASELINE_WAREHOUSES}
            for l in links
        ],
    }

def compute_cost(links):
    for l in links:
        l["transport_cost_eur"] = round(l["distance_km"] * l["demand_tonnes"] * COST_PER_TONNE_KM_EUR, 0)
    by_warehouse = {}
    for l in links:
        by_warehouse.setdefault(l["assigned_warehouse"], {"transport_cost_eur": 0.0, "tonnes": 0.0})
        by_warehouse[l["assigned_warehouse"]]["transport_cost_eur"] += l["transport_cost_eur"]
        by_warehouse[l["assigned_warehouse"]]["tonnes"] += l["demand_tonnes"]
    warehouse_summary = []
    for wh, v in by_warehouse.items():
        storage_cost = round(v["tonnes"] * STORAGE_COST_PER_TONNE_YEAR_EUR, 0)
        warehouse_summary.append({
            "warehouse": wh,
            "transport_cost_eur": round(v["transport_cost_eur"], 0),
            "storage_cost_eur": storage_cost,
            "total_cost_eur": round(v["transport_cost_eur"] + storage_cost, 0),
            "cost_eur_per_tonne": round((v["transport_cost_eur"] + storage_cost) / v["tonnes"], 2) if v["tonnes"] else 0,
        })
    warehouse_summary.sort(key=lambda w: w["total_cost_eur"], reverse=True)
    top_links = sorted(links, key=lambda l: l["transport_cost_eur"], reverse=True)[:10]
    return {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "assumptions": {
            "cost_eur_per_tonne_km": COST_PER_TONNE_KM_EUR,
            "storage_cost_eur_per_tonne_year": STORAGE_COST_PER_TONNE_YEAR_EUR,
        },
        "by_warehouse": warehouse_summary,
        "top_links": [
            {"city": l["city"], "warehouse": l["assigned_warehouse"],
             "distance_km": l["distance_km"], "transport_cost_eur": l["transport_cost_eur"]}
            for l in top_links
        ],
    }

def compute_network_scenarios(cities):
    total_population = sum(c["population"] for c in cities)
    results = []
    scenario_links = {}
    for name, warehouses in SCENARIOS.items():
        links = assign_all(cities, warehouses)
        scenario_links[name] = links
        weighted_distance = sum(l["population"] * l["distance_km"] for l in links)
        avg_km_per_resident_weighted = weighted_distance / total_population
        results.append({
            "scenario": name,
            "n_warehouses": len(warehouses),
            "warehouses": warehouses,
            "avg_distance_km_population_weighted": round(avg_km_per_resident_weighted, 2),
            "max_city_distance_km": round(max(l["distance_km"] for l in links), 1),
        })
    results.sort(key=lambda r: r["avg_distance_km_population_weighted"])
    best_non_baseline = next((r for r in results if not r["scenario"].startswith("baseline")), results[1])
    return {
        "generated_at": time.strftime("%Y-%m-%dT%H:%M:%SZ", time.gmtime()),
        "scenarios": results,
        "baseline_map": [
            {"city": l["city"], "lat": l["lat"], "lon": l["lon"], "warehouse": l["assigned_warehouse"]}
            for l in scenario_links["baseline (Paris, Lyon, Lille)"]
        ],
        "best_alternative_map": [
            {"city": l["city"], "lat": l["lat"], "lon": l["lon"], "warehouse": l["assigned_warehouse"]}
            for l in scenario_links[best_non_baseline["scenario"]]
        ],
        "best_alternative_name": best_non_baseline["scenario"],
    }

def main():
    cities = load_cities()
    baseline_links = assign_all(cities, BASELINE_WAREHOUSES)

    co2 = compute_co2(baseline_links)
    cost = compute_cost(baseline_links)
    network = compute_network_scenarios(cities)

    print("=== CO2 (baseline network) ===")
    print(f"Total network CO2: {co2['total_co2_kg']:.0f} kg/year")
    print(f"Network avg CO2/tonne: {co2['co2_kg_per_tonne_network_avg']} kg/tonne")
    for w in co2["by_warehouse"]:
        print(f"  {w['warehouse']}: {w['total_co2_kg']:.0f} kg, {w['co2_kg_per_tonne']} kg/tonne")

    print("\n=== Cost (baseline network) ===")
    for w in cost["by_warehouse"]:
        print(f"  {w['warehouse']}: transport {w['transport_cost_eur']:.0f} EUR + storage "
              f"{w['storage_cost_eur']:.0f} EUR = {w['total_cost_eur']:.0f} EUR "
              f"({w['cost_eur_per_tonne']} EUR/tonne)")

    print("\n=== Network scenarios (population-weighted avg distance) ===")
    for s in network["scenarios"]:
        print(f"  {s['scenario']}: {s['avg_distance_km_population_weighted']} km avg "
              f"(worst city: {s['max_city_distance_km']} km), {s['n_warehouses']} warehouses")
    print(f"Best-performing alternative: {network['best_alternative_name']}")

    with open("docs/data/co2.json", "w") as f:
        json.dump(co2, f, indent=2)
    with open("docs/data/cost.json", "w") as f:
        json.dump(cost, f, indent=2)
    with open("docs/data/network.json", "w") as f:
        json.dump(network, f, indent=2)
    print("\nWrote docs/data/co2.json, cost.json, network.json")

if __name__ == "__main__":
    main()
```

**Real output from this exact script, this exact data:**

```
=== CO2 (baseline network) ===
Total network CO2: 6236312 kg/year
Network avg CO2/tonne: 16.8 kg/tonne
  Paris: 1497094 kg, 9.09 kg/tonne
  Lyon: 4739218 kg, 24.34 kg/tonne
  Lille: 0 kg, 0.0 kg/tonne

=== Cost (baseline network) ===
  Lyon: transport 5687061 EUR + storage 7788952 EUR = 13476013 EUR (69.21 EUR/tonne)
  Paris: transport 1796513 EUR + storage 6589668 EUR = 8386181 EUR (50.91 EUR/tonne)
  Lille: transport 0 EUR + storage 466196 EUR = 466196 EUR (40.0 EUR/tonne)

=== Network scenarios (population-weighted avg distance) ===
  + Marseille: 115.03 km avg (worst city: 435.8 km), 4 warehouses
  + Bordeaux: 139.89 km avg (worst city: 382.3 km), 4 warehouses
  baseline (Paris, Lyon, Lille): 168.04 km avg (worst city: 435.8 km), 3 warehouses
  - Lille (close it): 174.43 km avg (worst city: 435.8 km), 2 warehouses
Best-performing alternative: + Marseille
```

**A real, surprising finding worth reading carefully, not glossing over:** Lille
contributes **zero** CO2 and zero transport cost in the baseline — every one of the
other 19 cities turns out to be geometrically closer to Paris or Lyon than to Lille
(confirmed by inspecting the real per-city assignment: Reims, at 130km from Paris, is
closer to Paris than to Lille). Lille only ever "delivers" to itself. And yet,
closing Lille (the fourth scenario) makes the network-wide average distance *worse*
(168.04 -> 174.43 km), not better — because Lille's own local population (233,098
people) must then travel to Paris instead of zero distance. **A warehouse can look
completely idle in a routes-served view and still be reducing total network
distance, because it serves its own local market.** This is exactly the kind of
thing a single KPI hides — which is precisely why this TD builds three different
views instead of one, and why the network dashboard's own caveat (Step 10) explicitly
says a distance-only model isn't the whole decision. Hold onto this — you'll need it
for Reflection 6.

> **Reflection 3.** This one script just produced three genuinely different KPI
> constructs (CO2, cost, network distance) from the exact same 20-city, 3-warehouse
> network. Pick any **two** of the three, and describe one real instance from your
> own output above where they'd point a decision-maker toward *different* actions
> for the same warehouse. (Lille's 0 kg / 0 EUR versus its role in the network
> scenarios is one example already sitting in front of you — you're free to use it,
> or find your own.)

### Step 5 — Run it, check the checkpoint

You'll need placeholder JSON files first so nothing 404s before the workflow runs —
**Add file → Create new file** for each of `docs/data/co2.json`, `docs/data/cost.json`,
`docs/data/network.json`, paste `{}` into each, commit.

**Checkpoint (after Step 11 runs the real workflow):** your own output should be
numerically close to the figures above (identical, since the data and formulas are
deterministic) — if `Lille: 0 kg` doesn't appear in your own run, something changed
in `data/cities.csv` or the script; that specific line is worth double-checking
against the real finding above.

---

## Pipeline stage 3 — Presentation infrastructure (supporting, not a decision point)

### Step 6 — Add the landing page (`docs/index.html`)

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Logistics network — three dashboards, three decisions</title>
<style>
  body { font-family: system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; }
  .card { display: block; padding: 1rem 1.2rem; margin: 1rem 0; border: 1px solid #ccc; border-radius: 8px; text-decoration: none; color: inherit; }
  .card:hover { border-color: #0d7c9e; }
  .card h2 { margin: 0 0 0.3rem; }
  .card .for { font-size: 0.8rem; color: #666; text-transform: uppercase; letter-spacing: 0.03em; }
</style>
</head>
<body>
  <h1>Same network, three different decisions</h1>
  <p>One real French logistics network (3 existing warehouses, 20 real French cities, distances computed from real coordinates) — three dashboards, each built for a different person and a different decision, each with a deliberately different chart-choice argument. See <code>TD_5_DELIVERY_MANUAL_dashboard_design.md</code> for the full reasoning behind each choice.</p>

  <a class="card" href="co2.html">
    <div class="for">Sustainability / CSR manager</div>
    <h2>CO2 dashboard</h2>
    <p>Which routes should we prioritize to cut network CO2? Ranked bar chart + a spatial scatter, because the decision is about both "how much" and "where."</p>
  </a>

  <a class="card" href="cost.html">
    <div class="for">Logistics / operations manager</div>
    <h2>Cost dashboard</h2>
    <p>Where does cost concentrate — transport or storage — per warehouse? A stacked bar (not a bar, not a pie) plus an exact-figures table, because the decision is about which lever to pull.</p>
  </a>

  <a class="card" href="network.html">
    <div class="for">Strategic / network planning manager</div>
    <h2>Network dashboard</h2>
    <p>Should we open a warehouse in Bordeaux or Marseille, or close Lille? A discrete-scenario bar chart plus side-by-side maps, because this decision is fundamentally geographic.</p>
  </a>
</body>
</html>
```

### Step 7 — Add the shared France map background (`docs/js/france-map-background.js`)

**The argument, before the code:** bare longitude/latitude axes make a viewer do the
work of mentally reconstructing "where is that" from two numbers. A real, even
lightly-drawn outline of France turns that into instant recognition — this is the
one visual upgrade that benefits multiple dashboards at once (CO2 and network both
plot real coordinates), so it belongs in one shared file both pages load, not copied
into each.

The outline itself is real data, not hand-drawn: it comes from
[gregoiredavid/france-geojson](https://github.com/gregoiredavid/france-geojson)'s
`metropole.geojson` (mainland France + Corsica, Etalab Open Licence), simplified from
~28,000 raw coordinate points down to 219 with the Ramer-Douglas-Peucker algorithm —
enough detail to be instantly recognizable as France, light enough to cost nothing to
load. "Shallow" here is deliberate: this is a background, not the main content, so it
renders as a faint fill with a thin outline, never competing with the actual data
points on top of it.

**Add file → Create new file** → `docs/js/france-map-background.js` → paste:

```javascript
// Real French border outline (mainland + Corsica) -- simplified from
// gregoiredavid/france-geojson's metropole.geojson (Etalab Open Licence),
// via Ramer-Douglas-Peucker simplification (epsilon 0.08 deg mainland,
// 0.025 deg Corsica): real coordinates, reduced from ~28,000 points to 219
// so a "shallow background" costs almost nothing to draw or download.
// Shared by every dashboard in this TD that plots real lon/lat points --
// see TD_5_DELIVERY_MANUAL_dashboard_design.md for how it was built.

const FRANCE_OUTLINE = {
  mainland: [[7.13,47.503],[6.983,47.493],[6.879,47.352],[7.057,47.335],[6.112,46.578],[6.064,46.416],[6.17,46.366],[5.956,46.132],[6.294,46.225],[6.305,46.367],[6.805,46.394],[6.797,46.139],[7.045,45.922],[6.808,45.725],[7.186,45.403],[7.08,45.213],[6.627,45.104],[6.751,44.906],[7.024,44.823],[7.077,44.681],[6.853,44.529],[7.009,44.235],[7.358,44.116],[7.685,44.174],[7.529,43.784],[6.745,43.418],[6.584,43.277],[6.698,43.266],[6.647,43.168],[6.149,43.026],[5.342,43.213],[5.314,43.36],[5.05,43.325],[4.861,43.454],[4.855,43.333],[4.659,43.346],[4.007,43.552],[3.107,43.089],[3.024,42.704],[3.174,42.436],[2.015,42.347],[1.731,42.493],[1.736,42.617],[1.438,42.603],[1.357,42.719],[0.708,42.861],[0.668,42.689],[-0.017,42.685],[-0.313,42.849],[-0.567,42.781],[-0.753,42.968],[-1.27,43.119],[-1.441,43.046],[-1.389,43.259],[-1.73,43.296],[-1.789,43.378],[-1.447,43.646],[-1.259,44.547],[-1.192,44.661],[-1.004,44.648],[-1.168,44.776],[-1.262,44.632],[-1.156,45.476],[-0.989,45.581],[-1.237,45.706],[-1.064,45.95],[-1.242,46.157],[-1.115,46.292],[-1.812,46.493],[-2.142,46.819],[-1.98,47.029],[-2.247,47.132],[-2.17,47.268],[-2.546,47.291],[-2.421,47.495],[-2.815,47.487],[-2.914,47.557],[-2.69,47.637],[-2.978,47.659],[-2.927,47.557],[-3.127,47.598],[-3.133,47.475],[-3.135,47.747],[-3.215,47.645],[-3.389,47.828],[-3.45,47.695],[-3.542,47.863],[-3.539,47.763],[-3.856,47.79],[-3.977,47.91],[-4.099,47.862],[-4.117,47.983],[-4.183,47.8],[-4.374,47.798],[-4.488,48.038],[-4.741,48.041],[-4.273,48.154],[-4.554,48.168],[-4.629,48.281],[-4.097,48.256],[-4.457,48.327],[-4.27,48.445],[-4.717,48.33],[-4.795,48.415],[-4.704,48.57],[-4.505,48.546],[-4.612,48.578],[-4.478,48.573],[-4.544,48.636],[-3.971,48.727],[-3.852,48.607],[-3.821,48.718],[-3.573,48.675],[-3.504,48.839],[-3.224,48.87],[-3.22,48.784],[-3.074,48.882],[-3.123,48.76],[-3.009,48.82],[-2.682,48.492],[-2.318,48.689],[-2.213,48.573],[-2.047,48.638],[-2.002,48.492],[-1.937,48.702],[-1.352,48.631],[-1.571,48.744],[-1.569,49.222],[-1.809,49.372],[-1.942,49.726],[-1.266,49.695],[-1.307,49.542],[-1.136,49.354],[-0.087,49.298],[0.341,49.434],[0.067,49.51],[0.206,49.714],[1.682,50.181],[1.538,50.279],[1.659,50.508],[1.582,50.871],[2.546,51.089],[2.599,50.849],[2.79,50.726],[3.151,50.79],[3.289,50.526],[3.608,50.497],[3.71,50.303],[4.208,50.273],[4.196,49.955],[4.694,49.996],[4.825,50.169],[4.851,49.794],[5.271,49.695],[5.465,49.499],[6.241,49.515],[6.542,49.432],[6.739,49.164],[7.445,49.184],[8.225,48.975],[7.84,48.641],[7.577,48.12],[7.585,47.576],[7.386,47.432],[7.13,47.503]],
  corsica: [[9.402,41.859],[9.385,41.646],[9.281,41.597],[9.368,41.591],[9.273,41.529],[9.288,41.484],[9.215,41.414],[9.26,41.414],[9.22,41.367],[9.097,41.392],[9.122,41.441],[9.07,41.445],[9.08,41.477],[8.92,41.489],[8.782,41.566],[8.794,41.633],[8.914,41.691],[8.659,41.74],[8.771,41.811],[8.751,41.845],[8.803,41.896],[8.752,41.933],[8.607,41.895],[8.592,41.964],[8.747,42.051],[8.7,42.112],[8.559,42.146],[8.593,42.168],[8.54,42.237],[8.693,42.268],[8.543,42.367],[8.657,42.417],[8.665,42.514],[8.72,42.525],[8.709,42.577],[8.781,42.557],[8.803,42.603],[9.015,42.641],[9.125,42.732],[9.221,42.736],[9.314,42.689],[9.346,42.743],[9.321,42.897],[9.361,42.925],[9.34,42.993],[9.42,43.012],[9.464,42.986],[9.493,42.806],[9.446,42.674],[9.533,42.548],[9.559,42.197],[9.55,42.104],[9.417,41.961],[9.402,41.859]]
};

// A France-sized bounding box (with a small margin) for any scatter chart's
// x/y scale min/max, so the outline is never clipped and every dashboard
// uses the same visual frame.
const FRANCE_BBOX = { minLon: -5.3, maxLon: 9.8, minLat: 41.2, maxLat: 51.3 };

// Chart.js plugin: draws the outline BEHIND the chart's own datasets, using
// the chart's own x/y scale pixel conversion -- this is what keeps the map
// perfectly aligned with the real lon/lat data points regardless of canvas
// size or padding, instead of a separately-positioned background image that
// would drift out of alignment on resize.
const franceBackgroundPlugin = {
  id: 'franceBackground',
  beforeDatasetsDraw(chart) {
    const { ctx, scales } = chart;
    const x = scales.x, y = scales.y;
    if (!x || !y) return;
    ctx.save();
    ctx.fillStyle = 'rgba(13, 124, 158, 0.07)';
    ctx.strokeStyle = 'rgba(13, 124, 158, 0.35)';
    ctx.lineWidth = 1;
    [FRANCE_OUTLINE.mainland, FRANCE_OUTLINE.corsica].forEach(ring => {
      ctx.beginPath();
      ring.forEach(([lon, lat], i) => {
        const px = x.getPixelForValue(lon);
        const py = y.getPixelForValue(lat);
        if (i === 0) ctx.moveTo(px, py); else ctx.lineTo(px, py);
      });
      ctx.closePath();
      ctx.fill();
      ctx.stroke();
    });
    ctx.restore();
  },
};
```

**Why a Chart.js plugin drawing on the chart's own canvas, not a background
`<img>` positioned with CSS:** a plugin using `chart.scales.x.getPixelForValue()`
stays perfectly aligned with the real data points at any canvas size, on any screen,
after any resize — because it's computed from the exact same coordinate
transformation Chart.js itself uses for the data points. A separately-positioned
background image would need its own manual pixel alignment against Chart.js's
computed padding/legend/title layout, which drifts the moment either element
changes.

---

## Pipeline stage 4 — Visualization for decision support (CM1), three times over

*Every chart from here on exists to answer one specific person's one specific
decision — not "to visualize the data." Read each argument before you paste the
code, and actually evaluate it: do you agree with the chart choice, or would you
have picked differently?*

### Step 8 — Add the CO2 dashboard (`docs/co2.html`) — the argued design

**The argument, before the code:** this decision is about *specific, actionable
routes* ("prioritize this one for consolidation") **and** about *where* emissions
concentrate geographically (rail-shift and consolidation options depend on regional
infrastructure, not just a route's raw number) — so this page uses **two** charts, not
one, each earning its place: a ranked bar chart for the first question, a spatial
scatter (real longitude/latitude as x/y — a crude but honest substitute for a real
map, no mapping library needed) for the second. No pie chart anywhere: comparing many
routes' relative share of total CO2 by angle is a well-known weak reading task next
to a sorted bar.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>CO2 dashboard — Sustainability</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
<script src="js/france-map-background.js"></script>
<style>
  body { font-family: system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; }
  nav a { margin-right: 1rem; }
  .kpis { display: flex; gap: 1rem; flex-wrap: wrap; margin: 1rem 0; }
  .kpi { flex: 1; min-width: 160px; padding: 0.8rem 1rem; border: 1px solid #ccc; border-radius: 8px; }
  .kpi .value { font-size: 1.5rem; font-weight: bold; color: #2e8b46; }
  .kpi .label { font-size: 0.8rem; color: #666; }
  #assumptions { font-size: 0.8rem; color: #666; padding: 0.6rem 0.9rem; border-left: 3px solid #2e8b46; margin: 1rem 0; }
  canvas { margin-top: 1.5rem; }
</style>
</head>
<body>
  <nav><a href="index.html">Overview</a> | <strong>CO2 (sustainability)</strong> | <a href="cost.html">Cost (operations)</a> | <a href="network.html">Network (strategy)</a></nav>
  <h1>Which routes should we prioritize to cut network CO2?</h1>
  <p>For: sustainability/CSR manager. Decision this supports: which specific delivery routes are the best candidates for consolidation or a shift to rail, and which warehouse is the least CO2-efficient per tonne delivered.</p>
  <div class="kpis" id="kpis">Loading...</div>
  <div id="assumptions"></div>

  <h2>Highest-CO2 routes (ranking → bar chart)</h2>
  <p>A ranked list of specific, actionable routes is exactly what a bar chart is for — each bar is one route someone could actually change.</p>
  <canvas id="topLinksChart" height="280"></canvas>

  <h2>Where emissions concentrate (spatial → scatter over a real France outline)</h2>
  <p>This decision is inherently about geography — which <em>region</em> to target for consolidation — so a plain ranking isn't enough; we plot each city at its real coordinates, colored by its CO2 contribution, over a real (simplified) outline of France so "where" reads at a glance instead of requiring you to mentally place bare longitude/latitude numbers. Existing warehouses get their own marker <em>shape</em> (a triangle), not just a color — a shape difference survives grayscale printing and color-blindness in a way color alone doesn't, and it answers a second real question at a glance: is CO2 concentrating near a warehouse (fixable by re-routing) or far from every warehouse (fixable only by opening a new one)?</p>
  <canvas id="mapChart" height="360"></canvas>

<script>
fetch('data/co2.json')
  .then(r => r.json())
  .then(data => {
    document.getElementById('kpis').innerHTML = `
      <div class="kpi"><div class="value">${(data.total_co2_kg / 1000).toFixed(0)} t</div><div class="label">Total network CO2 / year</div></div>
      <div class="kpi"><div class="value">${data.co2_kg_per_tonne_network_avg}</div><div class="label">kg CO2 per tonne delivered (network avg)</div></div>
    `;
    document.getElementById('assumptions').innerHTML =
      `Assumptions: ${data.assumptions.co2_factor_kg_per_tonne_km} kg CO2e/tonne-km (illustrative, within the commonly-cited range for road freight — verify against ADEME's current Base Empreinte figure), ` +
      `demand estimated at ${data.assumptions.tonnes_per_1000_residents_per_year} tonnes/year per 1,000 residents (a proxy, not measured shipment data).`;

    const top = data.top_links || [];
    new Chart(document.getElementById('topLinksChart'), {
      type: 'bar',
      data: { labels: top.map(l => `${l.city} <- ${l.warehouse}`), datasets: [{ label: 'CO2 (kg/year)', data: top.map(l => l.co2_kg), backgroundColor: '#2e8b46' }] },
      options: { indexAxis: 'y', plugins: { legend: { display: false }, title: { display: true, text: 'Top 10 highest-CO2 routes' } } },
    });

    const cities = data.all_cities || [];
    const maxCo2 = Math.max(...cities.map(c => c.co2_kg), 1);
    const warehouses = cities.filter(c => c.is_warehouse);
    const served = cities.filter(c => !c.is_warehouse);
    new Chart(document.getElementById('mapChart'), {
      type: 'scatter',
      data: {
        // Two datasets, not one, specifically so the warehouse cities get
        // their own legend entry and a shape (triangle) that reads as
        // "different kind of thing" at a glance, not just a bigger dot --
        // a shape difference survives grayscale printing and color-blindness
        // in a way a color-only distinction doesn't.
        datasets: [
          {
            label: 'Existing warehouse',
            data: warehouses.map(c => ({ x: c.lon, y: c.lat, co2: c.co2_kg, city: c.city, warehouse: c.warehouse })),
            backgroundColor: '#1a1a2e',
            pointStyle: 'triangle',
            pointRadius: 10,
            pointBorderColor: '#fff',
            pointBorderWidth: 1,
          },
          {
            label: 'City (size/color = CO2 contribution)',
            data: served.map(c => ({ x: c.lon, y: c.lat, co2: c.co2_kg, city: c.city, warehouse: c.warehouse })),
            backgroundColor: served.map(c => `hsl(${Math.max(0, 120 - 120 * (c.co2_kg / maxCo2))}, 70%, 45%)`),
            pointStyle: 'circle',
            pointRadius: served.map(c => 4 + 12 * Math.sqrt(c.co2_kg / maxCo2)),
          },
        ],
      },
      // franceBackgroundPlugin (js/france-map-background.js, Step 7) draws a
      // real, simplified France outline behind the points, using the chart's
      // own scale conversion -- see Step 7 for why this stays aligned at any
      // canvas size, unlike a separately-positioned background image.
      plugins: [franceBackgroundPlugin],
      options: {
        plugins: {
          legend: { position: 'bottom' },
          title: { display: true, text: 'Real coordinates over a real France outline — triangle = existing warehouse, bigger/redder circle = more CO2' },
          tooltip: { callbacks: { label: (item) => {
            const c = item.raw;
            return item.datasetIndex === 0
              ? [`${c.city} (existing warehouse)`]
              : [`${c.city} -> ${c.warehouse}`, `${c.co2.toFixed(0)} kg CO2/year`];
          } } },
        },
        scales: {
          x: { title: { display: true, text: 'Longitude' }, min: FRANCE_BBOX.minLon, max: FRANCE_BBOX.maxLon },
          y: { title: { display: true, text: 'Latitude' }, min: FRANCE_BBOX.minLat, max: FRANCE_BBOX.maxLat },
        },
      },
    });
  })
  .catch(err => { document.getElementById('kpis').innerHTML = '<p>Could not load data (' + err + ').</p>'; });
</script>
</body>
</html>
```

> **Reflection 4.** Look at your own real top-10 list on the live page. Would you
> actually recommend the #1-ranked route for consolidation as-is, or does something
> about it — which warehouse it's assigned to, the raw distance, or something you
> know about that region — make you want to double-check it before recommending it
> to the sustainability manager? Say what, specifically.

### Step 9 — Add the cost dashboard (`docs/cost.html`) — the argued design

**The argument, before the code:** the decision here is *which lever to pull* per
warehouse — transport or storage — so the chart needs to show **both the total and
the composition at once**. A plain bar shows total cost but hides the split. A pie
per warehouse would show the split but makes comparing *across* warehouses hard
(three separate pies, comparing wedge angles, is a known-weak reading task). A
**stacked bar** is the one chart that answers both parts of the question in one
view. The second component is a **table, not a chart** — deliberately, because a
manager about to renegotiate a specific route needs the exact number, the same
reasoning behind the exact-figures offer table in
`TP_1_DELIVERY_MANUAL_smart_internship_match.md`. No spatial/map chart on this page
at all — unlike the CO2 dashboard, this decision is about which lever, not where, so
a map would add clutter without adding decision value. That contrast is deliberate:
identical underlying data, and the right chart set is still different.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Cost dashboard — Operations</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
<style>
  body { font-family: system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; }
  nav a { margin-right: 1rem; }
  .kpis { display: flex; gap: 1rem; flex-wrap: wrap; margin: 1rem 0; }
  .kpi { flex: 1; min-width: 160px; padding: 0.8rem 1rem; border: 1px solid #ccc; border-radius: 8px; }
  .kpi .value { font-size: 1.5rem; font-weight: bold; color: #c46e0a; }
  .kpi .label { font-size: 0.8rem; color: #666; }
  #assumptions { font-size: 0.8rem; color: #666; padding: 0.6rem 0.9rem; border-left: 3px solid #c46e0a; margin: 1rem 0; }
  canvas { margin-top: 1.5rem; }
  table { width: 100%; border-collapse: collapse; margin-top: 1rem; font-size: 0.85rem; }
  th, td { text-align: left; padding: 0.3rem 0.5rem; border-bottom: 1px solid #ddd; }
</style>
</head>
<body>
  <nav><a href="index.html">Overview</a> | <a href="co2.html">CO2 (sustainability)</a> | <strong>Cost (operations)</strong> | <a href="network.html">Network (strategy)</a></nav>
  <h1>Where does logistics cost concentrate, and which lever matters?</h1>
  <p>For: logistics/operations cost manager. Decision this supports: for each warehouse, is its cost problem mainly transport (routes too long/frequent) or storage (holding too much stock) — because the fix is different for each.</p>
  <div class="kpis" id="kpis">Loading...</div>
  <div id="assumptions"></div>

  <h2>Cost composition per warehouse (stacked bar, not a bar or a pie)</h2>
  <p>A plain bar would show total cost but hide the transport/storage split; a pie per warehouse would show the split but make comparing warehouses to each other hard (comparing angles across three separate pies is a known weak reading task). A <strong>stacked bar</strong> shows both at once: bar height = total cost, segment size = which lever to pull.</p>
  <canvas id="stackedChart" height="280"></canvas>

  <h2>Highest-cost individual routes (table, not a chart)</h2>
  <p>A manager negotiating a specific route needs the exact number, not a visual estimate — same reasoning this course used for every other "exact figures a person will act on" table (e.g. the offer table in Smart Internship Match).</p>
  <table id="topLinksTable"><thead><tr><th>City</th><th>Warehouse</th><th>Distance (km)</th><th>Transport cost (EUR/year)</th></tr></thead><tbody></tbody></table>

<script>
fetch('data/cost.json')
  .then(r => r.json())
  .then(data => {
    const byWh = data.by_warehouse || [];
    const totalCost = byWh.reduce((s, w) => s + w.total_cost_eur, 0);
    const avgCostPerTonne = byWh.length ? (byWh.reduce((s, w) => s + w.cost_eur_per_tonne, 0) / byWh.length) : 0;

    document.getElementById('kpis').innerHTML = `
      <div class="kpi"><div class="value">${(totalCost / 1000).toFixed(0)}k EUR</div><div class="label">Total network cost / year</div></div>
      <div class="kpi"><div class="value">${avgCostPerTonne.toFixed(0)} EUR</div><div class="label">Avg cost per tonne (mean across warehouses)</div></div>
    `;
    document.getElementById('assumptions').innerHTML =
      `Assumptions (illustrative, not researched market rates): ${data.assumptions.cost_eur_per_tonne_km} EUR/tonne-km transport, ` +
      `${data.assumptions.storage_cost_eur_per_tonne_year} EUR/tonne/year storage.`;

    new Chart(document.getElementById('stackedChart'), {
      type: 'bar',
      data: {
        labels: byWh.map(w => w.warehouse),
        datasets: [
          { label: 'Transport cost', data: byWh.map(w => w.transport_cost_eur), backgroundColor: '#0d7c9e' },
          { label: 'Storage cost', data: byWh.map(w => w.storage_cost_eur), backgroundColor: '#c46e0a' },
        ],
      },
      options: {
        plugins: { title: { display: true, text: 'Total cost per warehouse, split by lever' } },
        scales: { x: { stacked: true }, y: { stacked: true, title: { display: true, text: 'EUR / year' } } },
      },
    });

    const tbody = document.querySelector('#topLinksTable tbody');
    tbody.innerHTML = (data.top_links || []).map(l => `
      <tr><td>${l.city}</td><td>${l.warehouse}</td><td>${l.distance_km}</td><td>${l.transport_cost_eur.toFixed(0)}</td></tr>
    `).join('');
  })
  .catch(err => { document.getElementById('kpis').innerHTML = '<p>Could not load data (' + err + ').</p>'; });
</script>
</body>
</html>
```

> **Reflection 5.** Look at your own real per-warehouse numbers on the live page.
> For whichever warehouse has the highest total cost, which lever would you pull
> first — transport or storage — and why? Name the actual number that convinced you.

### Step 10 — Add the network dashboard (`docs/network.html`) — the argued design

**The argument, before the code:** four candidate network configurations are
**discrete, mutually exclusive choices**, not points on a continuum — so the
comparison chart is a bar, one per scenario, sorted best-to-worst; a line chart here
would visually imply you could interpolate between "3 warehouses" and "4 warehouses,"
which makes no sense. The second half is **the clearest case for a spatial view in
this entire TD** — unlike the cost dashboard, this decision is fundamentally
*about* geography, so two small-multiple maps (baseline vs. the best alternative,
side by side, colored by assigned warehouse, with the warehouse city itself marked
by a distinct triangle so "which network node is a warehouse" is never ambiguous)
are the actual evidence, not a supporting visual. A stated caveat closes the page,
per CM2's multi-objective principle: this analysis is distance-only and doesn't
capture capex, lease-exit cost, or Lille's non-distance strategic value.

```html
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="utf-8">
<title>Network dashboard — Strategy</title>
<script src="https://cdn.jsdelivr.net/npm/chart.js@4"></script>
<script src="js/france-map-background.js"></script>
<style>
  body { font-family: system-ui, sans-serif; max-width: 900px; margin: 2rem auto; padding: 0 1rem; }
  nav a { margin-right: 1rem; }
  .kpis { display: flex; gap: 1rem; flex-wrap: wrap; margin: 1rem 0; }
  .kpi { flex: 1; min-width: 160px; padding: 0.8rem 1rem; border: 1px solid #ccc; border-radius: 8px; }
  .kpi .value { font-size: 1.5rem; font-weight: bold; color: #8b2ea0; }
  .kpi .label { font-size: 0.8rem; color: #666; }
  #caveat { padding: 0.8rem 1rem; border: 2px solid #c46e0a; border-radius: 8px; background: #faf3e8; margin: 1rem 0; }
  .maps { display: flex; gap: 1rem; flex-wrap: wrap; }
  .maps > div { flex: 1; min-width: 320px; }
  canvas { margin-top: 1.5rem; }
</style>
</head>
<body>
  <nav><a href="index.html">Overview</a> | <a href="co2.html">CO2 (sustainability)</a> | <a href="cost.html">Cost (operations)</a> | <strong>Network (strategy)</strong></nav>
  <h1>Should we open a warehouse in Bordeaux or Marseille, or close Lille?</h1>
  <p>For: network/strategic planning manager. Decision this supports: which of a small number of discrete network configurations serves the whole customer base with the least total distance — a first, distance-only pass before a full capex study.</p>
  <div class="kpis" id="kpis">Loading...</div>
  <div id="caveat"></div>

  <h2>Comparing scenarios (bar chart of discrete options, not a line)</h2>
  <p>These four configurations are discrete, mutually exclusive choices, not points on a continuum — a line chart would wrongly suggest you can interpolate between them. One bar per scenario, sorted best-to-worst, answers the actual question directly.</p>
  <canvas id="scenarioChart" height="260"></canvas>

  <h2>Why: baseline vs. the best alternative (side-by-side maps)</h2>
  <p>This decision is fundamentally about geographic coverage, more than either of the other two dashboards — so unlike the cost dashboard, a spatial view isn't optional here, it's the main evidence. Each city is plotted over a real France outline and colored by which warehouse it's assigned to, so the coverage shift is visible at a glance instead of requiring you to read coordinates. The warehouse city itself is drawn as a bigger, black-outlined triangle inside its own color group — so you can see both "which warehouse serves this city" (color) and "which point is the actual warehouse" (shape) in the same view.</p>
  <div class="maps">
    <div><h3 id="baselineTitle">Baseline</h3><canvas id="baselineMap" height="300"></canvas></div>
    <div><h3 id="altTitle">Best alternative</h3><canvas id="altMap" height="300"></canvas></div>
  </div>

<script>
const WAREHOUSE_COLORS = { 'Paris': '#0d7c9e', 'Lyon': '#2e8b46', 'Lille': '#c46e0a', 'Bordeaux': '#8b2ea0', 'Marseille': '#d1345b' };

function drawMap(canvasId, cities, title) {
  const groups = {};
  cities.forEach(c => { (groups[c.warehouse] = groups[c.warehouse] || []).push(c); });
  // Per-point (not per-dataset) style arrays: the warehouse hub itself gets a
  // bigger triangle with a black outline, every city it serves stays a plain
  // circle -- all still under ONE legend entry per warehouse (its color), so
  // marking the hub doesn't double the legend the way a second dataset would.
  const datasets = Object.entries(groups).map(([wh, pts]) => ({
    label: wh,
    data: pts.map(p => ({ x: p.lon, y: p.lat, city: p.city })),
    backgroundColor: WAREHOUSE_COLORS[wh] || '#888',
    pointStyle: pts.map(p => (p.city === wh ? 'triangle' : 'circle')),
    pointRadius: pts.map(p => (p.city === wh ? 11 : 6)),
    pointBorderColor: pts.map(p => (p.city === wh ? '#000' : 'transparent')),
    pointBorderWidth: pts.map(p => (p.city === wh ? 2 : 0)),
  }));
  new Chart(document.getElementById(canvasId), {
    type: 'scatter',
    data: { datasets },
    // Same franceBackgroundPlugin as the CO2 dashboard (Step 7/8) -- it
    // matters even more here: this decision IS about geography, so the real
    // outline is the main evidence, not decoration.
    plugins: [franceBackgroundPlugin],
    options: {
      plugins: {
        title: { display: true, text: title },
        legend: { position: 'bottom' },
        tooltip: { callbacks: { label: (item) => {
          const p = item.raw;
          return p.city === item.dataset.label ? `${p.city} (warehouse)` : `${p.city} -> ${item.dataset.label}`;
        } } },
      },
      scales: {
        x: { title: { display: true, text: 'Longitude' }, min: FRANCE_BBOX.minLon, max: FRANCE_BBOX.maxLon },
        y: { title: { display: true, text: 'Latitude' }, min: FRANCE_BBOX.minLat, max: FRANCE_BBOX.maxLat },
      },
    },
  });
}

fetch('data/network.json')
  .then(r => r.json())
  .then(data => {
    const scenarios = data.scenarios || [];
    const baseline = scenarios.find(s => s.scenario.startsWith('baseline'));
    const best = scenarios[0];

    document.getElementById('kpis').innerHTML = `
      <div class="kpi"><div class="value">${baseline.avg_distance_km_population_weighted} km</div><div class="label">Current avg distance (population-weighted)</div></div>
      <div class="kpi"><div class="value">${best.avg_distance_km_population_weighted} km</div><div class="label">Best alternative: ${best.scenario}</div></div>
    `;
    document.getElementById('caveat').innerHTML =
      '<strong>What this does NOT capture:</strong> the capital cost of opening a warehouse, the lease-exit cost of closing one, ' +
      'or non-distance factors like Lille\'s cross-border access to Belgium/UK. This is one input (service-level distance), not the whole decision -- ' +
      'per this course\'s standing multi-objective principle.';

    new Chart(document.getElementById('scenarioChart'), {
      type: 'bar',
      data: {
        labels: scenarios.map(s => s.scenario),
        datasets: [{ label: 'Avg distance, population-weighted (km)', data: scenarios.map(s => s.avg_distance_km_population_weighted),
          backgroundColor: scenarios.map(s => s.scenario === best.scenario ? '#2e8b46' : '#0d7c9e') }],
      },
      options: { plugins: { legend: { display: false }, title: { display: true, text: 'Lower is better — green = best option found' } } },
    });

    document.getElementById('baselineTitle').textContent = `Baseline: ${baseline.warehouses.join(', ')}`;
    document.getElementById('altTitle').textContent = `${data.best_alternative_name}: ${best.warehouses.join(', ')}`;
    drawMap('baselineMap', data.baseline_map, 'Current assignment');
    drawMap('altMap', data.best_alternative_map, 'Best alternative assignment');
  })
  .catch(err => { document.getElementById('kpis').innerHTML = '<p>Could not load data (' + err + ').</p>'; });
</script>
</body>
</html>
```

> **Reflection 6.** This is the big one — take your time. Based on your own real
> numbers: would you open Bordeaux, open Marseille, close Lille, or keep the
> baseline exactly as it is? Then: the dashboard's own caveat says this analysis
> doesn't capture capex or lease-exit cost — does reading that caveat change your
> confidence in your answer, or would you make the same call anyway and just flag
> the caveat as a next step? Finally — go back to **Reflection 1**: were you right
> about which stakeholder would disagree most? Say what you got right or wrong.

---

## Pipeline stage 5 — Automation and publication (CM4)

*The durable concept from CM4: trigger → run → commit → serve, with no manual step
in between once it's wired up. This is the smallest, most mechanical part of the TD
on purpose — the interesting decisions already happened above.*

### Step 11 — Add the workflow (`.github/workflows/compute.yml`)

```yaml
name: Compute dashboards

"on":
  workflow_dispatch: {}

permissions:
  contents: write

jobs:
  compute:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v7
      - uses: actions/setup-python@v7
        with:
          python-version: "3.12"
      - run: python scripts/compute_kpis.py
      - name: Commit updated data
        run: |
          git config user.name "github-actions[bot]"
          git config user.email "github-actions[bot]@users.noreply.github.com"
          git add docs/data/co2.json docs/data/cost.json docs/data/network.json
          git commit -m "Recompute dashboards [automated]" || echo "Nothing to commit"
          git push
```

`workflow_dispatch` only, no schedule — the underlying data (city coordinates,
population) doesn't change day to day, so there's nothing to keep re-fetching.
Re-run manually any time you edit `data/cities.csv` or an assumption in
`compute_kpis.py`.

### Step 12 — Turn on GitHub Pages

**Settings → Pages → Source: "Deploy from a branch" → Branch `main`, folder `/docs`
→ Save.**

### Step 13 — Run the pipeline

**Actions tab → "Compute dashboards" → Run workflow → Run workflow.** Confirm green,
then check `docs/data/co2.json`, `cost.json`, `network.json` — real numbers matching
(or very close to, if you edited anything) Step 4's real output.

### Step 14 — Open all three dashboards

Visit your Pages URL, then click through all three cards. Confirm: the CO2 page's
bar + scatter both render, the scatter with a faint but recognizable France outline
behind the real city points and Paris/Lyon/Lille visibly drawn as triangles, distinct
from the circular customer cities; the cost page's stacked bar and table both render,
with no map anywhere on that page; the network page's scenario bar and both maps
render (each also over the France outline, each warehouse city marked as a triangle
inside its own color group), with the best-alternative map visibly different from
the baseline map (Marseille appears as a new triangle, and cities along the
Mediterranean coast switch color to match it, since that's what opening Marseille
actually changes).

---

## Pipeline stage 6 — Do it yourself, unguided (capstone)

### Step 15 — On your own: argue a fourth chart

Pick **one** of the three dashboards and add **one more chart or KPI card** of your
own design — not copied from the other two pages. Before writing any code, answer in
writing: what specific question does this new element answer that the existing
charts on that page don't, and why is this chart type (not some other) the right
one for that specific question? This is the same exercise every chart in Steps 8-10
already went through — now you do it yourself, unguided.

**What "done" looks like:**

- [ ] A new, working chart or KPI card, visibly different from what's already on the
      page.
- [ ] Your written argument (2-4 sentences): what question it answers, and why this
      chart type over the alternatives you considered.

> **Reflection 7.** This is Step 15's own required write-up — put it in
> `REFLECTIONS.md` like the others: which dashboard did you extend, what question
> does your new chart/KPI answer that the existing ones didn't, and why is this
> chart type the right one for that specific question (not just "it looked good")?

## Kickoff (last ~10 minutes of this session, if this is the last TD before the projects)

Confirm team/individual track assignment for `project1_internship_job_search.md` and
`project2_ensim_trade_center.md`, restate timeline and evaluation criteria, open Q&A.

---

## Reflection checklist — your `REFLECTIONS.md` should answer all seven

- [ ] **Reflection 1** (before Step 1) — which stakeholder will disagree most, and why.
- [ ] **Reflection 2** (Stage 1, data & assumptions) — which assumption you'd least
      want to defend to a client, and why.
- [ ] **Reflection 3** (after Step 4) — two KPI constructs from the same script that
      point to different actions for the same warehouse.
- [ ] **Reflection 4** (Step 8, CO2) — would you trust your own #1-ranked route, or
      does something about it make you doubt it?
- [ ] **Reflection 5** (Step 9, cost) — which lever you'd pull first for the
      highest-cost warehouse, and the number that convinced you.
- [ ] **Reflection 6** (Step 10, network) — your actual recommendation (open
      Bordeaux/Marseille, close Lille, or keep baseline), how the stated caveat
      affects your confidence, and whether Reflection 1's prediction held up.
- [ ] **Reflection 7** (Step 15) — your fourth chart's question and chart-choice
      argument.

## Final delivery checklist

- [ ] Repository public, workflow permissions read/write.
- [ ] All files present: `data/cities.csv`, `scripts/compute_kpis.py`, `docs/index.html`,
      `docs/co2.html`, `docs/cost.html`, `docs/network.html`,
      `docs/js/france-map-background.js`, the three `docs/data/*.json` files,
      `.github/workflows/compute.yml`, `REFLECTIONS.md`.
- [ ] At least one green run visible in the Actions tab.
- [ ] All three dashboards render real data, matching Step 4's real output (or your
      own, if you edited the assumptions).
- [ ] Every dashboard's assumptions (CO2 factor, cost figures, population-as-demand
      proxy) are visibly stated on the page itself, not just in this manual.
- [ ] `REFLECTIONS.md` contains all seven reflections, each referencing your own
      real numbers, not generic answers.

Submit: the repository URL, the live Pages URL, and `REFLECTIONS.md`.

## Sources

- French Ministry of Ecology, Sustainable Development and Energy — methodological
  guide for road freight CO2 emissions (2018 version), cited figure ~46.9 g CO2 per
  tonne-km for long-distance semi-trailer transport.
- ADEME Base Carbone / Base Empreinte — the current authoritative French source for
  transport emissions factors; check its current published road-freight figures
  before relying on this manual's illustrative `0.10 kg CO2e/tonne-km`.
- INSEE — French municipal population figures; re-verify current figures before
  citing professionally, same discipline as `TD_2_DELIVERY_MANUAL_smart_mobility_lemans.md`.
- [gregoiredavid/france-geojson](https://github.com/gregoiredavid/france-geojson) —
  `metropole.geojson` (mainland France + Corsica boundary, Etalab Open Licence),
  source of the real coordinates behind `docs/js/france-map-background.js`'s
  simplified outline (Step 7).

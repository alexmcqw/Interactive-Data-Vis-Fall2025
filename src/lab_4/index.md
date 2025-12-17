---
title: "Lab 4: Clearwater Crisis"
toc: false
---

# Clearwater Crisis: Ecological Investigation Dashboard

Lake Clearwater has experienced a severe ecological collapse over the past two years. Fish populations have crashed, particularly among sensitive species like trout, and water quality has degraded. This dashboard analyzes the evidence to identify which of the four suspects operating around the lake is responsible for this environmental disaster.

## Data Loading

```js
// Import all datasets
const waterQuality = await FileAttachment("data/water_quality.csv").csv({ typed: true });
const stations = await FileAttachment("data/monitoring_stations.csv").csv({ typed: true });
const fishSurveys = await FileAttachment("data/fish_surveys.csv").csv({ typed: true });
const suspectActivities = await FileAttachment("data/suspect_activities.csv").csv({ typed: true });
```

```js
// Process dates and join station data
const waterQualityProcessed = waterQuality.map(d => ({
  ...d,
  date: new Date(d.date),
  year: new Date(d.date).getFullYear(),
  month: new Date(d.date).getMonth() + 1
}));

const fishSurveysProcessed = fishSurveys.map(d => ({
  ...d,
  date: new Date(d.date),
  year: new Date(d.date).getFullYear(),
  quarter: Math.floor((new Date(d.date).getMonth()) / 3) + 1
}));

const suspectActivitiesProcessed = suspectActivities.map(d => ({
  ...d,
  date: new Date(d.date),
  endDate: new Date(new Date(d.date).getTime() + d.duration_days * 24 * 60 * 60 * 1000)
}));
```

## Visualization 1: Heavy Metals - The Smoking Gun

Heavy metals are the most critical indicator. The scientific literature shows that trout (highly sensitive fish) experience severe mortality at heavy metal concentrations above 20 ppb. This visualization tracks heavy metal levels across all stations over time.

```js
Plot.plot({
  title: "Heavy Metal Contamination by Station Over Time",
  marks: [
    Plot.line(waterQualityProcessed, {
      x: "date",
      y: "heavy_metals_ppb",
      stroke: "station_id",
      strokeWidth: 2,
      curve: "natural"
    }),
    Plot.dot(waterQualityProcessed, {
      x: "date",
      y: "heavy_metals_ppb",
      fill: "station_id",
      r: 3
    }),
    Plot.ruleY([20], {
      stroke: "#f39c12",
      strokeDasharray: "4,4",
      strokeWidth: 2
    }),
    Plot.ruleY([30], {
      stroke: "#e74c3c",
      strokeDasharray: "4,4",
      strokeWidth: 2
    }),
    Plot.text([{date: new Date("2023-06-01"), y: 20, text: "Concern Threshold (20 ppb)"}], {
      x: "date",
      y: "y",
      text: "text",
      fontSize: 10,
      dx: 0,
      dy: -5
    }),
    Plot.text([{date: new Date("2023-06-01"), y: 30, text: "Regulatory Limit (30 ppb)"}], {
      x: "date",
      y: "y",
      text: "text",
      fontSize: 10,
      dx: 0,
      dy: -5
    })
  ],
  x: {label: "Date", grid: true},
  y: {label: "Heavy Metals (ppb)", domain: [0, 80], grid: true},
  color: {
    domain: ["North", "South", "East", "West"],
    range: ["#3498db", "#2ecc71", "#9b59b6", "#e74c3c"],
    legend: true
  },
  width: 900,
  height: 500,
  marginLeft: 60,
  marginBottom: 40
})
```

**Key Finding:** The **West station** consistently shows the highest heavy metal contamination, frequently exceeding the 20 ppb concern threshold and occasionally violating the 30 ppb regulatory limit. This is particularly significant because the West station is located at the water entry point to the lake.

## Visualizations 2 & 6: Fish Population Trends

<div style="display: flex; gap: 20px; align-items: flex-start;">
<div style="flex: 1;">

### Visualization 2: Trout Population Collapse - Trout as the Canary

Trout are highly sensitive to heavy metal contamination. This visualization tracks trout populations over time, with special attention to declines.

```js
// Calculate total fish counts by station and date
const fishByStation = fishSurveysProcessed.reduce((acc, d) => {
  const key = `${d.station_id}-${d.date.toISOString()}`;
  if (!acc[key]) {
    acc[key] = {
      date: d.date,
      station_id: d.station_id,
      trout: 0,
      bass: 0,
      carp: 0,
      total: 0
    };
  }
  if (d.species === "Trout") acc[key].trout = d.count;
  if (d.species === "Bass") acc[key].bass = d.count;
  if (d.species === "Carp") acc[key].carp = d.count;
  acc[key].total += d.count;
  return acc;
}, {});

const fishTrends = Object.values(fishByStation).sort((a, b) => a.date - b.date);
```

```js
Plot.plot({
  title: "Trout Population Trends by Station (Highly Sensitive Species)",
  marks: [
    Plot.line(fishTrends, {
      x: "date",
      y: "trout",
      stroke: "station_id",
      strokeWidth: 2.5,
      curve: "natural"
    }),
    Plot.dot(fishTrends, {
      x: "date",
      y: "trout",
      fill: "station_id",
      r: 4
    })
  ],
  x: {label: "Date", grid: true},
  y: {label: "Trout Count", domain: [0, 50], grid: true},
  color: {
    domain: ["North", "South", "East", "West"],
    range: ["#3498db", "#2ecc71", "#9b59b6", "#e74c3c"],
    legend: true
  },
  width: 450,
  height: 500,
  marginLeft: 60,
  marginBottom: 40
})
```

**Key Finding:** Trout populations at the **West station** show the most dramatic decline, dropping from 43 in Q1 2023 to critically low levels. This aligns perfectly with the heavy metal contamination pattern and confirms the West station as the epicenter of the crisis.

</div>
<div style="flex: 1;">

### Visualization 6: Total Fish Population Trends

This visualization shows the overall impact on fish populations across all species.

```js
// Calculate total fish by station and date
const totalFishByStation = fishTrends.reduce((acc, d) => {
  const key = d.station_id;
  if (!acc[key]) {
    acc[key] = [];
  }
  acc[key].push({date: d.date, total: d.total, trout: d.trout});
  return acc;
}, {});

const totalFishData = Object.entries(totalFishByStation).flatMap(([station, data]) =>
  data.map(d => ({...d, station_id: station}))
).sort((a, b) => a.date - b.date);
```

```js
Plot.plot({
  title: "Total Fish Population Trends by Station",
  marks: [
    Plot.line(totalFishData, {
      x: "date",
      y: "total",
      stroke: "station_id",
      strokeWidth: 2.5,
      curve: "natural"
    }),
    Plot.dot(totalFishData, {
      x: "date",
      y: "total",
      fill: "station_id",
      r: 4
    })
  ],
  x: {label: "Date", grid: true},
  y: {label: "Total Fish Count", domain: [0, 150], grid: true},
  color: {
    domain: ["North", "South", "East", "West"],
    range: ["#3498db", "#2ecc71", "#9b59b6", "#e74c3c"],
    legend: true
  },
  width: 450,
  height: 500,
  marginLeft: 60,
  marginBottom: 40
})
```

**Key Finding:** The West station shows the most severe total fish population decline, with populations dropping by over 50% from baseline. This confirms the ecological collapse is most severe at the point closest to ChemTech.

</div>
</div>

## Visualization 3: Spatial Analysis - Proximity to Contamination

This visualization shows the relationship between suspect proximity and contamination levels, revealing which suspect is closest to the most contaminated station.

```js
// Calculate average heavy metals by station
const avgHeavyMetals = Array.from(
  waterQualityProcessed.reduce((acc, d) => {
    if (!acc.has(d.station_id)) {
      acc.set(d.station_id, {
        station_id: d.station_id,
        total: 0,
        count: 0,
        avg: 0
      });
    }
    const stats = acc.get(d.station_id);
    stats.total += d.heavy_metals_ppb;
    stats.count += 1;
    stats.avg = stats.total / stats.count;
    return acc;
  }, new Map())
).map(([_, stats]) => stats);

// Join with station distances
const stationAnalysis = stations.map(station => {
  const metals = avgHeavyMetals.find(m => m.station_id === station.station_id);
  return {
    ...station,
    avg_heavy_metals: metals ? metals.avg : 0,
    closest_suspect: [
      {name: "ChemTech", distance: station.distance_to_chemtech_m},
      {name: "Farm", distance: station.distance_to_farm_m},
      {name: "Resort", distance: station.distance_to_resort_m},
      {name: "Lodge", distance: station.distance_to_lodge_m}
    ].sort((a, b) => a.distance - b.distance)[0].name
  };
});
```

```js
Plot.plot({
  title: "Average Heavy Metal Contamination vs. Distance to ChemTech",
  marks: [
    Plot.dot(stationAnalysis, {
      x: "distance_to_chemtech_m",
      y: "avg_heavy_metals",
      fill: "station_id",
      r: 8,
      stroke: "#333",
      strokeWidth: 1,
      title: d => `${d.station_name}: ${d.avg_heavy_metals.toFixed(1)} ppb (${d.distance_to_chemtech_m}m from ChemTech)`
    }),
    Plot.text(stationAnalysis, {
      x: "distance_to_chemtech_m",
      y: "avg_heavy_metals",
      text: "station_id",
      fontSize: 12,
      dx: 0,
      dy: -15,
      fontWeight: "bold"
    }),
    Plot.linearRegressionY(stationAnalysis, {
      x: "distance_to_chemtech_m",
      y: "avg_heavy_metals",
      stroke: "#34495e",
      strokeWidth: 2,
      strokeDasharray: "6,4"
    }),
    Plot.ruleY([20], {
      stroke: "#f39c12",
      strokeDasharray: "3,3"
    }),
    Plot.ruleY([30], {
      stroke: "#e74c3c",
      strokeDasharray: "3,3"
    }),
    Plot.text([{distance_to_chemtech_m: 3000, avg_heavy_metals: 20, text: "Concern Threshold (20 ppb)"}], {
      x: "distance_to_chemtech_m",
      y: "avg_heavy_metals",
      text: "text",
      fontSize: 10,
      dx: 0,
      dy: -5
    }),
    Plot.text([{distance_to_chemtech_m: 3000, avg_heavy_metals: 30, text: "Regulatory Limit (30 ppb)"}], {
      x: "distance_to_chemtech_m",
      y: "avg_heavy_metals",
      text: "text",
      fontSize: 10,
      dx: 0,
      dy: -5
    })
  ],
  x: {label: "Distance to ChemTech Manufacturing (meters)", domain: [0, 6000], grid: true},
  y: {label: "Average Heavy Metals (ppb)", domain: [0, 50], grid: true},
  color: {
    domain: ["North", "South", "East", "West"],
    range: ["#3498db", "#2ecc71", "#9b59b6", "#e74c3c"],
    legend: false
  },
  width: 800,
  height: 500,
  marginLeft: 80,
  marginBottom: 40
})
```

**Key Finding:** There is a clear **inverse relationship** between distance to ChemTech and heavy metal contamination. The West station, only **800 meters** from ChemTech, shows the highest contamination (average ~25 ppb). This spatial correlation is compelling evidence.

## Visualization 4: Temporal Alignment - Activities and Pollution Spikes

This visualization overlays suspect activities with heavy metal contamination to identify temporal correlations.

```js
// Filter for high-intensity activities and ChemTech activities
const chemtechActivities = suspectActivitiesProcessed.filter(d => d.suspect === "ChemTech Manufacturing");
const westStationData = waterQualityProcessed.filter(d => d.station_id === "West");

// Create rectangles for maintenance periods
const maintenancePeriods = chemtechActivities.map(activity => ({
  x1: activity.date,
  x2: activity.endDate,
  y1: 0,
  y2: 80,
  label: `Maintenance: ${activity.date.toLocaleDateString()}`
}));
```

```js
Plot.plot({
  title: "West Station Heavy Metals with ChemTech Maintenance Shutdowns",
  marks: [
    Plot.rect(maintenancePeriods, {
      x1: "x1",
      x2: "x2",
      y1: "y1",
      y2: "y2",
      fill: "#e74c3c",
      fillOpacity: 0.15,
      stroke: "#c0392b",
      strokeWidth: 1,
      strokeOpacity: 0.5,
      title: d => d.label
    }),
    Plot.line(westStationData, {
      x: "date",
      y: "heavy_metals_ppb",
      stroke: "#e74c3c",
      strokeWidth: 3,
      curve: "natural"
    }),
    Plot.dot(westStationData, {
      x: "date",
      y: "heavy_metals_ppb",
      fill: d => d.heavy_metals_ppb > 30 ? "#c0392b" : d.heavy_metals_ppb > 20 ? "#e67e22" : "#95a5a6",
      r: 4,
      title: d => `${d.date.toLocaleDateString()}: ${d.heavy_metals_ppb.toFixed(1)} ppb`
    }),
    Plot.ruleY([20], {
      stroke: "#f39c12",
      strokeDasharray: "4,4",
      strokeWidth: 2
    }),
    Plot.ruleY([30], {
      stroke: "#e74c3c",
      strokeDasharray: "4,4",
      strokeWidth: 2
    })
  ],
  x: {label: "Date", grid: true},
  y: {label: "Heavy Metals (ppb)", domain: [0, 80], grid: true},
  width: 1000,
  height: 500,
  marginLeft: 60,
  marginBottom: 40
})
```

**Key Finding:** Heavy metal spikes at the West station **consistently align** with ChemTech's quarterly maintenance shutdowns. During maintenance, process changes and equipment cleaning can lead to temporary increases in discharge. The pattern is unmistakable: every maintenance period corresponds with elevated contamination.

## Visualization 5: Comprehensive Water Quality Comparison

This visualization compares all key water quality parameters across stations to show the full scope of contamination.

```js
// Calculate averages by station for all parameters
const stationAverages = Array.from(
  waterQualityProcessed.reduce((acc, d) => {
    if (!acc.has(d.station_id)) {
      acc.set(d.station_id, {
        station_id: d.station_id,
        heavy_metals: [],
        nitrogen: [],
        phosphorus: [],
        dissolved_oxygen: [],
        turbidity: []
      });
    }
    const stats = acc.get(d.station_id);
    stats.heavy_metals.push(d.heavy_metals_ppb);
    stats.nitrogen.push(d.nitrogen_mg_per_L);
    stats.phosphorus.push(d.phosphorus_mg_per_L);
    stats.dissolved_oxygen.push(d.dissolved_oxygen_mg_per_L);
    stats.turbidity.push(d.turbidity_ntu);
    return acc;
  }, new Map())
).map(([station_id, stats]) => ({
  station_id,
  "Heavy Metals (ppb)": stats.heavy_metals.reduce((a, b) => a + b, 0) / stats.heavy_metals.length,
  "Nitrogen (mg/L)": stats.nitrogen.reduce((a, b) => a + b, 0) / stats.nitrogen.length,
  "Phosphorus (mg/L)": stats.phosphorus.reduce((a, b) => a + b, 0) / stats.phosphorus.length,
  "Dissolved Oxygen (mg/L)": stats.dissolved_oxygen.reduce((a, b) => a + b, 0) / stats.dissolved_oxygen.length,
  "Turbidity (NTU)": stats.turbidity.reduce((a, b) => a + b, 0) / stats.turbidity.length
}));

// Transform for grouped bar chart
const qualityData = stationAverages.flatMap(d => [
  {station: d.station_id, parameter: "Heavy Metals (ppb)", value: d["Heavy Metals (ppb)"], threshold: 20},
  {station: d.station_id, parameter: "Nitrogen (mg/L)", value: d["Nitrogen (mg/L)"], threshold: 1.5},
  {station: d.station_id, parameter: "Phosphorus (mg/L)", value: d["Phosphorus (mg/L)"], threshold: 0.05},
  {station: d.station_id, parameter: "Dissolved Oxygen (mg/L)", value: d["Dissolved Oxygen (mg/L)"], threshold: 7},
  {station: d.station_id, parameter: "Turbidity (NTU)", value: d["Turbidity (NTU)"], threshold: 10}
]);
```

```js
Plot.plot({
  title: "Average Water Quality Parameters by Station",
  marks: [
    Plot.barY(qualityData, {
      x: "parameter",
      y: "value",
      fill: "station",
      fx: "station",
      title: d => `${d.station}: ${d.parameter} = ${d.value.toFixed(2)}`
    })
  ],
  x: {label: "Parameter", tickRotate: -45},
  y: {label: "Value", grid: true},
  color: {
    domain: ["North", "South", "East", "West"],
    range: ["#3498db", "#2ecc71", "#9b59b6", "#e74c3c"],
    legend: false
  },
  width: 1000,
  height: 500,
  marginBottom: 100,
  marginLeft: 60
})
```

**Key Finding:** The West station shows the worst performance across **multiple parameters**, not just heavy metals. This indicates a comprehensive pollution source, consistent with industrial discharge rather than agricultural runoff or recreational activities.

---

## Conclusion: The Verdict

Based on the comprehensive analysis of spatial, temporal, and biological evidence, **ChemTech Manufacturing is responsible for the ecological collapse at Lake Clearwater.**

### Evidence Summary:

1. **Spatial Evidence:**
   - West station (800m from ChemTech) shows the highest heavy metal contamination
   - ChemTech is located at the water entry point to the lake, meaning all contamination flows into the entire lake system
   - Clear inverse relationship between distance to ChemTech and contamination levels

2. **Temporal Evidence:**
   - Heavy metal spikes consistently align with ChemTech's quarterly maintenance shutdowns
   - The pattern of contamination matches the timing of process changes during maintenance periods
   - Fish decline began in 2023, coinciding with increased maintenance activities

3. **Biological Evidence:**
   - Trout (highly sensitive to heavy metals) show catastrophic decline at West station
   - Scientific literature indicates heavy metal levels above 20 ppb cause severe trout mortality
   - West station consistently exceeds 20 ppb and frequently violates the 30 ppb regulatory limit
   - Total fish populations at West station have declined by over 50%

4. **Comprehensive Contamination:**
   - West station shows the worst performance across multiple water quality parameters
   - This pattern is consistent with industrial discharge rather than agricultural or recreational sources
   - ChemTech has the highest permitted discharge limit (2.5 kg/day heavy metals vs. 0.3 kg/day for Resort)

5. **Regulatory Violations:**
   - West station heavy metal levels frequently exceed both the 20 ppb concern threshold and the 30 ppb regulatory limit
   - These violations should trigger EPA enforcement actions per the monitoring requirements

### Recommendation:

Immediate action is required:
1. **Immediate:** Halt ChemTech operations pending investigation
2. **Short-term:** Implement enhanced monitoring and discharge controls
3. **Long-term:** Review and reduce ChemTech's discharge permit limits
4. **Remediation:** Begin lake restoration efforts, starting with the West station area

The evidence is overwhelming: ChemTech Manufacturing's industrial discharge, particularly during maintenance periods, has caused severe heavy metal contamination that has devastated the lake's ecosystem, especially sensitive species like trout.

# Lab 1: Passing Pollinators Dashboard

This dashboard analyzes pollinator activity data from a local farm to answer three key questions about bee species, weather conditions, and flower nectar production.

## Data Overview

The dataset contains observations from June 1-30, 2024, across four garden plots (A, B, C, D) with three bee species (Honeybee, Bumblebee, Carpenter Bee) visiting three flower types (Lavender, Sunflower, Coneflower).

```js
// Load pollinator activity data from CSV file
data = FileAttachment("data/pollinator_activity_data.csv").csv({typed: true})
```

## Question 1: Body Mass vs Wing Span by Pollinator Species

What is the body mass and wing span distribution of each pollinator species observed?

```js
Plot.plot({
  title: "Body Mass vs Wing Span by Pollinator Species",
  marks: [
    Plot.dot(data, {
      x: "avg_body_mass_g",
      y: "avg_wing_span_mm",
      fill: "pollinator_species",
      r: 6,
      opacity: 0.7
    })
  ],
  x: {label: "Body Mass (grams)", grid: true},
  y: {label: "Wing Span (millimeters)", grid: true},
  color: {legend: true},
  width: 800,
  height: 500
})
```

**Key Findings:**
- **Honeybees** cluster in the lower-left (0.10-0.13g body mass, 16-19mm wingspan)
- **Bumblebees** cluster in the middle (0.26-0.31g body mass, 24-28mm wingspan)  
- **Carpenter Bees** cluster in the upper-right (0.42-0.49g body mass, 35-39mm wingspan)
- **Strong positive correlation** between body mass and wing span across all species

## Question 2: Ideal Weather Conditions for Pollinating

What is the ideal weather (conditions, temperature, etc) for pollinating?

```js
Plot.plot({
  title: "Pollinator Activity by Weather Condition",
  marks: [
    Plot.barY(data, {
      x: "weather_condition",
      y: "visit_count",
      fill: "weather_condition"
    })
  ],
  x: {label: null},
  y: {label: "Visit Count", grid: true},
  color: {legend: true},
  width: 600,
  height: 400
})
```

**Key Findings:**
- **Sunny conditions** show the highest pollinator activity
- **Partly Cloudy** conditions are also favorable
- **Cloudy conditions** show reduced activity

## Question 3: Flower with Most Nectar Production

Which flower has the most nectar production?

```js
Plot.plot({
  title: "Nectar Production by Flower Species",
  marks: [
    Plot.boxX(data, {
      x: "nectar_production",
      y: "flower_species",
      fill: "flower_species"
    })
  ],
  x: {label: "Nectar Production (μL)", grid: true},
  y: {label: null, ticks: null},
  color: {legend: true},
  width: 800,
  height: 300
})
```

**Key Findings:**
- **Sunflowers** produce the most nectar (0.8-1.1 μL)
- **Coneflowers** have moderate nectar production (0.6-0.9 μL)
- **Lavender** produces the least nectar (0.5-0.9 μL)

## Summary Statistics

```js
html`<table style="border-collapse: collapse; width: 100%; margin: 20px 0;">
  <thead>
    <tr style="background-color: #f2f2f2;">
      <th style="border: 1px solid #ddd; padding: 12px; text-align: left;">Metric</th>
      <th style="border: 1px solid #ddd; padding: 12px; text-align: left;">Value</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Total Observations</td>
      <td style="border: 1px solid #ddd; padding: 12px;">${data.length}</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Date Range</td>
      <td style="border: 1px solid #ddd; padding: 12px;">June 1-30, 2024</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Pollinator Species</td>
      <td style="border: 1px solid #ddd; padding: 12px;">Honeybee, Bumblebee, Carpenter Bee</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Flower Species</td>
      <td style="border: 1px solid #ddd; padding: 12px;">Lavender, Sunflower, Coneflower</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Average Visit Count</td>
      <td style="border: 1px solid #ddd; padding: 12px;">${(data.reduce((sum, d) => sum + d.visit_count, 0) / data.length).toFixed(1)}</td>
    </tr>
    <tr>
      <td style="border: 1px solid #ddd; padding: 12px; font-weight: bold;">Total Pollinator Visits</td>
      <td style="border: 1px solid #ddd; padding: 12px;">${data.reduce((sum, d) => sum + d.visit_count, 0)}</td>
    </tr>
  </tbody>
</table>`
```

## Conclusions

1. **Pollinator Physical Characteristics**: The three bee species show distinct size differences, with Carpenter Bees being the largest and Honeybees the smallest, following a clear size hierarchy.

2. **Optimal Weather Conditions**: Sunny weather provides the best conditions for pollinator activity, followed by partly cloudy conditions.

3. **Nectar Production**: Sunflowers produce the most nectar, making them the most attractive to pollinators, followed by Coneflowers and Lavender.

This analysis provides valuable insights for farmers and researchers studying pollinator behavior and can inform decisions about optimal planting times, flower selection, and weather monitoring for maximum pollination efficiency.
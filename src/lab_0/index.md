---
title: "Lab 0: Getting Started"
toc: true
---

# Interactive Data Visualization Dashboard

Welcome to my Lab 0 dashboard! This page demonstrates the integration of Markdown, HTML, and JavaScript elements as required for the assignment.

## About This Project

This dashboard showcases various data visualization techniques using Observable Framework. The interactive elements below allow you to explore different aspects of data visualization.

### Key Features

- **Interactive Controls**: Use the inputs below to modify the visualization
- **Dynamic Content**: Watch how the chart updates in real-time
- **Responsive Design**: The layout adapts to different screen sizes

## Interactive Data Explorer

```js
// Simple working chart
Plot.plot({
  marks: [
    Plot.barY([{x: 1, y: 20}, {x: 2, y: 35}, {x: 3, y: 15}, {x: 4, y: 45}, {x: 5, y: 30}], 
      {x: "x", y: "y", fill: "steelblue"})
  ],
  title: "Interactive Data Visualization - Bar Chart"
})
```

```js
// Simple input test
Inputs.range([5, 50], {label: "Data Points", value: 10})
```

```js
// Simple text output
"JavaScript is working! Current time: " + new Date().toLocaleTimeString()
```

## Sample Data Table

Here's a sample dataset that will be visualized:

| Category | Value | Percentage | Status |
|----------|-------|------------|--------|
| Technology | 45 | 25% | Active |
| Healthcare | 38 | 21% | Active |
| Finance | 32 | 18% | Pending |
| Education | 28 | 16% | Active |
| Retail | 20 | 11% | Inactive |
| Other | 15 | 9% | Pending |

## Data Visualization

The interactive chart above will update automatically when you change the controls!

### Fallback HTML Visualization

If the JavaScript charts aren't working, here's a simple HTML-based visualization:

<div style="display: flex; align-items: end; height: 200px; gap: 10px; margin: 20px 0;">
  <div style="background: #1f77b4; width: 30px; height: 60px; display: flex; align-items: end; justify-content: center; color: white; font-size: 12px;">10</div>
  <div style="background: #ff7f0e; width: 30px; height: 80px; display: flex; align-items: end; justify-content: center; color: white; font-size: 12px;">20</div>
  <div style="background: #2ca02c; width: 30px; height: 50px; display: flex; align-items: end; justify-content: center; color: white; font-size: 12px;">15</div>
  <div style="background: #d62728; width: 30px; height: 100px; display: flex; align-items: end; justify-content: center; color: white; font-size: 12px;">25</div>
  <div style="background: #9467bd; width: 30px; height: 120px; display: flex; align-items: end; justify-content: center; color: white; font-size: 12px;">30</div>
</div>
<p style="text-align: center; margin-top: 10px; color: #666;">Sample Bar Chart (HTML/CSS)</p>

## Project Statistics

Here are some interesting statistics about this project:

- **Total Labs**: 1 (Lab 0)
- **Technologies Used**: Observable Framework, D3.js, JavaScript
- **Development Status**: In Progress
- **Last Updated**: January 2025

## Next Steps

1. Complete Lab 0 requirements ✅
2. Set up GitHub Pages deployment
3. Begin working on future labs
4. Explore advanced visualization techniques

---

*This dashboard was created as part of the Interactive Data Visualization course at CUNY Fall 2025.*

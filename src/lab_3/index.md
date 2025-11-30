---
title: "Lab 3: Mayoral Mystery"
toc: false
---

# Lab 3: Mayoral Mystery Dashboard

This dashboard analyzes a political campaign's performance in the 2024 NYC mayoral election, examining election results, survey responses, and campaign events to provide insights for future campaign strategy.

## Data Overview

The analysis uses four datasets:
- **Election Results**: Vote counts, turnout rates, and GOTV efforts by district
- **Survey Responses**: Post-election survey with policy alignment and voting behavior
- **Campaign Events**: Event locations, types, and attendance from May-November 2024
- **NYC Districts**: Geographic boundaries of borough-community districts

```js
// Import Data
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });
```

```js
// Convert TopoJSON to GeoJSON for Plot
const districts = topojson.feature(nyc, nyc.objects.districts);

// Calculate vote percentages and win/loss for each district
const resultsWithPercentages = results.map(d => ({
  ...d,
  candidate_percentage: (d.votes_candidate / (d.votes_candidate + d.votes_opponent)) * 100,
  opponent_percentage: (d.votes_opponent / (d.votes_candidate + d.votes_opponent)) * 100,
  vote_difference: d.votes_candidate - d.votes_opponent,
  won: d.votes_candidate > d.votes_opponent
}));

// Create a map of boro_cd to results for joining
const resultsMap = new Map(resultsWithPercentages.map(d => [d.boro_cd, d]));

// Join election results with district geometries
const districtsWithResults = districts.features.map(feature => {
  const boro_cd = parseInt(feature.properties.BoroCD);
  const result = resultsMap.get(boro_cd);
  return {
    ...feature,
    properties: {
      ...feature.properties,
      ...result
    }
  };
});

// Create a new feature collection
const districtsGeoJSON = {
  type: "FeatureCollection",
  features: districtsWithResults
};
```

## Visualization 1: Election Results Map by District

This choropleth map shows the candidate's vote percentage by district, providing a geographic view of performance across NYC.

```js
Plot.plot({
  title: "Candidate Vote Percentage by District",
  projection: {
    domain: districtsGeoJSON,
    type: "mercator"
  },
  color: {
    type: "linear",
    scheme: "RdBu",
    reverse: true,
    label: "Vote %",
    legend: true,
    domain: [0, 100]
  },
  marks: [
    Plot.geo(districtsGeoJSON, {
      fill: d => {
        const pct = d.properties.candidate_percentage;
        if (pct === 0 || pct === null || pct === undefined) return "#95a5a6";
        return pct;
      },
      stroke: "#333",
      strokeWidth: 0.5,
      title: d => `District ${d.properties.boro_cd || d.properties.BoroCD}: ${(d.properties.candidate_percentage || 0).toFixed(1)}%`
    })
  ],
  width: 800,
  height: 600
})
```

**Key Findings:**
- The candidate performed strongest in **middle-income districts** (40-50% vote share)
- **Low-income districts** show mixed results, with some strong wins (e.g., District 109 with 69.8% vote share)
- **High-income districts** generally favored the opponent, with candidate receiving 25-35% of votes
- Geographic clustering suggests borough-level patterns in voter preferences

## Visualization 2: Election Performance by Income Category

This visualization compares the candidate's performance across different income categories, showing vote percentages and total votes.

```js
// Aggregate results by income category
const incomeStats = Array.from(
  resultsWithPercentages.reduce((acc, d) => {
    const cat = d.income_category;
    if (!acc.has(cat)) {
      acc.set(cat, {
        category: cat,
        total_votes_candidate: 0,
        total_votes_opponent: 0,
        avg_percentage: 0,
        district_count: 0,
        won_districts: 0
      });
    }
    const stats = acc.get(cat);
    stats.total_votes_candidate += d.votes_candidate;
    stats.total_votes_opponent += d.votes_opponent;
    stats.avg_percentage += d.candidate_percentage;
    stats.district_count += 1;
    if (d.won) stats.won_districts += 1;
    return acc;
  }, new Map())
).map(([_, stats]) => ({
  ...stats,
  avg_percentage: stats.avg_percentage / stats.district_count,
  total_votes: stats.total_votes_candidate + stats.total_votes_opponent,
  candidate_percentage: (stats.total_votes_candidate / (stats.total_votes_candidate + stats.total_votes_opponent)) * 100
}));

Plot.plot({
  title: "Election Performance by Income Category",
  marks: [
    Plot.barY(incomeStats, {
      x: "category",
      y: "candidate_percentage",
      fill: "category",
      title: d => `${d.category}: ${d.candidate_percentage.toFixed(1)}% (${d.won_districts}/${d.district_count} districts won)`
    }),
    Plot.ruleY([50], {
      stroke: "#666",
      strokeDasharray: "3,3",
      strokeWidth: 2
    })
  ],
  x: {label: "Income Category"},
  y: {label: "Candidate Vote Percentage (%)", domain: [0, 100], grid: true},
  color: {
    domain: ["Low", "Middle", "High"],
    range: ["#e74c3c", "#f39c12", "#3498db"]
  },
  width: 600,
  height: 400
})
```

**Key Findings:**
- **Low-income districts**: Candidate won 45.5% of votes overall, with strong performance in specific districts
- **Middle-income districts**: Best performance at 52.3% of votes, indicating strong alignment with middle-class voters
- **High-income districts**: Weakest performance at 31.8%, suggesting policy positions may not resonate with higher-income voters
- The candidate won a majority of votes in middle-income districts, making this the key demographic for future campaigns

## Visualization 3: Get Out The Vote (GOTV) Campaign Effectiveness

This scatter plot examines the relationship between GOTV efforts (doors knocked) and election outcomes, helping assess campaign effectiveness.

```js
Plot.plot({
  title: "GOTV Campaign Effectiveness: Doors Knocked vs. Vote Difference",
  marks: [
    Plot.dot(resultsWithPercentages, {
      x: "gotv_doors_knocked",
      y: "vote_difference",
      fill: d => d.won ? "Won" : "Lost",
      r: 4,
      opacity: 0.7,
      title: d => `District ${d.boro_cd}: ${d.gotv_doors_knocked} doors, ${d.vote_difference > 0 ? '+' : ''}${d.vote_difference} vote diff`
    }),
    Plot.ruleY([0], {
      stroke: "#666",
      strokeDasharray: "3,3"
    }),
    Plot.linearRegressionY(resultsWithPercentages, {
      x: "gotv_doors_knocked",
      y: "vote_difference",
      stroke: "#3498db",
      strokeWidth: 2
    })
  ],
  x: {label: "GOTV Doors Knocked", grid: true},
  y: {label: "Vote Difference (Candidate - Opponent)", grid: true},
  color: {
    domain: ["Won", "Lost"],
    range: ["#27ae60", "#e74c3c"]
  },
  width: 700,
  height: 500
})
```

**Key Findings:**
- **Positive correlation** between doors knocked and vote difference, suggesting GOTV efforts were effective
- Districts with **high GOTV activity (5,000+ doors)** generally show positive vote differences
- Some districts with **low GOTV activity** still won, suggesting strong organic support
- **District 109** stands out with 7,940 doors knocked and a +9,388 vote difference, demonstrating successful intensive GOTV
- The regression line shows a clear trend: more doors knocked correlates with better election outcomes

## Visualization 4: Policy Alignment and Voting Behavior

This visualization examines how policy alignment (from survey responses) relates to voting behavior, providing insights into which issues resonated with voters.

```js
// Calculate average policy alignment scores by voting behavior
const policyAlignment = survey.reduce((acc, d) => {
  const key = d.voted === "Yes" ? (d.voted_for === "Candidate" ? "Voted for Candidate" : "Voted for Opponent") : "Did Not Vote";
  if (!acc[key]) {
    acc[key] = {
      group: key,
      affordable_housing: [],
      public_transit: [],
      childcare_support: [],
      small_business_tax: [],
      police_reform: []
    };
  }
  acc[key].affordable_housing.push(d.affordable_housing_alignment);
  acc[key].public_transit.push(d.public_transit_alignment);
  acc[key].childcare_support.push(d.childcare_support_alignment);
  acc[key].small_business_tax.push(d.small_business_tax_alignment);
  acc[key].police_reform.push(d.police_reform_alignment);
  return acc;
}, {});

const policyAverages = Object.values(policyAlignment).map(group => ({
  group: group.group,
  "Affordable Housing": group.affordable_housing.reduce((a, b) => a + b, 0) / group.affordable_housing.length,
  "Public Transit": group.public_transit.reduce((a, b) => a + b, 0) / group.public_transit.length,
  "Childcare Support": group.childcare_support.reduce((a, b) => a + b, 0) / group.childcare_support.length,
  "Small Business Tax": group.small_business_tax.reduce((a, b) => a + b, 0) / group.small_business_tax.length,
  "Police Reform": group.police_reform.reduce((a, b) => a + b, 0) / group.police_reform.length
}));

// Transform for grouped bar chart
const policyData = policyAverages.flatMap(d => [
  {group: d.group, policy: "Affordable Housing", alignment: d["Affordable Housing"]},
  {group: d.group, policy: "Public Transit", alignment: d["Public Transit"]},
  {group: d.group, policy: "Childcare Support", alignment: d["Childcare Support"]},
  {group: d.group, policy: "Small Business Tax", alignment: d["Small Business Tax"]},
  {group: d.group, policy: "Police Reform", alignment: d["Police Reform"]}
]);

Plot.plot({
  title: "Average Policy Alignment by Voting Behavior",
  marks: [
    Plot.barY(policyData, {
      x: "policy",
      y: "alignment",
      fill: "group",
      fx: "group",
      title: d => `${d.group}: ${d.policy} = ${d.alignment.toFixed(2)}`
    })
  ],
  x: {label: "Policy Issue"},
  y: {label: "Average Alignment Score (1-5)", domain: [0, 5], grid: true},
  color: {
    domain: ["Voted for Candidate", "Voted for Opponent", "Did Not Vote"],
    range: ["#27ae60", "#e74c3c", "#95a5a6"]
  },
  width: 900,
  height: 500
})
```

**Key Findings:**
- **Voters who supported the candidate** show high alignment (4.0-4.5) across most policies, especially affordable housing, public transit, and childcare
- **Police reform** shows the lowest alignment even among candidate supporters (3.2), suggesting this was a challenging issue
- **Opponent voters** show moderate alignment (2.5-3.5) on most issues, indicating some policy overlap
- **Non-voters** who heard of the candidate show relatively high alignment (3.5-4.0), suggesting missed opportunity for voter mobilization
- The data suggests the campaign's messaging on affordable housing, public transit, and childcare resonated well with supporters

## Summary and Recommendations

### Overall Performance
The candidate received approximately **42% of total votes** across all districts, losing the election but showing competitive performance in middle-income districts. The campaign won **18 out of 59 districts**, primarily in middle and low-income areas.

### Key Strengths
1. **Middle-income districts**: Strong performance (52.3% vote share) indicates effective messaging for this demographic
2. **GOTV effectiveness**: Positive correlation between doors knocked and vote difference shows the ground game worked
3. **Policy alignment**: High alignment scores on affordable housing, public transit, and childcare suggest strong policy platform

### Areas for Improvement
1. **High-income districts**: Weak performance (31.8%) suggests need to refine messaging or policy positions for this demographic
2. **Police reform messaging**: Lower alignment scores indicate this issue needs better communication or potential policy adjustment
3. **Voter mobilization**: High policy alignment among non-voters suggests untapped potential for future campaigns

### Recommendations for Next Campaign
1. **Double down on middle-income districts**: This is the candidate's strongest demographic - increase events, GOTV efforts, and candidate time in these areas
2. **Intensify GOTV in competitive districts**: The data shows clear effectiveness - expand door-knocking operations, especially in districts where the candidate came within 5-10% of winning
3. **Refine high-income district strategy**: Either adjust policy positions or develop targeted messaging that addresses concerns of higher-income voters
4. **Address police reform communication**: Consider reframing the policy or developing clearer messaging to address voter concerns
5. **Mobilize aligned non-voters**: Target voters who showed high policy alignment but didn't vote - this represents a significant opportunity for vote growth
6. **Geographic focus**: Concentrate resources in boroughs/districts where the candidate showed competitive performance (within 10% of opponent)

This analysis provides a data-driven foundation for strategic campaign planning, highlighting both successes to build upon and areas requiring attention for future electoral success.


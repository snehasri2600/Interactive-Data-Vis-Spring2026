---
title: "Lab 4: Clearwater Crisis"
toc: false
---

# Clearwater Crisis - Whodunnit
<br>

## Step-by-step Analysis
<br>
Looking at the data, we know we have 3 fish species and 4 different stations: North, South, East, and West. The first step is to visualize the fish species over time to see how their populations have fared since the 4 culprits were built.
<br>

```js
const fish = FileAttachment("data/fish_surveys.csv").csv();
```

```js
const fishAgg = Array.from(
  d3.rollup(fish, v => d3.sum(v, d => +d.count), d => d.date, d => d.species),
  ([date, speciesMap]) => Array.from(speciesMap, ([species, count]) => ({date, species, count}))
).flat();
```

```js
Plot.plot({
  title: "Fish Population Over Time",
  width: 900,
  height: 400,
  marginBottom: 60,
  marginLeft: 50,
  x: {label: "Date", tickRotate: -30},
  y: {label: "Total Count", grid: true, zero: true},
  color: {
    legend: true,
    label: "Species",
    domain: ["Trout", "Bass", "Carp"],
    range: ["#4e8cb9", "#e8923a", "#6dbf8a"]
  },
  marks: [
    Plot.lineY(fishAgg, {x: "date", y: "count", stroke: "species", tip: true}),
    Plot.dotY(fishAgg, {x: "date", y: "count", fill: "species"})
  ]
})
```
<br>

<table>
  <thead><tr><th>Species</th><th>Pollution Sensitivity</th><th>Start (2023-01-15)</th><th>End (2024-10-13)</th><th>Change</th></tr></thead>
  <tbody>
    <tr><td>Trout</td><td>High</td><td>165</td><td>122</td><td style="color:#c0392b">-43</td></tr>
    <tr><td>Bass</td><td>Medium</td><td>234</td><td>201</td><td style="color:#c0392b">-33</td></tr>
    <tr><td>Carp</td><td>Low</td><td>98</td><td>112</td><td style="color:#27ae60">+14</td></tr>
  </tbody>
</table>

<br>
The most negative variation in fish species occurs in the trout, whereas bass and carp species stay comparatively stable. This aligns with the understanding from the attached paper that trout populations are most impacted by heavy metal contamination. Visualizing the trout population will likely give the most significant insights into the culprit of Clearwater.

<i>Note: carp populations increased slightly over time, but their pollution sensitivity is also low. If we want to know the culprit responsible for the most pollution, we should focus on the species most affected by pollution.</i>

<br>
We can use the trout population as an indicator of increasing pollution levels, and visualize the locations at which trout populations decreased the most.

```js
const trout = fish.filter(d => d.species === "Trout");
```

```js
Plot.plot({
  title: "Trout Population Over Time by Station",
  width: 900,
  height: 400,
  marginBottom: 60,
  marginLeft: 50,
  x: {label: "Date", tickRotate: -30},
  y: {label: "Count", grid: true, zero: true},
  color: {legend: true, label: "Station"},
  marks: [
    Plot.line(trout, {x: "date", y: d => +d.count, stroke: "station_id", tip: true}),
    Plot.dot(trout, {x: "date", y: d => +d.count, fill: "station_id"})
  ]
})
```
<br>
Seeing this plot, it is clear that the most severe decline in trout populations occurred in the west. Given that the ChemTech Manufacturing plant is located in the west, this <i>could</i> be an indication that the plant is the culprit in Clearwater. But, this needs further verification.

We now know that trout populations in the west experience the most significant changes. We need to see the concentrations of different pollutants over time to assess which ones were most impactful in the the lake, with an emphasis on the western side.
<br>
<br>

```js
const wq = await FileAttachment("data/water_quality.csv").csv({typed: true});
const westWQ = wq.filter(d => d.station_id === "West");
```

```js
const pollutantLabels = {
  "Nitrogen (mg/L)": "nitrogen_mg_per_L",
  "Phosphorus (mg/L)": "phosphorus_mg_per_L",
  "Heavy Metals (ppb)": "heavy_metals_ppb",
  "Turbidity (NTU)": "turbidity_ntu",
  "pH": "ph",
  "Dissolved Oxygen (mg/L)": "dissolved_oxygen_mg_per_L",
};

const pollutant = view(Inputs.select(Object.keys(pollutantLabels), {label: "Pollutant"}));
```
```js
Plot.plot({
  title: `All Stations Average: ${pollutant} Over Time`,
  width: 900,
  height: 400,
  marginBottom: 60,
  marginLeft: 60,
  x: {label: "Date"},
  y: {label: pollutant, grid: true, zero: true},
  marks: [
    Plot.line(
      Array.from(
        d3.rollup(wq, v => d3.mean(v, d => d[pollutantLabels[pollutant]]), d => d.date),
        ([date, value]) => ({date, value})
      ),
      {x: "date", y: "value", tip: true}
    ),
    Plot.dot(
      Array.from(
        d3.rollup(wq, v => d3.mean(v, d => d[pollutantLabels[pollutant]]), d => d.date),
        ([date, value]) => ({date, value})
      ),
      {x: "date", y: "value"}
    )
  ]
})
```

```js
Plot.plot({
  title: `West Station: ${pollutant} Over Time`,
  width: 900,
  height: 400,
  marginBottom: 60,
  marginLeft: 60,
  x: {label: "Date"},
  y: {label: pollutant, grid: true, zero: true},
  marks: [
    Plot.line(westWQ, {x: "date", y: d => d[pollutantLabels[pollutant]], tip: true}),
    Plot.dot(westWQ, {x: "date", y: d => d[pollutantLabels[pollutant]]})
  ]
})
```
<br>
Most of the variables are relatively stable over time, except for heavy metal concentration, which shows significant spikes in values. Once again, research shows that heavy metal concentrations significantly impact trout mortality rates, and this chart correlates with that argument. Next, we need to make sure that the dates of spikes in heavy metals line up with our previous guess of the primary pollutant, ChemTech Manufacturing.
<br>
<br>

```js
const activities = await FileAttachment("data/suspect_activities.csv").csv({typed: true});
```

```js
const selectedSuspects = view(Inputs.checkbox(
  ["ChemTech Manufacturing", "Clearwater Fishing Lodge", "Lakeview Resort", "Riverside Farm"],
  {label: "Suspects", value: ["ChemTech Manufacturing"]}
));
```
```js
Plot.plot({
  title: "Heavy Metals vs. Suspect Activities",
  width: 900,
  height: 400,
  marginBottom: 60,
  marginLeft: 60,
  x: {label: "Date"},
  y: {label: "Heavy Metals (ppb)", grid: true},
  color: {legend: true, label: "Suspect"},
  marks: [
    Plot.ruleX(activities.filter(d => selectedSuspects.includes(d.suspect)), {
      x: "date", stroke: "suspect", strokeWidth: 1.5, tip: true,
      title: d => `${d.suspect}: ${d.activity_type}`
    }),
    Plot.line(wq.filter(d => d.station_id === "West"), {
      x: "date", y: "heavy_metals_ppb", stroke: "#999", strokeWidth: 2, tip: true
    })
  ]
})
```
<br>
The results are clear.
<br>
<br>

## Who did it?
We started by visualizing fish populations over time and found that the most significant difference was in trout species. We then found that the greatest decline happened on the west side of the lake, suggesting that ChemTech Manufacturing may have been responsible. Next, we looked at the concentrations of different pollutants over time, both on average across the lake, and found that the greatest variations were in heavy metal concentrations. When comparing station activities with average heavy metal concentrations across the lake, we found that the closest overlap in dates was with ChemTech Manufacturing's maintenance shutdowns.

The biological, spatial, and temporal trends all point to one culprit responsible for majority of the pollution in Clearwater Lake: <b>ChemTech Manufacturing</b>.
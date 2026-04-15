---
title: "Lab 2: Subway Staffing"
toc: true
---

# Subway Staffing


<!-- Import Data -->
```js
const incidents = FileAttachment("./data/incidents.csv").csv({ typed: true })
const local_events = FileAttachment("./data/local_events.csv").csv({ typed: true })
const upcoming_events = FileAttachment("./data/upcoming_events.csv").csv({ typed: true })
const ridership = FileAttachment("./data/ridership.csv").csv({ typed: true })
```

<!-- Include current staffing counts from the prompt -->
```js
const currentStaffing = {
  "Times Sq-42 St": 19,
  "Grand Central-42 St": 18,
  "34 St-Penn Station": 15,
  "14 St-Union Sq": 4,
  "Fulton St": 17,
  "42 St-Port Authority": 14,
  "Herald Sq-34 St": 15,
  "Canal St": 4,
  "59 St-Columbus Circle": 6,
  "125 St": 7,
  "96 St": 19,
  "86 St": 19,
  "72 St": 10,
  "66 St-Lincoln Center": 15,
  "50 St": 20,
  "28 St": 13,
  "23 St": 8,
  "Christopher St": 15,
  "Houston St": 18,
  "Spring St": 12,
  "Chambers St": 18,
  "Wall St": 9,
  "Bowling Green": 6,
  "West 4 St-Wash Sq": 4,
  "Astor Pl": 7
}
```

## Local Events and Ridership 2025

Plotting entrances and exits, grouped by their stations, during the summer of 2025
```js
const ridership_with_events = ridership
  .flatMap(r => {
    const match = local_events.find(e =>
      e.date.getTime() === r.date.getTime() && e.nearby_station === r.station
    );
    if (!match) return [];
    return [
      { event: match.event_name, type: "entrances", count: r.entrances },
      { event: match.event_name, type: "exits", count: r.exits }
    ];
  })
```

```js
Plot.plot({
  title: "Entrances and Exits During Local Events",
  marginLeft: 200,
  x: { label: "Count", grid: true },
  y: { label: "Event" },
  color: { legend: true },
  marks: [
    Plot.barX(ridership_with_events, Plot.groupY({ x: "sum" }, {
      y: "event",
      x: "count",
      fill: "type",
      tip: true,
      sort: { y: "-x" }
    }))
  ]
})
```
Summer concert series, book fairs, and gallery openings were correlated with the most entrances and exits on the subway during the summer of 2025.
</br>
</br>

Summing up daily ridership to see the effects of the fare increase on July 15th, 2025.
```js
const dailyRidership = d3.rollups(
  ridership,
  rows => d3.sum(rows, d => d.entrances + d.exits),
  d => d.date
).map(([date, total]) => ({ date, total }))
  .sort((a, b) => a.date - b.date)
```

```js
Plot.plot({
  title: "Total Daily Ridership (All Stations, 2025)",
  width: 700,
  x: { label: "Date", type: "utc" },
  y: { label: "Total Ridership (Entrances + Exits)", grid: true },
  marks: [
    Plot.lineY(dailyRidership, { x: "date", y: "total", stroke: "blue", tip: true }),
    Plot.ruleX([new Date("2025-07-15")], { stroke: "pink", strokeWidth: 2, strokeDasharray: "4,4" }),
    Plot.text([{ date: new Date("2025-07-15"), total: d3.max(dailyRidership, d => d.total) }], {
      x: "date",
      y: "total",
      text: ["July 15: Fare Increase"],
      dx: 6,
      textAnchor: "start",
      fill: "red",
      fontSize: 12
    })
  ]
})
```

Ridership experienced a drastic decrease after the fare increase in July, 2025. 
</br>
</br>
</br>
## Response Time by Station

```js
Plot.plot({
  title: "Average Incident Response Time by Station (2015–2025)",
  width: 700,
  height: 530,
  marginLeft: 100,
  x: { label: "Average Response Time (minutes)", grid: true },
  y: { label: "Station" },
  marks: [
    Plot.barX(incidents, Plot.groupY({ x: "mean" }, {
      y: "station",
      x: "response_time_minutes",
      fill: "pink",
      sort: { y: "-x" },
      tip: true
    })),
    // Overall mean annotation
    Plot.ruleX([d3.mean(incidents, d => d.response_time_minutes)], {
      stroke: "grey",
      strokeWidth: 2
    }),
  
    Plot.ruleX([0])
  ]
})
```

The grey line marks the mean. Stations above it are slow to respond, while stations below the mean are quicker.
</br>
</br>
</br>

## Staffing Needs - Top 3 Stations

```js
Plot.plot({
  width: 700,
  title: "Stations by Expected Attendance Counts",
  x: { tickRotate: -45, label: "Station" },
  y: {label: "Expected Attendance"},
  marginBottom: 80,
  marks: [
    Plot.barY(upcoming_events, Plot.groupX({ y: "sum" }, {
      y: "expected_attendance",
      x: "nearby_station",
      tip: true
    }))
  ]
})
```

Looking at the previous chart of historical response times and comparing it with the expected attendance counts in 2026, Canal Street is one of the top stations in need of staffing. Though Chambers St and Penn Station expect the most attendance, they may not be in need of as much staff because their response times are low as it is. Mapping the intensity of past historical events may be beneficial here.

```js
Plot.plot({
  title: "Incident Severity by Station",
  marginLeft: 200,
  color: {
    domain: ["low", "medium", "high"],
    range: ["yellow", "orange", "red"]
  },
  marks:[
    Plot.barX(incidents, Plot.groupY({x: "count"}, {
      y: "station",
      fill: "severity",
      order: ["low", "medium", "high"],
      tip:true
    }))
  ]
})
```

It looks like Chambers Street does have highly severe incidents along with high expected attendance. Though response time is low, additional staff at this station would be helpful. Finally, Penn Station has a high number of incidents and a high expected attendance count, so additional staff here would be useful. It is worth noting that Wall St has the highest incident count, but quick response times and low expected attendance. As a result, additional staff is likely not needed.
</br>

The three stations that should have higher staff counts in summer 2026 are Canal St, Chambers St, and Penn Station. 
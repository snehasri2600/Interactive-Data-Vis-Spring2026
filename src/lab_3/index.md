---
title: "Lab 3: Mayoral Mystery"
toc: false
---

<!-- Import Data -->
```js
const nyc = await FileAttachment("data/nyc.json").json();
const results = await FileAttachment("data/election_results.csv").csv({ typed: true });
const survey = await FileAttachment("data/survey_responses.csv").csv({ typed: true });
const events = await FileAttachment("data/campaign_events.csv").csv({ typed: true });

// Note: you don't have to keep this, but some helpful data exposure to see what we've loaded. 
// NYC geoJSON data
display(nyc)
// Campaign data (first 10 objects)
display("Election results")
display(results.slice(0,10))
display("Survey Responses")
display(survey.slice(0,10))
display("Campaign Events")
display(events.slice(0,10))
```


```js
// The nyc file is saved in data as a topoJSON instead of a geoJSON. Thats primarily for size reasons -- it saves us 3MB of data. For Plot to render it, we have to convert it back to its geoJSON feature collection. 
const districts = topojson.feature(nyc, nyc.objects.districts)
display(districts)
```
```js
const lookup = new Map(results.map(d => [d.boro_cd, d]))
```

```js
// Simple rendering of the NYC districts topoJSON
Plot.plot({
  // this projection is already zoomed into NYC
  projection: {
    domain: districts,
    type: "mercator",
  },
  color: {
    domain: ["Candidate", "Opponent"],
    range: ["#8989e4", "#e8aef0"],
    legend: true,
  },
  marks: [
    Plot.geo(districts, {
      fill: district => {
        const d = lookup.get(district.properties.BoroCD);
        if (!d) return "gray";
        return d.votes_candidate > d.votes_opponent ? "#8989e4" : "#e8aef0";
      },
      tip: true,
      channels: {
        "Winner": district => {
          const d = lookup.get(district.properties.BoroCD);
          if (!d) return "No data";
          return d.votes_candidate > d.votes_opponent ? "Candidate" : "Opponent";
        },
        "Candidate Votes": district => lookup.get(district.properties.BoroCD)?.votes_candidate?.toLocaleString(),
        "Opponent Votes": district => lookup.get(district.properties.BoroCD)?.votes_opponent?.toLocaleString(),
      }
    }
    )
  ]
})
```

It looks like the candidate lost lower Manhattan, all of Staten Island, and much of the western sides of Brooklyn and Queens. Maybe the candidate needs to spend more time campaigning there. The following visualization will attempt to explore this.

```js
const boroughNames = {1: "Manhattan", 2: "Bronx", 3: "Brooklyn", 4: "Queens", 5: "Staten Island"};

const resultsWithBorough = results.map(d => ({
  ...d,
  borough: boroughNames[Math.floor(d.boro_cd / 100)]
}));
```

```js
Plot.plot({
  title: "Total Hours Spent Campaigning, by Borough",
  x: { label: "Borough" },
  y: { label: "Total Hours Spent", grid: true},
  color: { scheme: "blues", range: [0.3, 1] },
  marks: [
    Plot.barY(resultsWithBorough, Plot.groupX({ y: "sum", fill: "sum" }, {
      x: "borough",
      y: "candidate_hours_spent",
      fill: "candidate_hours_spent",
      tip: { format: { fill: false } }
    })),
    Plot.ruleY([0])
  ]
})
```
The total hours spent campaigning in each borough does align with the map of wins and losses, for the most part. Brooklyn and the Bronx was almost completely won by the candidate, and the candidate spent the most time there. However, the amount of time spent in Manhattan, despite being close in number of hours to Brooklyn and the Bronx, was split in votes. This would require further analysis of time spent in each district within Manhattan. Queens and Staten Island results align with the map, with both being won mostly by the opponent. 
<br>
<br>

For the purposes of this lab, I will leave this analysis of hours spent campaigning there, so that we can visualize some other variables. 
<br>
<br>

I want to be able to see the populations' alignment of issues with the candidate based on survey responses. In other words, visualizing which issus were most important to voters who voted for this candidate.

```js
const alignmentIssues = [
  "affordable_housing_alignment",
  "public_transit_alignment",
  "childcare_support_alignment",
  "small_business_tax_alignment",
  "police_reform_alignment"
];

const issueLabels = {
  "affordable_housing_alignment": "Affordable Housing",
  "public_transit_alignment": "Public Transit",
  "childcare_support_alignment": "Childcare Support",
  "small_business_tax_alignment": "Small Business Tax",
  "police_reform_alignment": "Police Reform"
};

const tidySurvey = survey
  .filter(d => d.voted_for === "Candidate" || d.voted_for === "Opponent")
  .flatMap(d => alignmentIssues.map(issue => ({
    voted_for: d.voted_for === "Candidate" ? "Voted for Candidate" : "Voted for Opponent",
    issue: issueLabels[issue],
    alignment: d[issue]
  })));
  ```

```js
Plot.plot({
  title: "Average Issue Alignment by Voter Choice",
  marginLeft: 10,
  width: 1000,
  x: { label: "Avg Alignment (1–5)", domain: [1, 7], grid: true },
  y: { label: null, axis: null },
  fy: { domain: ["Voted for Candidate", "Voted for Opponent"], label: null },
  color: { domain: ["Voted for Candidate", "Voted for Opponent"], range: ["#8989e4", "#e8aef0"], legend: true },
  marks: [
    Plot.barX(tidySurvey, Plot.groupY({ x: "mean" }, {
      y: "issue",
      x: "alignment",
      fy: "voted_for",
      fill: "voted_for",
      tip: { format: { y: false, fill: false, fy: false } }
    })),
    Plot.text(tidySurvey, Plot.groupY({ x: "mean", text: "first" }, {
      y: "issue",
      x: "alignment",
      fy: "voted_for",
      text: "issue",
      textAnchor: "start",
      dx: 4,
      fill: "black",
    })),
    Plot.ruleX([0])
  ]
})
```

It doesn't seem like alignment made much of a difference in who the voters voted for. On each issue, the voters seemed similarly aligned between those who voted for the opponent and those who voted for the candidate. The candidate should spend time communicating these policies, since alignment seems to be the same regardless of the vote. That being said, the biggest difference seems to be within the candidate's small business policies.

<b>Initial Conclusions</b>
<br>
I think we would need more information on the time spent by the candidate in each district, though they should definitely spend more time in Queens and Staten Island. Communication of policies should be clearer, and the campaign should focus on strategies that appeal to voter demographics. The immediate next step would be to assess voter demographics to find where the differences lie, because they are not in policy.
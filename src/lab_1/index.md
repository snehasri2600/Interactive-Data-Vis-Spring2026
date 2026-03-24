---
title: "Lab 1: Passing Pollinators"
toc: true
---

# Lab 1: Prolific Pollinators

<br>

### Question 1
<b>What is the body mass and wing span distribution of each pollinator species observed?</b>

```js
const data = FileAttachment("./data/pollinator_activity_data.csv").csv({typed: true}); //load in the data
```

```js
Plot.plot({
    title: "Pollinator Body Mass",
    y: {grid: true},
    width: 500,
    height: 300,
    marks: [
        Plot.boxY(
            data,
            {x: "pollinator_species", 
            y: "avg_body_mass_g", 
            fill: "pollinator_species"}
        )
    ]
})
```

```js
Plot.plot({
    title: "Pollinator Wingspan",
    y: {grid: true},
    width: 500,
    height: 300,
    marks:[
        Plot.boxY(
            data,
            {x: "pollinator_species", 
            y: "avg_wing_span_mm", 
            fill: "pollinator_species"}
        )
    ]
})
```
<br>
The highest body mass and wingspan values are seen in the carpenter bee, followed by the bumblebee. The smallest species in both mass and wingspan is the honeybee.
<br>
<br>
<br>

### Question 2
<b>What is the ideal weather (conditions, temperature, etc) for pollinating?</b>

```js
Plot.plot({
    title: "Weather conditions vs. nectar production",
    y: {grid: true},
    width: 400,
    height: 400,
    marks: [
        Plot.barY(
            data,
            {x: "weather_condition", 
            y: "nectar_production", 
            fill: "weather_condition"}
        )
    ]
})
```

```js
Plot.plot({
    title: "Temperature vs. Nectar Production",
    y: {grid: true},
    width: 500,
    height: 300,
    color: {legend: true},
    marks: [
        Plot.dot(
            data,
            {x: "temperature", 
            y: "nectar_production", 
            fill: "pollinator_species"}
        )
    ]
})
```

```js
Plot.plot({
    title: "Humidity vs. Nectar Production",
    y: {grid: true},
    width: 500,
    height: 300,
    color: {legend: true},
    marks: [
        Plot.dot(
            data,
            {x: "humidity", 
            y: "nectar_production", 
            fill: "pollinator_species"}
        )
    ]
})
```

```js
Plot.plot({
    title: "Wind Speed vs. Nectar Production",
    y: {grid: true},
    width: 500,
    height: 300,
    color: {legend: true},
    marks: [
        Plot.dot(
            data,
            {x: "wind_speed", 
            y: "nectar_production", 
            fill: "pollinator_species"}
        )
    ]
})
```
<br>
Looking at the plots above, it doesn't seem like there is any relationship between temperature, humidity, or wind speed and nectar production. However, the bar plot shows that cloudy weather is the best for nectar production, while partly cloudy, followed by sunny, decrease production. 
<br>
<br>
<br>

### Question 3
<b>Which flower has the most nectar production?</b>

```js
Plot.plot({
    title: "Flower Species vs. Nectar Production",
    y: {grid: true},
    width: 500,
    height: 300,
    marks: [
        Plot.barY(
            data,
            {x: "flower_species", 
            y: "nectar_production",
            fill: "flower_species"}
        )
    ]
})
```
<br>
It is easy to see that sunflowers are the highest nectar-producing species from the sample, followed by coneflowers, and finally, lavender. 

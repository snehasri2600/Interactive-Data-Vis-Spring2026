---
title: "Lab 0: Getting Started"
toc: true
---

# Beginning my data visualization journey!
## Interactive Data Visualization, Spring 2026
<br>

### What I want to accomplish
<table>
<thead>
<th>Goals for the Semester</th>
<th>Timeline</th>
</thead>
<tbody>
<tr>
<td>Develop general graphs and charts</td>
<td>By end of March 2026</td>
</tr>
<tr>
<td>Develop moveable graphs and charts</td>
<td>By mid-April 2026</td>
</tr>
<tr>
<td>Customize maps for visualizations</td>
<td>By mid-April 2026</td>
</tr>
<tr>
<td>Make maps with moving and interactive features</td>
<td>By end of April 2026</td>
</tr>
</tbody>
</table>

The reason I want to develop customized, moving maps by the end of April is that my capstone will involve a time-series visualization of water quality in the Jamaica Bay between different sensor stations. 

<br>
<br>

### How I am going to accomplish these goals:
<ul>
    <li>Keep on top of lab work</li>
    <li>Self-study ArcGIS and ArcGIS Hub</li>
    <li>Do optional work to develop more skills</li>
</ul>

```js
const goalsInput = Inputs.checkbox(["Develop general graphs and charts", 
"Develop moveable graphs and charts", 
"Customize maps for visualizations", 
"Make maps with moving and interactive features"], 
{label: "Goals:"});

goalsInput.style.maxWidth = "500px";

const goals = view(goalsInput);
```
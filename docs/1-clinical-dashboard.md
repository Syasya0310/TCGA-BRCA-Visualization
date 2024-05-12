---
theme: [air]
---

# Clinical Data Summary

```js
import { PieChart } from "./functions.js";
```

```js
const brcac = await FileAttachment("data/BRCA_Clinical.csv").csv({
  typed: true,
});

const genderRolld = d3.flatRollup(
  brcac,
  (v) => v.length,
  (d) => d.gender
);
const gender = genderRolld.map(([gender, Count]) => ({ gender, Count }));

const ethnicRolld = d3.flatRollup(
  brcac,
  (v) => v.length,
  (d) => d.ethnicity
);
const ethnic = ethnicRolld.map(([ethnicity, Count]) => ({ ethnicity, Count }));

const ageRolld = d3.flatRollup(
  brcac,
  (v) => v.length,
  (d) => d.age_diagnosis_range
);
const age = ageRolld.map(([age_diagnosis_range, Count]) => ({
  age_diagnosis_range,
  Count,
}));

const genderColor = ["#e6308a", "#0073e6"];
const statusColor = ["#ffffbf", "#91cf60", "#fc8d59"];
const ethnicColor = ["#ffffcc", "#fec965", "#fa5c2e"];
```

```js
function cancerPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Patient Status",
    width,
    x: { label: null },
    y: { grid: true, label: "Count" },
    color: { legend: true, range: statusColor },
    marks: [
      Plot.barY(
        data,
        Plot.groupX(
          { y: "count" },
          {
            x: "vital_status",
            y: "person_neoplasm_cancer_status",
            fill: "person_neoplasm_cancer_status",
            tip: true,
          }
        )
      ),
      Plot.ruleY([0]),
    ],
  });
}

function racePlot(data, { width } = {}) {
  return Plot.plot({
    title: "Race",
    width,
    x: { label: null },
    y: { grid: true, label: "Count" },
    color: { legend: true },
    marks: [
      Plot.barY(
        data,
        Plot.groupX(
          { y: "count" },
          {
            x: "race_list",
            fill: "race_list",
            tip: true,
            sort: { x: "y", reverse: true },
          }
        )
      ),
      Plot.ruleY([0]),
    ],
  });
}

function ethnicPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Ethnicity",
    width,
    x: { label: null },
    y: { grid: true, label: "Count" },
    color: { legend: true, range: ethnicColor },
    marks: [
      Plot.barY(
        data,
        Plot.groupX(
          { y: "count" },
          {
            x: "ethnicity",
            fill: "ethnicity",
            tip: true,
            sort: { x: "y", reverse: true },
          }
        )
      ),
      Plot.ruleY([0]),
    ],
  });
}

function ageDiagnosisPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Age At Diagnosis",
    width,
    x: { label: null },
    y: { grid: true, label: "Count" },
    marks: [
      Plot.rectY(
        data,
        Plot.binX(
          { y: "count" },
          {
            x: "age_at_initial_pathologic_diagnosis",
            fill: "age_at_initial_pathologic_diagnosis",
            tip: true,
          }
        )
      ),
      Plot.ruleY([0]),
    ],
  });
}
```

<div class="grid grid-cols-3">
  <div class="card"><h3>Gender</h3><br> ${PieChart(gender, {
      name: (d) => d.gender,
      value: (d) => d.Count,
      colors: genderColor,
      width,
    })}
  </div>
  <div class="card"><h3>Age At Diagnosis</h3><br> ${PieChart(age, {
      name: (d) => d.age_diagnosis_range,
      value: (d) => d.Count,
      width,
    })}
  </div>
  <div class="card"><h3>Ethnicity</h3><br> ${PieChart(ethnic, {
      name: (d) => d.ethnicity,
      value: (d) => d.Count,
      colors: ethnicColor,
      width,
    })}
  </div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => racePlot(brcac, {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => cancerPlot(brcac, {width}))}</div>
</div>

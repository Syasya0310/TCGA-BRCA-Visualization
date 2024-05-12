---
theme: [air]
---

# Relationships Data Summary

```js
import { joinData } from "./functions.js";
```

```js
const brcac = await FileAttachment("data/BRCA_Clinical.csv").csv({
  typed: true,
});
const brcam = await FileAttachment("data/BRCA_Mutational.csv").csv({
  typed: true,
});
```

```js
const brcaCombined = joinData(
  brcac,
  brcam,
  "bcr_patient_barcode",
  "patient_barcode",
  function (brcam, brcac) {
    return {
      bcr_patient_barcode: brcac.bcr_patient_barcode,
      gender: brcac.gender,
      vital_status: brcac.vital_status,
      race_list: brcac.race_list,
      age_at_diagnosis: brcac.age_at_initial_pathologic_diagnosis,
      ethnicity: brcac.ethnicity,
      person_neoplasm_cancer_status: brcac.cancer_status,
      age_diagnosis_range: brcac.age_diagnosis_range,
      Hugo_Symbol: brcam.Hugo_Symbol,
      Tumor_Sample_Barcode: brcam.Tumor_Sample_Barcode,
      HGVSc: brcam.HGVSc,
      t_depth: brcam.t_depth,
      t_ref_count: brcam.t_ref_count,
      t_alt_count: brcam.t_alt_count,
      n_depth: brcam.n_depth,
      One_Consequence: brcam.One_Consequence,
      VARIANT_CLASS: brcam.VARIANT_CLASS,
      snv_class: brcam.snv_class,
      VAF: brcam.VAF,
    };
  }
);

const mutationRolld = d3.flatRollup(
  brcaCombined,
  (v) => v.length,
  (d) => d.age_at_diagnosis,
  (d) => d.bcr_patient_barcode
);
const mutation = mutationRolld.map(
  ([age_at_diagnosis, bcr_patient_barcode, Count]) => ({
    age_at_diagnosis,
    bcr_patient_barcode,
    Count,
  })
);
```

```js
// Zooming for quantitative axes, found here
// https://observablehq.com/d/afe64fd13a4194eb
// Original author: Cui
// Modified by: Syasya R.N.B. Rosli
var arr = [];
arr.push(
  Plot.dot(mutation, {
    y: "Count",
    x: "age_at_diagnosis",
    tip: true,
    channels: { bcr_patient_barcode: "bcr_patient_barcode" },
  })
);
const div = d3.create("div");
const plotDiv = div.append("div");

const plot = (xDomain, yDomain) =>
  Plot.plot({
    title: "Mutation Count at Diagnosis",
    x: { domain: xDomain },
    y: { domain: yDomain },
    inset: 10,
    grid: true,
    color: { legend: true },
    marks: arr,
  });

function insertPlot(xDomain = [25, 90], yDomain = [0, 5000]) {
  console.log(1);
  const chart = plot(xDomain, yDomain);
  plotDiv.html("").append(() => chart);
  return [chart.scale("x"), chart.scale("y")];
}
const [xScale, yScale] = insertPlot();
const l = d3.scaleLinear();
function scaledDomain({ k, x }) {
  l.domain(xScale.domain).range(xScale.range);
  return [
    l.invert((xScale.range[0] - x) / k),
    l.invert((xScale.range[1] - x) / k),
  ];
}
function scaledDomainy({ k, y }) {
  l.domain(yScale.domain).range(yScale.range);
  return [
    l.invert((yScale.range[0] - y) / k),
    l.invert((yScale.range[1] - y) / k),
  ];
}
const zoom = d3
  .zoom()
  .on("zoom", ({ transform }) =>
    insertPlot(scaledDomain(transform), scaledDomainy(transform))
  );

div.call(zoom);
```

```js
const deadPatients = brcaCombined.filter(function (d) {
  return d.vital_status.match(/Dead/);
});
```

```js
function variantSamplePlot(data, title, { width } = {}) {
  return Plot.plot({
    title: title,
    width,
    marginLeft: 70,
    x: { label: "Count" },
    y: { grid: true, label: null },
    color: { legend: true, scheme: "Tableau10" },
    marks: [
      Plot.barX(
        data,
        Plot.groupY(
          { x: "count" },
          {
            x: "One_Consequence",
            y: "bcr_patient_barcode",
            fill: "One_Consequence",
            tip: true,
            sort: { y: "x", reverse: true },
          }
        )
      ),
      Plot.ruleX([0]),
    ],
  });
}

function vafPlot(data, title, { width } = {}) {
  return Plot.plot({
    title: title,
    width,
    marginLeft: 70,
    x: { grid: true, label: "VAF" },
    y: { label: null },
    color: { legend: true },
    marks: [
      Plot.boxX(data, {
        y: "bcr_patient_barcode",
        x: "VAF",
        sort: { y: "x" },
        tip: true,
        channels: { bcr_patient_barcode: "bcr_patient_barcode" },
      }),
    ],
  });
}

function deadGenesPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Top 10 Deadliest Genes",
    width,
    x: { label: null },
    y: { grid: true, label: "Count" },
    color: { legend: true, scheme: "Tableau10" },
    marks: [
      Plot.barY(
        data,
        Plot.groupX(
          { y: "count" },
          {
            x: "Hugo_Symbol",
            fill: "One_Consequence",
            tip: true,
            sort: { x: "y", reverse: true, limit: 10 },
          }
        )
      ),
      Plot.ruleY([0]),
    ],
  });
}

const search = view(
  Inputs.search(d3.group(brcaCombined), {
    label: "Search",
    sort: true,
    unique: true,
  })
);
```

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => variantSamplePlot(search, "Variants per Sample", {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => vafPlot(search, "Variant Allele Frequency (VAF)", {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => deadGenesPlot(deadPatients, {width}))}</div>
</div>

<div class="card">${div.node()}</div>

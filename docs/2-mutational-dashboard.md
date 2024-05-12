---
theme: [air]
---

# Mutational Data Summary

```js
const brcam = await FileAttachment("data/BRCA_Mutational.csv").csv({
  typed: true,
});

const snvbrcam = brcam.filter((d) => d.snv_class != "NA");
const vafbrcam = brcam.filter((d) => d.gnomAD_AF != "NA");

const variantColor = ["#e49ac5", "#ee9ca5", "#eea38f", "#ecc28f", "#c2d492"];
```

```js
function varClassPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Variant Classification",
    width,
    marginLeft: 115,
    x: { grid: true, label: "Count" },
    y: { label: null },
    color: { legend: true, scheme: "Tableau10" },
    marks: [
      Plot.barX(
        data,
        Plot.groupY(
          { x: "count" },
          {
            y: "One_Consequence",
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

function topGenesPlot(data, { width } = {}) {
  return Plot.plot({
    title: "Top 10 Mutated Genes",
    width,
    x: { grid: true, label: "Count" },
    y: { label: null },
    color: { legend: true, scheme: "Tableau10" },
    marks: [
      Plot.barX(
        data,
        Plot.groupY(
          { x: "count" },
          {
            x: "One_Consequence",
            y: "Hugo_Symbol",
            fill: "One_Consequence",
            sort: { y: "x", reverse: true, limit: 10 },
            tip: true,
          }
        )
      ),
      Plot.ruleX([0]),
    ],
  });
}

function varTypePlot(data, { width } = {}) {
  return Plot.plot({
    title: "Variant Type",
    width,
    x: { grid: true, label: "Count" },
    y: { label: null },
    marginLeft: 50,
    color: { legend: true, range: variantColor },
    marks: [
      Plot.barX(
        data,
        Plot.groupY(
          { x: "count" },
          {
            y: "VARIANT_CLASS",
            fill: "VARIANT_CLASS",
            tip: true,
            sort: { y: "x", reverse: true },
          }
        )
      ),
      Plot.ruleX([0]),
    ],
  });
}

function snvClassPlot(data, { width } = {}) {
  return Plot.plot({
    title: "SNV Class",
    width,
    x: { grid: true, label: "Count" },
    y: { label: null },
    color: { legend: true },
    marks: [
      Plot.barX(
        data,
        Plot.groupY(
          { x: "count" },
          { y: "snv_class", fill: "snv_class", tip: true }
        )
      ),
      Plot.ruleX([0]),
    ],
  });
}
```

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => varClassPlot(brcam, {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => topGenesPlot(brcam, {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => varTypePlot(brcam, {width}))}</div>
</div>

<div class="grid grid-cols-1">
  <div class="card">${resize((width) => snvClassPlot(snvbrcam, {width}))}</div>
</div>

---
theme: [air]
---

```js
const brcac = await FileAttachment("data/BRCA_Clinical.csv").csv({
  typed: true,
});
const brcam = await FileAttachment("data/BRCA_Mutational.csv").csv({
  typed: true,
});
```

## BRCA Clinical Table

<br><div>${Inputs.table(brcac)}</div><br>

## BRCA Mutational Table

<br><div>${Inputs.table(brcam)}</div>

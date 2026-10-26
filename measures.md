# DAX Measures and Formatting Notes

This document records the dynamic-unit approach used in the report. Because the exact full set of measures in the PBIX was not available for inspection, this is a focused reference rather than a complete measure export.

## Dynamic GNPA Product Chart

The chart's existing `[Closing GNPA (Lakhs)]` measure is assumed to return GNPA in lakhs. The dynamic measure converts that lakhs-valued result into crores, lakhs, or thousands based on its magnitude:

```DAX
Closing GNPA (Dynamic) =
VAR Amount = [Closing GNPA (Lakhs)]
VAR AbsAmount = ABS(Amount)
RETURN
    SWITCH(
        TRUE(),
        AbsAmount >= 100, DIVIDE(Amount, 100),
        AbsAmount >= 1, Amount,
        AbsAmount >= 0.01, Amount * 100,
        Amount * 100000
    )
```

### Dynamic unit label

```DAX
GNPA Dynamic Unit =
VAR Amount = [Closing GNPA (Lakhs)]
VAR AbsAmount = ABS(Amount)
RETURN
    SWITCH(
        TRUE(),
        AbsAmount >= 100, "₹ Crores",
        AbsAmount >= 1, "₹ Lakhs",
        AbsAmount >= 0.01, "₹ Thousands",
        "₹"
    )
```

### Dynamic chart title

The title is based on the largest product-level value under the current selection. Replace `Dummy_Data[Product]` with the exact product field used by the chart if its model name differs.

```DAX
GNPA Product Chart Title =
VAR MaxProductGNPA =
    MAXX(
        ALLSELECTED(Dummy_Data[Product]),
        CALCULATE([Closing GNPA (Lakhs)])
    )
VAR AbsAmount = ABS(MaxProductGNPA)
VAR Unit =
    SWITCH(
        TRUE(),
        AbsAmount >= 100, "₹ Crores",
        AbsAmount >= 1, "₹ Lakhs",
        AbsAmount >= 0.01, "₹ Thousands",
        "₹"
    )
RETURN
    "Closing GNPA by Product (" & Unit & ")"
```

Apply the title measure using **Format visual → General → Title → fx → Field value**. The title thresholds must match the conversion thresholds used by the plotted numeric measure. Validate the displayed unit and values in Power BI after changing slicers.

## General calculation patterns

### Absolute movement

For a measure whose opening and closing values share the same unit:

```DAX
Movement = [Closing Value] - [Opening Value]
```

### Percentage movement

```DAX
Movement % =
DIVIDE(
    [Closing Value] - [Opening Value],
    [Opening Value]
)
```

Format the percentage measure as a percentage. Confirm the intended handling of zero opening values and negative amounts before relying on percentage changes.

### ECL-to-Asset Ratio

```DAX
ECL-to-Asset Ratio =
DIVIDE([Closing ECL], [Closing GL Asset])
```

Format as a percentage only after confirming that both measures use compatible units and definitions.

## Validation checklist

- Confirm the source unit of each raw financial field before applying conversions.
- Confirm that opening and closing measures use consistent filters and definitions.
- Verify that titles and plotted values use the same unit thresholds.
- Check dynamic titles under slicer selections and when values are zero or negative.
- Validate the branch cumulative percentage and its sort order.
- Confirm official meanings of GA ACC, GL Asset, stage categories, and bucket categories before operational interpretation.

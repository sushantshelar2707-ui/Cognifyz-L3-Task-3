
# Level 3 — Google Colab cells (copy one by one, top to bottom)

Paste each numbered block into its **own Colab code cell** and run in order.
Lines marked *optional text cell* can be added with Colab's **+ Text** button for a polished submission.


---


### ▶ CODE CELL 1

```python
import os
import pandas as pd
import numpy as np
import matplotlib
import matplotlib.pyplot as plt

pd.set_option("display.max_columns", None)
pd.set_option("display.width", 140)

OUT_DIR = "level3/output"
os.makedirs(OUT_DIR, exist_ok=True)

print("pandas", pd.__version__, "| matplotlib", matplotlib.__version__)
```


---



### ▶ CODE CELL 2

```python


CANDIDATE_PATHS = [
    "level2/output/trains_enriched.csv",
    "trains_enriched.csv",
    "level1/output/trains_cleaned.csv",
    "trains_cleaned.csv",
    "Railway_info.csv",
    "data/raw/Railway_info.csv",
]

def load_dataset():
    """Return (dataframe, source_path). Falls back to a Colab upload widget."""
    for path in CANDIDATE_PATHS:
        if os.path.exists(path):
            print(f"Loading dataset from: {path}")
            return pd.read_csv(path), path
    try:  # Google Colab upload fallback
        from google.colab import files
        print("Dataset not found in this session — please upload your CSV:")
        uploaded = files.upload()
        name = next(iter(uploaded))
        return pd.read_csv(name), name
    except ImportError:
        raise FileNotFoundError(
            "Could not locate the dataset CSV. Upload it (Colab: folder icon) and re-run."
        )

df, source_path = load_dataset()

def find_column(frame, *patterns):
    """First column whose lower-cased name contains any of the patterns."""
    for pattern in patterns:
        for col in frame.columns:
            if pattern in col.lower():
                return col
    return None

train_col  = find_column(df, "train no", "train_no", "train id", "train code", "train") or df.columns[0]
source_col = find_column(df, "source", "origin") or find_column(df, "from")
dest_col   = find_column(df, "destination") or find_column(df, "to")
day_col    = find_column(df, "day") or "days"

# Idempotent enrichment: repair corrupted day tokens & ensure Day_Category exists
DAYS = ["Monday", "Tuesday", "Wednesday", "Thursday", "Friday", "Saturday", "Sunday"]

def normalize_day(value):
    v = str(value).strip().capitalize()
    for day in DAYS:
        if v.startswith(day):
            return day
    return v

_clean = df[day_col].fillna("UNKNOWN").astype(str).str.strip().str.capitalize()
n_dirty = int((~_clean.isin(DAYS)).sum())
if n_dirty:
    df[day_col] = _clean.apply(normalize_day)
    print(f"Repaired {n_dirty:,} corrupted day value(s).")

def categorize_day(day):
    if day in {"Saturday", "Sunday"}:
        return "Weekend"
    if day in {"Monday", "Tuesday", "Wednesday", "Thursday", "Friday"}:
        return "Weekday"
    return "Unknown"

if "Day_Category" not in df.columns:
    df["Day_Category"] = df[day_col].apply(categorize_day)

print(f"Rows: {len(df):,} | Columns: {df.shape[1]}")
df.head(10)
```


### ▶ CODE CELL 3

```python

weekly = df[day_col].value_counts().reindex(DAYS)
weekly_share = (weekly / weekly.sum() * 100).round(2)

fig, ax = plt.subplots(figsize=(10, 5.5))
colors = ["#f2a03d" if d in ("Saturday", "Sunday") else "#5b8def" for d in DAYS]
bars = ax.bar(DAYS, weekly.values, color=colors)
ax.bar_label(bars, fmt="{:,.0f}", padding=3)
ax.set_title("Train Journeys by Day of Week", fontsize=14, fontweight="bold")
ax.set_xlabel("Day of week")
ax.set_ylabel("Number of train services")
ax.set_ylim(0, weekly.max() * 1.15)
ax.grid(axis="y", alpha=0.3)
plt.tight_layout()
plt.savefig(f"{OUT_DIR}/fig_weekly_distribution.png", dpi=150)
plt.show()

print(weekly.rename("services").to_frame().assign(share_pct=weekly_share))
print(f"\nBusiest day : {weekly.idxmax()} ({weekly.max():,} services)")
print(f"Quietest day: {weekly.idxmin()} ({weekly.min():,} services)")
```


### ▶ CODE CELL 4

```python

top_routes = (
    df.groupby([source_col, dest_col]).size()
      .rename("services").sort_values(ascending=False)
)
print("Top 10 routes (source -> destination):")
print(top_routes.head(10))

# Direction-agnostic two-way corridors
corridors = (
    df.apply(lambda r: " <-> ".join(sorted([r[source_col], r[dest_col]])), axis=1)
      .value_counts()
)
print("\nTop 5 two-way corridors:")
print(corridors.head())

top10_hub_share = (df[source_col].value_counts().head(10).sum() / len(df) * 100).round(1)
print(f"\n Top-10 source stations carry {top10_hub_share}% of all services (hub concentration).")

labels = [f"{s} -> {d}" for s, d in top_routes.head(10).index]
fig, ax = plt.subplots(figsize=(10, 6))
ax.barh(labels[::-1], top_routes.head(10).values[::-1], color="#3aa17e")
ax.set_title("Top 10 Routes by Number of Services", fontsize=14, fontweight="bold")
ax.set_xlabel("Number of services")
ax.grid(axis="x", alpha=0.3)
plt.tight_layout()
plt.savefig(f"{OUT_DIR}/fig_top_routes.png", dpi=150)
plt.show()
```


### ▶ CODE CELL 5

```python
top8 = df[source_col].value_counts().head(8).index
pivot = (
    df[df[source_col].isin(top8)]
    .groupby([source_col, day_col]).size()
    .unstack(fill_value=0)
    .reindex(columns=DAYS)
)

fig, ax = plt.subplots(figsize=(10, 6))
im = ax.imshow(pivot.values, cmap="YlOrRd", aspect="auto")
ax.set_xticks(range(len(DAYS)))
ax.set_xticklabels(DAYS, rotation=45, ha="right")
ax.set_yticks(range(len(pivot)))
ax.set_yticklabels(pivot.index)
for i in range(pivot.shape[0]):
    for j in range(pivot.shape[1]):
        ax.text(j, i, pivot.values[i, j], ha="center", va="center", fontsize=8,
                color="white" if pivot.values[i, j] > pivot.values.max() / 2 else "black")
ax.set_title("Services per Day — Top 8 Source Stations", fontsize=14, fontweight="bold")
fig.colorbar(im, ax=ax, label="services")
plt.tight_layout()
plt.savefig(f"{OUT_DIR}/fig_hub_heatmap.png", dpi=150)
plt.show()
```


---



### ▶ CODE CELL 6

```python

day_index  = pd.Series(range(7), index=DAYS)        # Mon=0 ... Sun=6
is_weekend = pd.Series([0, 0, 0, 0, 0, 1, 1], index=DAYS)

r_ordinal = weekly.corr(day_index)     # volume vs position in the week
r_weekend = weekly.corr(is_weekend)    # point-biserial: weekend effect

print(f"Pearson r (service volume vs day-of-week position) : {r_ordinal:+.3f}")
print(f"Pearson r (service volume vs weekend flag)         : {r_weekend:+.3f}")

wd_mean = weekly[DAYS[:5]].mean()
we_mean = weekly[DAYS[5:]].mean()
weekend_share = weekly[DAYS[5:]].sum() / weekly.sum() * 100

print(f"\nAverage services per weekday     : {wd_mean:,.0f}")
print(f"Average services per weekend day : {we_mean:,.0f}")
print(f"Weekend share of weekly services : {weekend_share:.1f}%")

def strength(r):
    a = abs(r)
    return "negligible" if a < 0.2 else "weak" if a < 0.4 else "moderate" if a < 0.7 else "strong"

print(f"\n Interpretation: the relationship between volume and day position is "
      f"{strength(r_ordinal)} (r={r_ordinal:+.3f}); the weekend effect is {strength(r_weekend)} "
      f"(r={r_weekend:+.3f}). Demand is spread evenly across the week." if abs(r_ordinal) < 0.4
      else f"\n Interpretation: a {strength(r_ordinal)} weekly trend exists (r={r_ordinal:+.3f}).")
```


### ▶ CODE CELL 7

```python

top3_hubs = ", ".join(df[source_col].value_counts().head(3).index)
busiest_route = f"{top_routes.index[0][0]} -> {top_routes.index[0][1]}"

insights = [
    f"1. Weekly volume is remarkably even: {weekly.min():,}-{weekly.max():,} services per day "
    f"({weekly.idxmin()} lowest, {weekly.idxmax()} highest).",
    f"2. Day-of-week position explains little of the variance (r={r_ordinal:+.3f}, computed on "
    f"n=7 weekly aggregates — indicative, not definitive) and the weekend effect is "
    f"{strength(r_weekend)} (r={r_weekend:+.3f}) — demand is distributed across the week.",
    f"3. Weekend days still carry {weekend_share:.1f}% of all services "
    f"({we_mean:,.0f}/day vs {wd_mean:,.0f} on weekdays) — weekend operations are business-critical.",
    f"4. Strong hub concentration: the top-10 source stations carry {top10_hub_share}% of "
    f"services; the busiest single route is {busiest_route}.",
]
recommendations = [
    f"1. Peak capacity: keep reserve rakes/crew for {weekly.idxmax()} (busiest day) and schedule "
    f"maintenance windows on {weekly.idxmin()} (quietest day).",
    "2. Protect weekend frequency — cutting weekend services would remove ~"
    f"{weekend_share:.0f}% of weekly volume.",
    f"3. Prioritize infrastructure, staffing and punctuality monitoring at the dominant hubs: {top3_hubs}.",
    "4. Optimize at route level, not calendar level: since day-of-week explains little variance, "
    "focus on the top corridors and hub operations.",
]

report = "# Level 3 — Insights & Recommendations\n\n## Insights\n"
report += "\n".join(insights)
report += "\n\n## Recommendations\n" + "\n".join(recommendations) + "\n"

with open(f"{OUT_DIR}/insights_report.md", "w") as fh:
    fh.write(report)

print(report)
print(f"Saved {OUT_DIR}/insights_report.md")
```


### ▶ CODE CELL 8

```python

weekly_summary = weekly.rename("services").to_frame()
weekly_summary["share_pct"] = (weekly_summary["services"] / weekly_summary["services"].sum() * 100).round(2)
weekly_summary.to_csv(f"{OUT_DIR}/weekly_summary.csv")
print(weekly_summary)

print("\n Level 3 artifacts in", OUT_DIR + ":")
for f in sorted(os.listdir(OUT_DIR)):
    print("   -", f)

try:  # auto-download the report when running in Colab
    from google.colab import files
    files.download(f"{OUT_DIR}/insights_report.md")
except ImportError:
    pass
```


---

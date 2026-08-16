# CLAUDE.md

Guidance for Claude Code and other AI assistants working in this repository.

## What this repository actually is

A **portfolio / documentation repository**, not a software project. It contains a written
walkthrough of a feature-engineering and EDA exercise on the classic Indian domestic
flight-fare dataset. There is **no runnable code, no data, and no dependency manifest**
checked in.

```
.
├── README.md                          # Narrative write-up of the analysis + business insights
└── flights_feature_engineering.pdf    # 5.8 MB PDF export of the source Jupyter notebook
```

That is the entire tree — two files, two commits (`Add files via upload`, `MODEL EXPLAINATION`),
both made through the GitHub web uploader. There is no `.github/`, no CI, no tests, no
`requirements.txt`, no `.ipynb`.

**Implication:** you cannot run, lint, or test anything here. Do not invent a build step,
propose "fixing the failing tests," or claim to have executed the pipeline. Any code you are
asked to produce is *new* code being introduced to the repo for the first time.

## Reading the PDF (the only real source of truth)

The PDF is a **scanned/image-only export** — there is no embedded text layer:

```bash
pdftotext -layout flights_feature_engineering.pdf out.txt   # produces a 0-line file
```

To read it you must render the pages as images and view them:

```bash
apt-get update -q && apt-get install -y poppler-utils   # provides pdftoppm, needed by the Read tool
```

Then use the `Read` tool with a `pages` range (16 pages total, max 20 per call). Do **not**
try `pypdf` — the environment's `cryptography` module is broken and importing `pypdf` panics.

## The pipeline, as reconstructed from the notebook

Ordered by narrative, not by execution count (see *Out-of-order execution* below).

| # | Step | Code (as written in the notebook) |
|---|------|-----------------------------------|
| 1 | Imports | `pandas as pd`, `seaborn as sns`, `matplotlib.pyplot as plt`, `numpy as np`, `%matplotlib inline` |
| 2 | Load train | `train = pd.read_excel(r'C:\Users\krish\Downloads\Data_Train.xlsx')` — 10,683 rows |
| 3 | Load test | `test = pd.read_excel(r'C:\Users\krish\OneDrive\Desktop\Test_set.xlsx')` — 2,671 rows |
| 4 | Combine | `final = pd.concat([train, test])` |
| 5 | Split date | `final['journey_day'/'journey_month'/'journey_year'] = final['Date_of_Journey'].apply(lambda x: x.split('/')[0/1/2])` |
| 6 | Encode stops | `final['Total_Stops'].map({'non-stop':0, '2 stops':2, '3 stops':3, ...})` → `.fillna(0)` → `.astype('int32')` |
| 7 | Rename | `final.rename(columns={'Total_Stops':'Num of Stops'}, inplace=True)` |
| 8 | Arrival time | strip date suffix via `.split(' ')[0]`, then `Arrival_Hour` / `Arrival_Min` via `.split(':')`, then `drop('Arrival_Time')` |
| 9 | Departure time | `Dep_Hour` / `Dep_Min` via `.split(':')`, then `drop('Dep_Time')` |
| 10 | One-hot encode | `pd.get_dummies(final['Additional_Info'], dtype=int)` concatenated onto `final` (→ 37 cols) |
| 11 | Drop columns | `drop('Additional_Info')`, `drop('Route')` (→ 35 cols) |
| 12 | Datetime cast | `final['Date_of_Journey'] = pd.to_datetime(final['Date_of_Journey'])` |

### Raw dataset schema

`Airline`, `Date_of_Journey` (`DD/MM/YYYY` string), `Source`, `Destination`,
`Route` (`BLR → NAG → DEL`), `Dep_Time`, `Arrival_Time` (sometimes `HH:MM DD Mon`),
`Duration` (`2h 50m`), `Total_Stops`, `Additional_Info`, `Price` (train only).

`Additional_Info` categories: `No info`, `No Info`, `In-flight meal not included`,
`No check-in baggage included`, `1 Short layover`, `1 Long layover`, `2 Long layover`,
`Change airports`, `Business class`, `Red-eye flight`.

### EDA findings reported

- Mean price by stop count: 0 → ₹8,461 · 2 → ₹12,716 · 3 → ₹13,112 · 4 → ₹17,686
- Jet Airways Business has the highest mean fare (~₹60k); Trujet the lowest (~₹4k)
- Top 3 carriers by volume: Jet Airways 49.95%, IndiGo 26.98%, Air India 23.07%
- 1 red-eye flight; 8 airport changes; 1,349 flights departing on the 1st of a month

## Known issues — do not propagate these

If you are asked to write code, reproduce the notebook, or update the README, be aware the
source notebook contains real defects. Fix them in new code; flag them rather than silently
copying them.

1. **`'1 stop'` is missing from the stops map.** The `.map()` dict covers `non-stop`, `2 stops`,
   `3 stops`, `4 stops` but not `1 stop`, so every single-stop flight becomes `NaN` and is then
   swept to `0` by `.fillna(0)`. `unique()` confirms `[0, 2, 3, 4]` — there is no `1`.
   **The README's claim that `1 stop → 1` is therefore wrong**, and the "price rises with stops"
   finding is computed with one-stop flights mislabelled as non-stop.
2. **`fillna(0)` conflates missing with non-stop**, compounding the above.
3. **Test rows have no `Price`.** `pd.concat([train, test])` yields `NaN` prices for 2,671 rows;
   every `groupby(...)['Price'].mean()` silently drops them. Train/test are never separated again.
4. **Index is not reset after concat**, so positional labels repeat (`0..10682`, then `0..2670`).
5. **`Duration` is never parsed** — it stays as the string `"2h 50m"` and is unusable as a feature.
6. **Derived time/date features stay as strings** (`dtype: object`), never cast to numeric.
7. **`journey_year` is constant** (2019) and carries no signal.
8. **Hardcoded Windows absolute paths** (`C:\Users\krish\...`) — not portable.
9. **Out-of-order execution.** Cell counts jump `5 → 108 → 123 → 15 → 133 → 19 → 139 → 32 → 156 → 36`.
   `Dep_Time` is dropped at `In[16]` yet referenced again at `In[133]`. The PDF is **not** a
   reproducible top-to-bottom run; treat the narrative order, not the cell numbers, as canonical.
10. **Deprecation warnings** left in output: `to_datetime` without `dayfirst=True` (dates are
    `DD/MM/YYYY`, so this misparses ambiguous days), and seaborn `palette` without `hue`.

## Conventions

**Documentation.** The README is emoji-sectioned, heavily headed, and business-insight oriented.
Match that voice when editing it. It cites PDF page numbers (e.g. "page 14") — keep those
accurate if content moves. Note the README also contains typos in the source material's spirit
(`COVERTING`, `JOUNEY`, `EXPLAINATION` in a commit message); do not "correct" the PDF, which is
a fixed binary export, but do keep new prose clean.

**If you add code**, there is no precedent to follow, so establish a sane one:
- Put the notebook or scripts at the repo root or under `src/` and add a `requirements.txt`
  (`pandas`, `numpy`, `matplotlib`, `seaborn`, `openpyxl` — the last is required by `read_excel`).
- Read data from a relative path (e.g. `data/Data_Train.xlsx`) with the data gitignored, never
  from an absolute user path.
- The notebook style is procedural pandas with `.apply(lambda ...)` string splitting and
  `inplace=True`. Prefer vectorized `.str` accessors and `pd.to_datetime(..., dayfirst=True)`
  in anything new, and say so if the user expects a faithful transcription instead.

**Data files are not in the repo.** `Data_Train.xlsx` and `Test_set.xlsx` are the standard
public flight-fare dataset. Do not fabricate them; ask the user to supply them if a task
requires actually running the pipeline.

## Git workflow

- Default branch: `main`. Active working branch: `claude/claude-md-docs-zmjqjk`.
- Develop on the assigned branch, commit with descriptive messages, and push with
  `git push -u origin <branch>`. Never push directly to `main`.
- No CI runs on push, so nothing validates a commit automatically — review changes yourself
  before pushing.
- The PDF is a 5.8 MB binary. Avoid re-committing it; git cannot diff it meaningfully and each
  copy adds ~6 MB to history.

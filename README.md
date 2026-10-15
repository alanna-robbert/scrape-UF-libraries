# UF Libraries Occupancy Scraper

An hourly scraper I built to report and write a story about <h = "https://www.alligator.org/article/2025/12/marston-will-offer-later-hours-this-spring-but-data-shows-library-use-drops-off-at-night">occupancy rates</h> at the University of Florida's <h = "https://uflib.ufl.edu/">George A. Smathers Libraries</h>.

The scraper records how many people are in each UF library, using the live counts on the [UF Libraries status page](https://www.uflib.ufl.edu/status/). GitHub Actions runs it on a schedule and commits each new reading to a CSV in this repo.

## Repository structure

```text
UF-libraries-scraper/
├── .github/workflows/
│   ├── lib-scraper.yml        # all libraries, daytime and evening
│   └── mar-scraper.yml        # Marston only, overnight
├── data/
│   ├── library_occupancy-archive.csv
│   └── library_occupancy.csv
├── scraper/
│   └── scraper.py
├── LICENSE
└── README.md
```

## The data

### Files

| File | Coverage | Status |
|---|---|---|
| `data/library_occupancy-archive.csv` | Oct 2 – Dec 30, 2025 | Finished; no longer updated |
| `data/library_occupancy.csv` | June 8, 2026 – present | Updated automatically by GitHub Actions |

There is no data between Dec 30, 2025 and June 8, 2026. The scraper wasn't running during that time. The gap doesn't mean the libraries were empty.

### Columns

Both files share the same layout: one row per scrape and one column per library.

| Column | Description |
|---|---|
| `timestamp` | When the scrape ran, in **US Eastern time** (`YYYY-MM-DD HH:MM:SS`) |
| `currocclw` | Library West |
| `currocc` | Marston Science Library |
| `currocchscl` | Health Science Center Library |
| `curroccafa` | Architecture & Fine Arts Library |
| `curroccedu` | Education Library |
| `curroccsmath` | Smathers Library |
| `currocclacc` | Latin American & Caribbean Collection |
| `curroccgrr` | Smathers Grand Reading Room |
| `curroccmap` | Map & Imagery Library |
| `curroccpanama` | UF Panama |

<!-- TODO: confirm each library name above against the status page -->

Each library column holds the number of people the status page showed for that library when the scrape ran. A **blank cell** means no number was recorded. Usually that's because the library wasn't scraped at that hour (see the schedule below). Sometimes the page didn't return a value.

## How it works

`scraper/scraper.py` loads the status page in headless Chrome using Selenium. It reads the occupancy count for each library ID passed to it and appends one row to `data/library_occupancy.csv`.

Two GitHub Actions workflows run the scraper and commit the updated CSV:

| Workflow | Libraries | Scheduled hours (ET) |
|---|---|---|
| `lib-scraper.yml` | All 10 | Hourly, 9 AM – 2 AM |
| `mar-scraper.yml` | Marston only | Hourly, 3 AM – 8 AM |

During the overnight hours, Marston is the only library open (24/5 hours). That's why overnight rows have a count only in the `currocc` column.

You can also start either workflow by hand from the repo's **Actions** tab.

## Limitations

- **Readings are not truly hourly.** GitHub Actions doesn't guarantee that scheduled runs happen. When GitHub's servers are busy, runs start late and some are skipped entirely. In practice, the data has about 4–6 readings a day, at irregular times. Analyze each row by its `timestamp`, not by its scheduled hour. 
- **Counts come from UF.** This repo records the numbers the UF Libraries status page displays. It doesn't independently check how accurate they are.

## Running it locally

You need Python 3.9 or newer, plus Chrome or Chromium installed.

```bash
pip install selenium pytz
python scraper/scraper.py --ids currocclw currocc currocchscl
```

Pass any combination of the column IDs listed above to `--ids`. The scraper adds one row to `data/library_occupancy.csv`. Libraries you didn't request are left blank.
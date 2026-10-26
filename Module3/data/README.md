# Module 3 data

| File | What it is |
|---|---|
| `class_listings_master_2026-10-22.csv` | **The class-wide regex dataset** (optional for the A08 midterm): 10,241 cars from 22 students' Lab 4 `listings_master.csv` files, scraped Sep 30 to Oct 22, 2026, each from a different Craigslist area |
| `lab4_listings_master_snapshot.csv` | Dr. Dave's own `listings_master.csv` on Oct 13 (the Lab 4 backup data) |
| `lab3_hartford_sample.zip` | The Lab 3 backup scrape (raw ads, for the regex detective work) |

## The class-wide regex dataset
One row per car, the same regex fields your Lab 4 scraper writes:

| Column | Meaning |
|---|---|
| `post_id` | The ad's id (unique; no car appears twice) |
| `scraper_id` | Which student's scraper found it, as an anonymous code `S01` to `S22` (one Craigslist area each) |
| `scraped_at` | When it was scraped (UTC) |
| `price`, `year`, `make`, `model`, `mileage` | The fields the regular expressions pulled out of the ad |

**Raw, on purpose.** It has the same junk your own data has (prices of $0 or $1, mileage like 9,999,999, years in the
future): clean it the same way you clean your own cars. It has **regex fields only**: the Gemini fields exist only for
your own cars, so use it to ask "does a regex model trained on 10,000 cars from 22 markets beat one trained on my own
market?", tested on **your** new cars.

Load it in Colab:
```python
import pandas as pd
url = "https://raw.githubusercontent.com/drdave-teaching/OPIM5512-notebooks/main/Module3/data/class_listings_master_2026-10-22.csv"
cars = pd.read_csv(url, dtype={"post_id": str})
```

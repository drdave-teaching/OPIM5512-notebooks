# Lab 5 training data: Bradley weather + Connecticut electricity load, 2023–2025

**File:** `bdl_ct_load_hourly_2023_2025.csv`, one row per hour, **26,304 rows** (3 Connecticut calendar years:
midnight Jan 1, 2023 to 11 p.m. Dec 31, 2025, Eastern time). Built step by step in
`../Lab5_Building_the_Training_Data.ipynb` from the untouched downloads in `raw/`.

| Column | Meaning | Units |
|---|---|---|
| `hour_utc` | Start of the hour, in UTC (the row's unique key; never jumps for daylight saving) | timestamp |
| `temp_f` | Air temperature at Bradley Airport (BDL) | °F |
| `dewpoint_f` | Dew point | °F |
| `rel_humidity_pct` | Relative humidity | % |
| `feels_like_f` | Heat index / wind chill | °F |
| `wind_speed_mph` | Sustained wind | mph |
| `wind_gust_mph` | Gust (equals the wind speed when no gust was reported) | mph |
| `wind_dir_deg` | Direction the wind blows from (0 when calm) | degrees |
| `pressure_inhg` | Altimeter pressure | inches of mercury |
| `visibility_mi` | Visibility (10 = 10 or more) | miles |
| `cloud_cover_pct` | Sky covered by the thickest cloud layer | % |
| `precip_in` | Rain/melted snow in the hour (0.001 = a trace) | inches |
| `is_rain`, `is_snow`, `is_fog` | Rain or drizzle / snow, sleet or hail / fog or mist reported | 0 or 1 |
| `weather_imputed` | 1 = no airport report this hour, values were filled (22 hours) | 0 or 1 |
| `load_mw` | Average electricity used in the Connecticut zone during the hour | MW |
| `hour_local` | The same hour on a Connecticut clock, for humans | text |

**How the weather was cleaned**
1. Bradley sends a routine report at :51 every hour plus "special" reports when the weather changes fast. Each report is
   rounded to the nearest hour (10:51 → 11:00, the start of the hour ISO New England labels "11:00"). One report per
   hour: the routine one, otherwise the one closest to the hour.
2. Lined up against a complete list of hours, so missing hours show up (22 of 26,304).
3. Units matched to the forecast.weather.gov forecast: knots → mph, sky codes (CLR, FEW, SCT, BKN, OVC, VV) → percent,
   using the thickest of up to 4 cloud layers. Trace rain "T" → 0.001 in. No gust reported → gust = wind speed.
4. Ranges checked (−9 to 99 °F, humidity ~10–100 %, wind ≤ 35 mph): no broken-sensor values.
5. Gaps filled: numbers by straight-line interpolation (6 hours at most), wind direction and yes/no flags carry the last
   value, rain = 0. Every filled hour has `weather_imputed = 1`.

**How the load was pulled:** ISO New England web services, real-time hourly demand, Connecticut load zone
(location 4004), one day at a time. March days have 23 hours and November days 25; in UTC nothing collides.

**Credits:** weather from the Iowa State Mesonet (mesonet.agron.iastate.edu) archive of NWS/FAA airport reports;
load from ISO New England (iso-ne.com, ISO Express web services).

# Lab 5 training data: Bradley weather + Connecticut electricity load, Jan 2023 to Oct 2026

**File:** `bdl_ct_load_hourly_2023_2026.csv`, one row per hour, **33,215 rows** (midnight Jan 1, 2023 to
11 p.m. Oct 15, 2026, Connecticut time: 1,384 days × 24, minus 1 because there are four spring-forward days and only three
fall-backs in that span). Built step by step in
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
| `weather_imputed` | 1 = no airport report this hour, values were filled (33 hours) | 0 or 1 |
| `load_mw` | Average electricity used in the Connecticut zone during the hour | MW |
| `hour_local` | The same hour on a Connecticut clock, for humans | text |

**One airport only: Bradley (BDL).** Every weather column comes from a single station, Bradley International Airport
in Windsor Locks. Connecticut's load is statewide, so the weather in Bridgeport, New Haven or New London on a given hour
can differ from Bradley's (sea breeze along the shore, snow inland). Bradley alone is a fine model, and it is all the
labs need.

**Optional (final project): add more airports.** If you want more weather information, pull other airport stations from
the Iowa State Mesonet archive (https://mesonet.agron.iastate.edu/request/download.phtml) and clean them the same way
as `../Lab5_Building_the_Training_Data.ipynb`. Claude can help you write that code. Good candidates:
- Connecticut: Hartford (HFD), Bridgeport (BDR), New Haven (HVN), Groton/New London (GON), Danbury (DXR),
  Oxford (OXC), Meriden (MMK), Windham (IJD)
- Neighbors: White Plains NY (HPN), Westchester and Long Island stations, Westfield/Springfield MA (BAF, CEF),
  Worcester MA (ORH), Providence RI (PVD), Westerly RI (WST)

The catch: production has to have the same columns. A model trained on extra stations needs **those stations'
forecasts** every hour too, but `scraper-weather-forecast` only downloads Bradley's. So adding stations also means
adding their locations to `scraper-weather-forecast` (forecast.weather.gov works for any latitude/longitude),
building the same columns in `make_features` in your fork's `forecast-24h/main.py`, and updating the feature check in
Lab 5 Part 7a. (The A09 file in `a09/` has six Connecticut airports, but its problems were planted on purpose for that
assignment, so pull fresh data instead.)

**How the weather was cleaned**
1. Bradley sends a routine report at :51 every hour plus "special" reports when the weather changes fast. Each report is
   rounded to the nearest hour (10:51 → 11:00, the start of the hour ISO New England labels "11:00"). One report per
   hour: the routine one, otherwise the one closest to the hour.
2. Lined up against a complete list of hours, so missing hours show up (33 of 33,215).
3. Units matched to the forecast.weather.gov forecast: knots → mph, sky codes (CLR, FEW, SCT, BKN, OVC, VV) → percent,
   using the thickest of up to 4 cloud layers. Trace rain "T" → 0.001 in. No gust reported → gust = wind speed.
4. Ranges checked (−9 to 99 °F, humidity ~10–100 %, wind ≤ 35 mph): no broken-sensor values.
5. Gaps filled: numbers by straight-line interpolation (6 hours at most), wind direction and yes/no flags carry the last
   value, rain = 0. Every filled hour has `weather_imputed = 1`.

**How the load was pulled:** ISO New England web services, real-time hourly demand, Connecticut load zone
(location 4004), one day at a time. March days have 23 hours and November days 25; in UTC nothing collides.
`raw/ct_hourly_load_2023_2025_raw.csv` (the original 3 years) is kept because A09 uses it.

**Credits:** weather from the Iowa State Mesonet (mesonet.agron.iastate.edu) archive of NWS/FAA airport reports;
load from ISO New England (iso-ne.com, ISO Express web services).

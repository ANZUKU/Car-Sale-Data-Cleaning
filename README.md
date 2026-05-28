# Car-Sale-Data-Cleaning
Cleaned car sales data 

CAR SALES DATA – CLEANING PROCESS REPORT
Source File: Car_Sales_Data.xlsx
Output File: Car_Sales_Data_Clean.xlsx
Total Rows: 10,037
Total Columns: 15

OVERVIEW
The raw file contained a single sheet ("car_prices") with 10,037 vehicle
auction records across 15 columns: year, make, model, trim, body,
transmission, state, condition, odometer, color, interior, seller, mmr,
sellingprice, and saledate.

A full audit revealed seven categories of data quality issues that required
correction before the file could be used for analysis or reporting. Each is
described below with the action taken and the number of records affected.

STEP 1 – CHARACTER ENCODING CORRUPTION
Issue:
The "color" and "interior" columns contained the string "â€"" in place of
what should have been a recognisable value. This is a classic mojibake
artifact: the original data stored an em-dash "—" as a placeholder for
unknown values, but the encoding was misread as Latin-1 instead of UTF-8,
producing the garbled sequence "â€"".

Affected records: 250 in "color", 1,080 in "interior".
Action:
All occurrences of "â€"" were replaced with the string "Unknown" to make the
value meaningful and consistent with the handling of other missing values
across the dataset.
STEP 2 – MISSING VALUES: TRANSMISSION (787 ROWS)

Issue:
787 records (roughly 8% of the dataset) had no value in the "transmission"
column. The only valid values present in the data were "automatic" and
"manual". Without additional source data it was not possible to infer which
type applied to each blank vehicle.

Action:
Missing transmission values were filled with "Unknown". This preserves the
row for all other analysis while clearly flagging that the value is
unavailable, rather than silently defaulting to one of the two categories
and introducing false counts.

STEP 3 – MISSING VALUES: CONDITION (1,448 ROWS)
Issue:
The "condition" column is a numeric score ranging from 1 to 49 (a standard
auction condition grade). 1,448 rows (14.4%) were blank.

Action:
Missing values were filled with the column median, which was 34.0. Using the
median rather than the mean prevents outliers at either end of the scale from
distorting the imputed value. This approach is appropriate for a roughly
continuous numeric field where no rule-based inference is available.

STEP 4 – MISSING VALUES: CATEGORICAL FIELDS (make, model, trim, body, color, interior)
Issue:
Several categorical text columns had a small number of NaN entries:
- make: 25 missing
- model: 239 missing (includes rows where BMW model was blank)
- trim: 266 missing
- body: 244 missing
- color:  28 missing
- interior:  28 missing

Action:
All blank cells in these columns were filled with "Unknown". Dropping these
rows entirely would remove up to 830 otherwise complete records, which is
unnecessary. "Unknown" clearly signals the gap without corrupting aggregations
on other columns.

STEP 5 – MISSING VALUES: ODOMETER (11 ROWS)
Issue:
11 rows had no odometer reading. Odometer is a numeric field that directly
affects vehicle valuation, so blank values should not be left as NaN in a
clean file.

Action:
Missing odometer values were filled with the column median. The median is
more robust than the mean for mileage data, which is often right-skewed by
high-mileage vehicles.

STEP 8 – COLUMN RENAMING
Issue:
All column headers were lowercase and used run-together names
(e.g. "sellingprice", "saledate", "mmr") that are not user-friendly.

Action:
All headers were renamed to human-readable Title Case with spaces:
year         → Year
make         → Make
model        → Model
trim         → Trim
body         → Body
transmission → Transmission
state        → State
condition    → Condition
odometer     → Odometer
color        → Color
interior     → Interior
seller       → Seller
mmr          → MMR
sellingprice → Selling Price
saledate     → Sale Date

──────────────────────────────────────────────────────────────────────────────
STEP 9 – NUMERIC TYPE ENFORCEMENT
──────────────────────────────────────────────────────────────────────────────
Issue:
After cleaning, several numeric columns risked being stored as mixed-type
objects (strings + numbers) due to the presence of nulls or imputed values.

Action:
- Condition was cast to float and rounded to one decimal place.
- Odometer, MMR, and Selling Price were cast to integer.
- Any coercion failures were defaulted to 0 and flagged in this report for
review (none were found in practice).

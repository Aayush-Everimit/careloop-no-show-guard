# CareLoop Clinics — No-Show Guard

**End-to-end capstone: Python, SQL, Machine Learning and GenAI**

Public repository URL: add the public GitHub URL here before submission.

This run was executed locally on 24 November 2026 from the three original CSVs in `data/`. Parts 1–3 completed and were checked against the SQL database. Part 4 is ready to run once `GEMINI_API_KEY` is set in `.env`. Live Open-Meteo temperatures and Gemini labels can change on a later run.

## Project structure

```text
CareLoop-NoShow-Guard/
├── data/
│   ├── appointments.csv          original, 1,020 rows
│   ├── clinics.csv               original, 8 rows
│   ├── followup_notes.csv        original, 30 rows
│   └── README.md
├── 01_ingest_clean.ipynb         executed
├── 02_queries.sql                executed in SQLite
├── 03_model.ipynb                executed
├── 04_genai.ipynb                waits for GEMINI_API_KEY
├── Capstone_Project_original.ipynb
├── appointments_clean.csv
├── clinics_enriched.csv
├── rejected_rows.json
├── load_data.sql
├── test_risk_scores.csv
├── requirements.txt
├── .env.example
├── .gitignore
└── README.md
```

`SQLite.db` is created when the queries are run and is gitignored. `.env` is gitignored and is the only place the Gemini key should live.

## Setup

```text
python -m pip install -r requirements.txt
copy .env.example .env
```

Put a Gemini API key in `.env` as `GEMINI_API_KEY=...`. Do not paste the key into a notebook, and do not commit `.env`.

## Run order

1. Run `01_ingest_clean.ipynb` from the project root. It reads `data/` and writes `rejected_rows.json`, `appointments_clean.csv`, `clinics_enriched.csv`, and `load_data.sql`.
2. In SQLite, create the tables from `02_queries.sql`, then execute `load_data.sql`, then run the `SELECT` statements in `02_queries.sql`. The duplicate-key `INSERT` stays commented in the file. Running it fails on purpose: `UNIQUE constraint failed: appointments.appointment_id`.
3. Run `03_model.ipynb`. It writes `test_risk_scores.csv`.
4. After `GEMINI_API_KEY` is set, run `04_genai.ipynb`. It writes `notes_analysed.csv` (10 rows, 8 columns) and prints the prompt comparison. The key is loaded from `.env` and deleted from the session. In Colab, a blank key still falls back to the prompt.

`Capstone_Project_original.ipynb` is the supplied Colab notebook. Its data paths point at `/content/`, and its saved outputs are the earlier Colab run. The local run used the numbered notebooks and `02_queries.sql`.

## Measured results

### Task 1 — Ingest and clean

| Check | Result |
|---|---|
| CSV rows read by the type parser | 1,020 |
| Accepted / rejected by the type parser | 988 / 32 |
| Exact duplicate rows dropped | 20, leaving 1,000 |
| Rows after dropping missing appointment date or status | 990 |
| Invalid ages set to missing, then imputed | 5 invalid, 17 ages imputed with the median |
| Distance capped / fee capped | 26 / 3 |
| Lead days | 1 to 45 |
| No-shows | 218 / 990 (22.0%) |
| Column means (age, distance_km, fee_inr, lead_days) | 48.17, 9.50, 635.71, 11.82 |
| Lead days over 14 | 296 rows (29.9%) |
| Distance \|z\| over 2 | 53 |
| Unique `appointment_id` and missing export cells | 990 and 0 |
| `load_data.sql` inserts | 998 (8 clinics + 990 appointments) |

Open-Meteo temperature at the first forecast hour in this run:

| Clinic | City | temp_c |
|---|---|---|
| Kothrud Family Clinic | Pune | 18.3 |
| Hadapsar Care Centre | Pune | 20.0 |
| Sitabuldi Health Point | Nagpur | 20.4 |
| Malviya Nagar Clinic | Jaipur | 16.2 |
| Vaishali Nagar Clinic | Jaipur | 16.2 |
| Edappally Medical Centre | Kochi | 26.2 |
| Vijay Nagar Clinic | Indore | 16.6 |
| Gomti Nagar Care Point | Lucknow | 16.9 |

### Task 2 — SQL

Load check: 8 clinics, 990 appointments, 218 no-shows.

Clinics with at least 95 appointments:

| Clinic | City | Appointments | No-shows | No-show % |
|---|---|---:|---:|---:|
| Hadapsar Care Centre | Pune | 143 | 39 | 27.3 |
| Malviya Nagar Clinic | Jaipur | 142 | 38 | 26.8 |
| Kothrud Family Clinic | Pune | 160 | 36 | 22.5 |
| Vijay Nagar Clinic | Indore | 101 | 21 | 20.8 |
| Edappally Medical Centre | Kochi | 142 | 23 | 16.2 |
| Vaishali Nagar Clinic | Jaipur | 130 | 18 | 13.8 |

All eight clinics, highest rate first: Gomti Nagar Care Point 33.3% (90 appointments), then Hadapsar 27.3%, Malviya Nagar 26.8%, Kothrud 22.5%, Vijay Nagar 20.8%, Edappally 16.2%, Sitabuldi Health Point 15.9% (82 appointments), Vaishali Nagar 13.8%.

Lead-time bands:

| Lead band | Appointments | No-shows | No-show % |
|---|---:|---:|---:|
| 1–7 | 390 | 63 | 16.2 |
| 8–14 | 304 | 53 | 17.4 |
| 15–30 | 253 | 76 | 30.0 |
| 31+ | 43 | 26 | 60.5 |

Monthly no-show rate and change from the previous month:

| Month | Appointments | No-show % | Percentage-point change |
|---|---:|---:|---:|
| 2025-01 | 157 | 24.8 | — |
| 2025-02 | 151 | 20.5 | -4.3 |
| 2025-03 | 142 | 20.4 | -0.1 |
| 2025-04 | 177 | 20.9 | +0.5 |
| 2025-05 | 198 | 23.7 | +2.8 |
| 2025-06 | 165 | 21.2 | -2.5 |

Rank inside each city versus the overall rate (22.02%): Gomti Nagar, Hadapsar, Malviya Nagar, and Kothrud are above the overall rate. Vijay Nagar, Edappally, Sitabuldi, and Vaishali Nagar are below it. In Jaipur, Malviya Nagar ranks 1 and Vaishali Nagar ranks 2. In Pune, Hadapsar ranks 1 and Kothrud ranks 2.

### Task 3 — Model

Split: 792 train rows, 198 test rows, 49 test no-shows. Baseline accuracy from always predicting Attended: 0.7525. Eleven feature columns after encoding.

| Model | Test accuracy | Test recall | Test ROC-AUC |
|---|---:|---:|---:|
| Logistic regression | 0.7879 | 0.3469 | 0.7889 |
| Decision tree, depth 3 | 0.7778 | 0.2245 | 0.6997 |
| Random forest, 100 trees | 0.7475 | 0.3061 | 0.7057 |

An unrestricted tree reached depth 15, with training accuracy 1.0000 and test accuracy 0.6717. The depth-3 tree trained at 0.8220 and tested at 0.7778.

Five-fold training ROC-AUC: logistic regression 0.7737, depth-3 tree 0.6784, random forest 0.7466, tuned forest (`max_depth=3`, `n_estimators=200`) 0.7673. Logistic regression is the selected model. Random-forest importance ranks lead days, age, and distance first. A reminder is associated with lower no-show odds: `exp(coef / scale)` for `reminder_sent` is 0.3094.

Out-of-fold threshold check on the training set, then one test evaluation:

| Threshold | Where | Precision | Recall | Flagged |
|---|---|---:|---:|---:|
| 0.50 | training, out of fold | 0.5753 | 0.2485 | 73 |
| 0.35 | training, out of fold | 0.5130 | 0.4675 | 154 |
| 0.35 | test, once | 0.5714 | 0.5714 | 49 / 198 |

`test_risk_scores.csv` has 198 rows and 49 flags. Test ROC-AUC at this stage is 0.7889.

### Task 4 — GenAI notes

`04_genai.ipynb` loads `GEMINI_API_KEY` from `.env` and does not store the key in the notebook. This environment has no key, so the Gemini calls were not run and `notes_analysed.csv` was not created. After the key is added, run the notebook and confirm the export has 10 rows and 8 columns, then clear any output that could contain the key.

## Recommendation

Call the longest-lead bookings first. No-show rate is 16.2% when the visit is 1–7 days out and 60.5% when it is 31 days or more. Bookings more than 14 days ahead are 296 of 990 visits (29.9%) and are the practical outreach pool, with the 31+ day band at the front of the list.

Among clinics that already have at least 95 appointments, start with Hadapsar Care Centre (27.3%) and Malviya Nagar Clinic (26.8%). Gomti Nagar Care Point is at 33.3% on 90 appointments, so it stays on the local watch list even though the 95-appointment filter leaves it out. Vaishali Nagar Clinic is the low-rate comparison site at 13.8%.

Score the call list with logistic regression. It has the best training cross-validation ROC-AUC (0.7737) and a test ROC-AUC of 0.7889. At threshold 0.35 the test list flags 49 of 198 bookings and recovers 57.1% of test no-shows at 57.1% precision. Use 0.35 when the desk can make about one call for every four upcoming visits. A smaller calling team should move the threshold up toward 0.50, where training recall falls to 0.2485 and the list gets shorter. Prior no-shows, lead time, and fee push risk up. A recorded reminder multiplies the odds of a no-show by about 0.31, so send the reminder before spending a phone call.

## Limitations

- Median age, median distance, and the distance and fee caps were calculated on all cleaned rows before the train/test split, so the test rows took part in the cleaning rules.
- The split is by appointment. The same patient can appear in both the training set and the test set.
- The split is random. It does not train on earlier months and test on later months.
- Threshold 0.35 matches one assumed call-list size. A different front-desk capacity needs a different threshold, chosen again from training out-of-fold scores.
- The associations are observational. Lead time, reminders, and clinic rates move together with no-shows in this extract. They do not by themselves show that changing one of them causes the rate to change.
- The note analysis, once the API key is set, covers only the first 10 of 30 follow-up notes.

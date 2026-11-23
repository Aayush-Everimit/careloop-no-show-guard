# Generated output status

Produced by the local run on 24 November 2026.

| File | Produced by | Status |
|---|---|---|
| rejected_rows.json | Task 1 | 32 rejected rows |
| appointments_clean.csv | Task 1 | 990 rows, 16 columns, 0 missing values |
| clinics_enriched.csv | Task 1 | 8 clinics with Open-Meteo `temp_c` |
| load_data.sql | Task 1 | 998 INSERT statements |
| test_risk_scores.csv | Task 3 | 198 test rows, 49 flagged at threshold 0.35 |
| notes_analysed.csv | Task 4 | Not created. Set `GEMINI_API_KEY` in `.env`, then run `04_genai.ipynb` |
| SQLite.db | Task 2 | Local database after loading `load_data.sql`. Gitignored |

SQL load check on that database: 8 clinics, 990 appointments, 218 no-shows. The duplicate `appointment_id` insert failed with `UNIQUE constraint failed: appointments.appointment_id`.

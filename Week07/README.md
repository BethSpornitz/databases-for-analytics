# Module 7 – Final Project  
Beth Spornitz
February 11, 2026

# Healthcare No-Show Database Project  
Database: week07_noshows  
Total Appointment Records: 106,988  
Tables: staging_noshows, patients, appointments, locations

## Locating My Data

### Initial Data Source

The dataset used for this project is a healthcare appointment no-show dataset downloaded in CSV format. The dataset contains records of scheduled medical appointments and whether the patient showed up for their appointment.

### Data Format

- **File type:** CSV  
- **Total rows:** 106,988  
- **Total columns:** 15  
- **Data types present:** string, numeric, date, and boolean-like values  

The raw file was stored locally in the project directory:  Week07/data_raw/healthcare_noshows_appt.csv

This dataset exceeds the minimum project requirements because:

- It contains more than 100,000 rows.
- It includes multiple data types (date, numeric, and string).
- It supports relational modeling into multiple normalized tables.

## Installing My Data

After locating the dataset, I created a PostgreSQL database named `week07_noshows` using pgAdmin.

The original CSV file was first loaded into a staging table called `staging_noshows`. This allowed me to preserve the raw data exactly as it appeared in the CSV file.

From the staging table, I normalized the data into three relational tables:

- `patients`
- `appointments`
- `locations`

This transformation improved data organization and reduced redundancy.

The final database structure includes:

- A fact-style table (`appointments`) containing appointment-level records.
- A patient dimension table (`patients`) containing demographic and health information.
- A location dimension table (`locations`) containing neighborhood information.

The database now contains more than 100,000 appointment records across multiple related tables, exceeding the minimum project requirements.

## Verifying My Data

To verify the database structure and integrity:

- I confirmed the active database using `SELECT current_database();`
- I listed all tables using `information_schema.tables`
- I confirmed all column names and data types using `information_schema.columns`
- I verified row counts and sample data using `SELECT * FROM table LIMIT 20;`

The database includes:

- String data types (e.g., gender, neighbourhood_name)
- Numeric data types (e.g., age, sms_received)
- Date and timestamp data types (appointment_day, scheduled_day)

Screenshots verifying these steps are included in the screenshots folder.

## Data Dictionary

| Table Name         | Column Name        | Data Type                      |
|--------------------|-------------------|--------------------------------|
| staging_noshows    | appointment_id     | bigint                         |
| staging_noshows    | patient_id         | bigint                         |
| staging_noshows    | neighbourhood_id   | integer                        |
| staging_noshows    | scheduled_day      | timestamp without time zone    |
| staging_noshows    | appointment_day    | date                           |
| staging_noshows    | sms_received       | integer                        |
| staging_noshows    | showed_up          | character varying              |
| staging_noshows    | date_diff          | integer                        |
| staging_noshows    | gender             | character varying              |
| staging_noshows    | age                | integer                        |
| staging_noshows    | scholarship        | integer                        |
| staging_noshows    | hipertension       | integer                        |
| staging_noshows    | diabetes           | integer                        |
| staging_noshows    | alcoholism         | integer                        |
| staging_noshows    | handcap            | integer                        |
| appointments       | appointment_id     | bigint                         |
| appointments       | patient_id         | bigint                         |
| appointments       | neighbourhood_id   | integer                        |
| appointments       | scheduled_day      | timestamp without time zone    |
| appointments       | appointment_day    | date                           |
| appointments       | sms_received       | integer                        |
| appointments       | showed_up          | character varying              |
| appointments       | date_diff          | integer                        |
| patients           | patient_id         | bigint                         |
| patients           | gender             | character varying              |
| patients           | age                | integer                        |
| patients           | scholarship        | integer                        |
| patients           | hipertension       | integer                        |
| patients           | diabetes           | integer                        |
| patients           | alcoholism         | integer                        |
| patients           | handcap            | integer                        |
| locations          | neighbourhood_id   | integer                        |
| locations          | neighbourhood_name | character varying              |


## Interesting Queries and Insights

### Join Example

I joined the `appointments` and `locations` tables to associate each appointment with its neighborhood name.

This verified the relational integrity of the database and demonstrated successful normalization.

---

### No-Show Rate by Neighborhood

Using a GROUP BY query, I calculated no-show percentages by neighborhood.

Results show that no-show rates vary significantly by neighborhood, indicating potential geographic or socioeconomic differences.

---

### No-Show Rate by SMS Reminder

I grouped appointments by `sms_received` to determine whether reminder messages impacted attendance.

Interestingly:

- Patients who did NOT receive SMS reminders had a no-show rate of approximately **16.73%**
- Patients who DID receive SMS reminders had a higher no-show rate of approximately **27.67%**

This suggests that SMS reminders may be targeted toward higher-risk patients rather than directly reducing no-shows.

---

### No-Show Rate by Age Band

Appointments were grouped into age categories:

- 0–17: 22.48%
- 18–29: 24.64%
- 30–44: 21.82%
- 45–59: 17.88%
- 60–74: 15.04%
- 75+: 16.03%

Younger adults (18–29) showed the highest no-show rates, while older adults (60–74) showed the lowest.

This demonstrates how relational databases can be used to extract meaningful operational insights from healthcare data.

## Obstacles Encountered

One challenge was determining how to structure the data relationally. The original dataset existed as a single flat CSV file.

To improve database design and meet project requirements, I created a staging table for raw import and then separated the data into normalized tables (patients, appointments, and locations).

Another challenge was correctly interpreting the `showed_up` field, which was stored as text ('TRUE'/'FALSE') rather than a boolean. This required conditional logic in aggregation queries to calculate accurate no-show rates.

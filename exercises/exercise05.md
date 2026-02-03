# Exercise 05: SQLDA Database - Dates, Data Quality, Arrays, and JSON

- Name:
- Course: Database for Analytics
- Module:
- Database Used:  `sqlda` (Sample Datasets)
- Tools Used: PostgreSQL (pgAdmin or psql)

---

## Instructions

- Use the **sqlda** database from the "Loading the Sample Datasets" instructions.
- For each SQL task:
  - Include your SQL in a fenced code block
  - Execute it and include a **screenshot** showing the query and results
- Store screenshots in the `screenshots/` folder and embed them below each answer.
- For explanation questions:
  - Write your answer in complete sentences
  - Include a screenshot if requested

---

## Question 1

Using the `sqlda` database, write the SQL needed to show a **list of years** that emails were sent.

Your results should list years like this (order matters):

```
year
2011
2013
2014
2015
2016
2017
2018
2019
```

### SQL

```sql
SELECT DISTINCT
  EXTRACT(YEAR FROM sent_at)::int AS year
FROM emails
WHERE sent_at IS NOT NULL
ORDER BY year;
```

### Screenshot

![Q1 Screenshot](screenshots/q1_email_years.png)

---

## Question 2

Using the `sqlda` database, write the SQL needed to show the **number of messages sent by year**, ordered by year (as shown in the prompt).

Output should resemble:

```
count   year
...
```

### SQL

```sql
SELECT
  COUNT(*) AS count,
  EXTRACT(YEAR FROM sent_date)::int AS year
FROM emails
WHERE sent_date IS NOT NULL
GROUP BY EXTRACT(YEAR FROM sent_date)
ORDER BY year;
```

### Screenshot

![Q2 Screenshot](screenshots/q2_message_count_by_year.png)

---

## Question 3

Using the `sqlda` database, write the SQL needed to show:
- the **sent date**
- the **opened date**
- the **interval** between the two

Only include emails that contain **both** a sent date and an opened date.

### SQL

```sql
SELECT
  sent_date,
  opened_date,
  opened_date - sent_date AS sent_to_open_interval
FROM emails
WHERE sent_date IS NOT NULL
  AND opened_date IS NOT NULL
ORDER BY sent_date;
```

### Screenshot

![Q3 Screenshot](screenshots/q3_sent_opened_interval.png)

---

## Question 4

Using the `sqlda` database, write the SQL needed to show emails that contain an **opened date BEFORE the sent date**.

### SQL

```sql
SELECT
  email_id,
  sent_date,
  opened_date
FROM emails
WHERE sent_date IS NOT NULL
  AND opened_date IS NOT NULL
  AND opened_date < sent_date
ORDER BY sent_date, opened_date;
```

### Screenshot

![Q4 Screenshot](screenshots/q4_opened_before_sent.png)

---

## Question 5

Using the `sqlda` database: there are **over 100 emails** that contain an opened date **BEFORE** the sent date.

After looking at the data, **why is this the case?**

### Answer

It looks like the sent_date is not a real timestamp.  Instead it is a date with an arbritrary time that has been turned into a timestamp.  The open_date is an actual timestamp with an actual time of day.  So, this creates emails that look like they were opened before they were sent.

### Screenshot (if requested by instructor)

![Q5 Screenshot](screenshots/q5_explain_date_issue.png)

---

## Question 6

Using the `sqlda` database, explain in your own words what the following code does:

```sql
CREATE TEMP TABLE customer_points AS (
    SELECT
        customer_id,
        point(longitude, latitude) AS lng_lat_point
    FROM customers
    WHERE longitude IS NOT NULL
    AND latitude IS NOT NULL
);

CREATE TEMP TABLE dealership_points AS (
    SELECT
        dealership_id,
        point(longitude, latitude) AS lng_lat_point
    FROM dealerships
);

CREATE TEMP TABLE customer_dealership_distance AS (
    SELECT
       customer_id,
       dealership_id,
       c.lng_lat_point <@> d.lng_lat_point AS distance
    FROM customer_points c
    CROSS JOIN dealership_points d
);
```

### Answer

The code creates three tables.  One table is customer longitude and latitude.  The second table stores the dealership longitude and lattitude, the third table joins the first two tables and calculates the distance to dealerships for each customer. The output will be a table with rows with the customer and dealership listed with a distance between the two.

---

## Question 7

Using the `sqlda` database, write SQL to display an **array of salespeople for each dealership**, sorted by dealership.

For example - dealership 1 is below:

```text
"{""Fidell,Granville"",""Onele,Jereme"",""Sheriff,Lelia"",""McSpirron,Massimiliano"",""Rennick,Nadia"",""Mace,Eveleen"",""Oxteby,Dukie"",""Spong,Marcos"",""Wogden,Quent"",""Duny,Sandye"",""Loraine,Englebert"",""Meere,Ira"",""Gibbens,Cristine"",""Prine,Lyda"",""McCoughan,Sheff"",""Schule,Giselbert"",""McAndie,Eleen"",""Dosedale,Dorie"",""Nafziger,Shay""}"
```

### SQL

```sql
SELECT
  dealership_id,
  array_agg(
    last_name || ',' || first_name
    ORDER BY last_name, first_name
  ) AS salespeople
FROM salespeople
GROUP BY dealership_id
ORDER BY dealership_id;
```

### Screenshot

![Q7 Screenshot](screenshots/q7_salespeople_array_by_dealership.png)

---

## Question 8

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state** of the dealership
- the **number of salespeople** for the dealership

Sort by **state**.

Reference image:

![05-ExerciseArray](./instructions/05-ExerciseArray.jpg)

### SQL

```sql
SELECT
  d.dealership_id,
  d.state,
  array_agg(
    s.last_name || ',' || s.first_name
    ORDER BY s.last_name, s.first_name
  ) AS salespeople,
  COUNT(s.salesperson_id) AS salesperson_count
FROM dealerships d
JOIN salespeople s
  ON s.dealership_id = d.dealership_id
GROUP BY d.dealership_id, d.state
ORDER BY d.state, d.dealership_id;
```

### Screenshot

![Q8 Screenshot](screenshots/q8_salespeople_array_state_count.png)

---

## Question 9

Using the `sqlda` database, write the SQL needed to convert the **customers** table to **JSON**.

### SQL

```sql
SELECT
  row_to_json(c) AS customer_json
FROM customers c;
```

### Screenshot

![Q9 Screenshot](screenshots/q9_customers_to_json_v2.png)

---

## Question 10

Using the `sqlda` database, write SQL to display:
- an **array of salespeople for each dealership**
- the **state**
- the **number of salespeople**
- sorted by **state**

Then **convert this result to JSON**.

Reference image:

![05-ExerciseArray-1](./instructions/05-ExerciseArray-1.jpg)

### SQL

```sql
SELECT
  row_to_json(dealership_salespeople)
FROM (
  SELECT
    d.dealership_id,
    d.state,
    COUNT(s.salesperson_id) AS num_salespeople,
    array_agg(
      s.last_name || ',' || s.first_name
      ORDER BY s.last_name, s.first_name
    ) AS array_agg
  FROM dealerships d
  JOIN salespeople s
    ON s.dealership_id = d.dealership_id
  GROUP BY d.dealership_id, d.state
  ORDER BY d.state, d.dealership_id
) AS dealership_salespeople;
```

### Screenshot

![Q10 Screenshot](screenshots/q10_salespeople_array_to_jsonv2.png)

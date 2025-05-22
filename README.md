# SQL Practice
Practice writing SQL queries using Google BigQuery and publicly available datasets.

For these exercises, I used the CFPB complaints table uploaded to BigQuery as a public dataset. 
The table schema is below:

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/cfpb_table_schema.png "bigquery-public-data.cfpb_complaints.complaint_database")

## Beginner (1–5): SELECTs, WHERE, LIMIT, ORDER BY

<details>
<summary>1. List the first 10 complaints with their complaint_id, product, and date_received.</summary>

```sql
SELECT complaint_id, product, date_received
FROM bigquery-public-data.cfpb_complaints.complaint_database
LIMIT 10
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/first_10_complaints.png "First 10 complaints in the table")
</details>

<details>
<summary>2. Find all complaints related to the product "Credit card".</summary>

```sql
SELECT *
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE product = "Credit card"
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/credit_card_complaints.png "All complaints related to credit cards")
</details>

<details>
<summary>3. Count how many complaints were submitted via the "Web".</summary>

```sql
SELECT COUNT(*) AS submitted_via_web
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE submitted_via = "Web"
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/count_web_complaints.png "Total number of complaints submitted online")
</details>

<details>
<summary>4. Find the top 5 states with the highest number of complaints.</summary>

```sql
SELECT state, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY state
ORDER BY number_of_complaints DESC LIMIT 5
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/top_five_states.png "These states had the highest number of complaints")
</details>

<details>
<summary>5. Retrieve complaints where timely_response is false.</summary>

```sql
SELECT *
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE timely_response = FALSE
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/timely_response_false.png "These complaints were not considered to be handled in a timely manner")
</details>

## Intermediate: GROUP BY, Aggregates, Filtering, Subqueries

<details>
<summary>6. Count the number of complaints received each month.</summary>

```sql
SELECT FORMAT_DATE('%Y-%m', date_received) AS month, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY month
ORDER BY month
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/monthly_complaints.png "Total number of complaints received for each month in the table")
</details>

<details>
<summary>7. Find the most common issue consumers report.</summary>

```sql
SELECT issue, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY issue
ORDER BY number_of_complaints DESC LIMIT 1
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/most_common_issue.png "The most common issue reported by consumers")
</details>

<details>
<summary>8. Show how many disputes (consumer_disputed = TRUE) each company has had.</summary>

```sql
SELECT company_name, COUNT(*) AS number_of_disputes
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_disputed = TRUE
GROUP BY company_name
ORDER BY number_of_disputes DESC
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/disputes_by_company.png "The total number of disputed claims for each company")
</details>

<details>
<summary>9. Find the average number of complaints submitted per state.</summary>

```sql
SELECT AVG(number_of_complaints) AS average_complaints_per_state
FROM (
  SELECT state, COUNT(*) AS number_of_complaints
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY state
)
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/average_complaints_per_state.png "The average number of complaints per state")
</details>

<details>
<summary>10. List all companies that received more than 500 complaints.</summary>

```sql
SELECT company_name, COUNT(*) as number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY company_name
HAVING number_of_complaints > 500
ORDER BY number_of_complaints DESC
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/companies_over_500.png "These companies all had over 500 complaints lodged against them")
</details>

<details>
<summary>11. Show the number of complaints for each combination of product and subproduct.</summary>

```sql
SELECT product, subproduct, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY product, subproduct
ORDER BY product, number_of_complaints DESC
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/product_subproduct_complaints.png "The number of complaints for each combination of product and subproduct")
</details>

<details>
<summary>12. Find the number of complaints that included a consumer_complaint_narrative.</summary>

```sql
SELECT COUNT(*) AS complaints_with_consumer_narrative
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_complaint_narrative IS NOT NULL
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/consumer_narrative.png "These complaints all include a description given by the consumer")
</details>

<details>
<summary>13. List the companies with at least 100 complaints where the company_response_to_consumer was "Closed with explanation".</summary>

```sql
SELECT company_name, COUNT(*) as number_of_closed_with_explanation
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE company_response_to_consumer = "Closed with explanation"
GROUP BY company_name
HAVING number_of_closed_with_explanation >= 100
ORDER BY number_of_closed_with_explanation DESC
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/companies_closed_with_explanation.png "These companies al have at least 100 complaints that are closed with an explanation")
</details>

<details>
<summary>14. Find the proportion of complaints with a timely response by product.</summary>

```sql
SELECT 
  A.product, 
  B.number_of_timely,
  A.total_number_of_complaints,
  ROUND(B.number_of_timely * 100 / A.total_number_of_complaints, 2) AS proportion_of_timely_responses
FROM (
  SELECT product, COUNT(*) AS total_number_of_complaints
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY product
) AS A
JOIN (
  SELECT product, COUNT(*) AS number_of_timely
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  WHERE timely_response = TRUE
  GROUP BY product
) AS B
ON A.product = B.product
ORDER BY proportion_of_timely_responses DESC
```
![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/proportion_of_timely_responses.png "The percentage of timely responses vs. total complaints for each product")
</details>

## Advanced (15-20): Window Functions, Date Math, CTEs, CASE, Complex Filters

<details>
<summary>15. Identify the earliest complaint received for each company.</summary>

```sql
SELECT company_name, MIN(date_received) AS earliest_complaint
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY company_name
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/earliest_complaint_by_company.png "The date of the first complaint received for each company")
</details>

<details>
<summary>16. For each state, calculate the average number of days between date_received and date_sent_to_company.</summary>

```sql
SELECT
  state, 
  ROUND(AVG(DATE_DIFF(date_sent_to_company, date_received, DAY)), 3) AS average_days_between_sent_received
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY state
ORDER BY average_days_between_sent_received DESC
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/average_days_between_sent_received.png "The average number of days between the receipt of the complaint and the date which the complaint was sent to the company, for each state")
</details>

<details>
<summary>17. Create a ranking of companies based on the number of disputes (consumer_disputed = TRUE) using a window function.</summary>

```sql
SELECT
  company_name,
  COUNT(*) AS total_disputes,
  RANK() OVER(ORDER BY COUNT(*) DESC) AS dispute_rank
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_disputed = TRUE
GROUP BY company_name
ORDER BY dispute_rank
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/dispute_ranking.png "Ranking of the coompanies with the most disputed complaints")
</details>

<details>
<summary>18. Using a CASE statement, categorize complaints into:</summary>
  - "Fast" if sent to company on the same day it was received
  - "Delayed" if sent after 1 or more days
  - "Unknown" if date_sent_to_company is null

```sql
SELECT
  complaint_id,
  CASE
    WHEN date_sent_to_company IS NULL THEN "Unknown"
    WHEN DATE_DIFF(date_sent_to_company, date_received, DAY) < 1 THEN "Fast"
    WHEN DATE_DIFF(date_sent_to_company, date_received, DAY) >= 1  THEN "Delayed"
    ELSE "Error"
    END AS complaint_category
FROM bigquery-public-data.cfpb_complaints.complaint_database
LIMIT 1000
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/complaint_category.png "Categorizing complaints as being handled Fast, Delayed, or Unknown")
</details>

<details>
<summary>19. Write a CTE to find the total complaints per company, and from that CTE, return companies with more than 1,000 total complaints and less than 10 disputed ones.</summary>

```sql
WITH company_complaints AS (
  SELECT 
    company_name, 
    COUNT(*) AS total_complaints,
    COUNT(
      CASE
        WHEN consumer_disputed = TRUE THEN 1
        END
    ) AS total_disputed
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY company_name
)

SELECT company_name, total_complaints, total_disputed
FROM company_complaints
WHERE total_complaints > 1000 AND total_disputed < 10
```
![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/CTE_total_complaints.png "Companies with more than 1,000 total complaints and less than 10 disputed ones")
</details>

<details>
<summary>20. Find the percentage of complaints per company that resulted in a timely response, and rank companies by this percentage in descending order.</summary>s

```sql
WITH most_timely_companies AS (
  SELECT 
    company_name,
    COUNT(*) AS total_number_of_complaints,
    COUNTIF(timely_response = TRUE) AS number_of_timely,
    ROUND(COUNTIF(timely_response = TRUE) * 100.0 / COUNT(*), 2) AS percentage_timely
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY company_name
)

SELECT
  company_name,
  total_number_of_complaints,
  number_of_timely,
  percentage_timely,
  RANK() OVER(ORDER BY percentage_timely DESC) AS timely_rank
FROM most_timely_companies
ORDER BY timely_rank
```

![alt text](https://github.com/AustinBroadbent/SQL_Practice/blob/main/images/most_timely_companies.png "Ranking the companies by the percentage of timely responses vs. total complaints")
</details>

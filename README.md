# SQL Practice
Practice writing SQL queries using Google BigQuery and publicly available datasets.

For these exercises, I used the CFPB complaints table uploaded to BigQuery as a public dataset. 
The table schema is below:

[Image - table_schema]

## Beginner (1–5): SELECTs, WHERE, LIMIT, ORDER BY

### 1. List the first 10 complaints with their complaint_id, product, and date_received.

```sql
SELECT complaint_id, product, date_received
FROM bigquery-public-data.cfpb_complaints.complaint_database
LIMIT 10
```

![Image - first_ten_complaints]

### 2. Find all complaints related to the product "Credit card".

```sql
SELECT *
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE product = "Credit card"
```

[Image - credit_card_complaints]

### 3. Count how many complaints were submitted via the "Web".

```sql
SELECT COUNT(*) AS submitted_via_web
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE submitted_via = "Web"
```

[Image - count_web_complaints]

### 4. Find the top 5 states with the highest number of complaints.

```sql
SELECT state, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY state
ORDER BY number_of_complaints DESC LIMIT 5
```

[Image - top_five_states]

### 5. Retrieve complaints where timely_response is false.

```sql
SELECT *
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE timely_response = FALSE
```

[Image - timely_response_false]

## Intermediate: GROUP BY, Aggregrates, Fil;tering, Subqueries

### 6. Count the number of complaints received each month.

```sql
SELECT FORMAT_DATE('%Y-%m', date_received) AS month, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY month
ORDER BY month
```

[Image - monthly_complaints]

### 7. Find the most common issue consumers report.

```sql
SELECT issue, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY issue
ORDER BY number_of_complaints DESC LIMIT 1
```

[Image - most_common_issue]

### 8. Show how many disputes (consumer_disputed = TRUE) each company has had.

```sql
SELECT company_name, COUNT(*) AS number_of_disputes
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_disputed = TRUE
GROUP BY company_name
ORDER BY number_of_disputes DESC
```

[Image - disputes_by_company]

### 9. Find the average number of complaints submitted per state.

```sql
SELECT AVG(number_of_complaints) AS average_complaints_per_state
FROM (
  SELECT state, COUNT(*) AS number_of_complaints
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY state
)
```

[Image - average_complaints_per_state]

### 10. List all companies that received more than 500 complaints.

```sql
SELECT *
FROM (
  SELECT company_name, COUNT(*) AS number_of_complaints
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY company_name
)
WHERE number_of_complaints > 500
ORDER BY number_of_complaints DESC
```

[Image - companies_over_500]

### 11. Show the number of complaints for each combination of product and subproduct.

```sql
SELECT product, subproduct, COUNT(*) AS number_of_complaints
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY product, subproduct
ORDER BY product, number_of_complaints DESC
```

[Image - product_subproduct_complaints]

### 12. Find the number of complaints that included a consumer_complaint_narrative.

```sql
SELECT COUNT(*) AS complaints_with_consumer_narrative
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_complaint_narrative IS NOT NULL
```

[Image - consumer_narrative]

### 13. List the companies with at least 100 complaints where the company_response_to_consumer was "Closed with explanation".

```sql
SELECT *
FROM (
  SELECT company_name, COUNT(*) AS number_of_closed_with_explanation
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  WHERE company_response_to_consumer = "Closed with explanation"
  GROUP BY company_name
)
WHERE number_of_closed_with_explanation >= 100
ORDER BY number_of_closed_with_explanation DESC
```

[Image - companies_closed_with_explanation]

### 14. Find the proportion of complaints with a timely response by product.

```sql
SELECT 
  A.product, 
  B.number_of_timely,
  A.total_number_of_complaints,
  ROUND(B.number_of_timely * 100 / A.total_number_of_complaints, 2) AS proportion_of_timely_responses
FROM 
  (SELECT product, COUNT(*) AS total_number_of_complaints
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  GROUP BY product) AS A
JOIN 
  (SELECT product, COUNT(*) AS number_of_timely
  FROM bigquery-public-data.cfpb_complaints.complaint_database
  WHERE timely_response = TRUE
  GROUP BY product) AS B
ON A.product = B.product
ORDER BY proportion_of_timely_responses DESC
```
[Image - proportion_of_timely_responses]

## Advanced (15-20): Window Functions, Date Math, CTEs, CASE, Complex Filters

### 15. Identify the earliest complaint received for each company.

```sql
SELECT company_name, MIN(date_received) AS earliest_complaint
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY company_name
```

[Image - earliest_complaint_by_company]

### 16. For each state, calculate the average number of days between date_received and date_sent_to_company.

```sql
SELECT state, 
  ROUND(AVG(DATE_DIFF(date_sent_to_company, date_received, DAY)), 3) AS average_days_between_sent_received
FROM bigquery-public-data.cfpb_complaints.complaint_database
GROUP BY state
ORDER BY average_days_between_sent_received DESC
```

[Image - average_days_between_sent_received]
  
### 17. Create a ranking of companies based on the number of disputes (consumer_disputed = TRUE) using a window function.

```sql
SELECT company_name, COUNT(*) AS total_disputes,
  RANK() OVER(ORDER BY COUNT(*) DESC) AS dispute_rank
FROM bigquery-public-data.cfpb_complaints.complaint_database
WHERE consumer_disputed = TRUE
GROUP BY company_name
ORDER BY dispute_rank
```

[Image - dispute_ranking]

### 18. Using a CASE statement, categorize complaints into:
  - "Fast" if sent to company on the same day it was received
  - "Delayed" if sent after 1 or more days
  - "Unknown" if date_sent_to_company is null

```sql
SELECT complaint_id,
  CASE
    WHEN date_sent_to_company IS NULL THEN "Unknown"
    WHEN DATE_DIFF(date_sent_to_company, date_received, DAY) < 1 THEN "Fast"
    WHEN DATE_DIFF(date_sent_to_company, date_received, DAY) >= 1  THEN "Delayed"
    ELSE "Error"
    END
    AS complaint_category
FROM bigquery-public-data.cfpb_complaints.complaint_database
LIMIT 1000
```

[Image - complaint_category]

### 19. Write a CTE to find the total complaints per company, and from that CTE, return companies with more than 1,000 total complaints and less than 10 disputed ones.

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
[Image - CTE_total_complaints]

### 20. Find the percentage of complaints per company that resulted in a timely response, and rank companies by this percentage in descending order.

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

SELECT company_name, total_number_of_complaints, number_of_timely, percentage_timely,
  RANK() OVER(ORDER BY percentage_timely DESC) AS timely_rank
FROM most_timely_companies
ORDER BY timely_rank
```

[Image - most_timely_companies]

![DATABRICKS](https://img.shields.io/badge/Databricks-FF3621?style=for-the-badge&logo=Databricks&logoColor=white)
![Excel](https://img.shields.io/badge/Microsoft_Excel-217346?style=for-the-badge&logo=microsoft-excel&logoColor=white)
![MICROSOFT AZURE](https://img.shields.io/badge/microsoft%20azure-0089D6?style=for-the-badge&logo=microsoft-azure&logoColor=white)
	


## Overview

What if you could predict which insurance customers are 3x more likely to file a claim before they even do?

UK insurers lose millions annually due to:

* Inaccurate risk pricing based on incomplete data analysis
* Reactive claim management instead of predictive insights
* Fragmented customer data across multiple systems
* Manual reporting processes that delay critical decisions

 SOLUTION:

I developed a comprehensive SQL analytics framework using a complete UK car insurance, randomly generated, dataset (1,200+ customers, 1,500+ policies, 1,000+ claims):

* Advanced SQL queries for customer segmentation & risk profiling
* Claims pattern analysis identifying high-risk indicators
* Automated reporting dashboards for real-time insights
* Predicting Claim Likelihood by Segment Based on Historical Data

This project demonstrates ability to:
* Reduce claim costs through better risk assessment
* Improve customer retention via targeted pricing strategies
* Accelerate decision-making with automated SQL reporting
* Transform raw data into actionable business intelligence


## Table of Contents

- <a href="#background-information"
  id="toc-background-information">Ask</a>
  - <a href="#business-task" id="toc-business-task">Business Task</a>
- <a href="#prepare" id="toc-data-preparation">Prepare</a>
- <a href="#process" id="toc-data-exploration">Process</a>
- <a href="#analyze" id="toc-data-visualization">Analyze</a>
- <a href="#share" id="toc-recommendations">Share</a>
- <a href="#act" id="toc-recommendations">Act</a>


## Business Task

Analyze customer behavior, policy performance, and claims patterns within the UK car insurance market to identify high-risk customers, optimize pricing strategies, and reduce overall claim costs through data-driven insights.


## Prepare

Dataset overview:

Randomly generated data using Juilius AI

- customers.csv - 1200 rows, 11 columns

  - Primary Key: customer_id

  - Contains: Customer demographics, contact info, driving history

- policies.csv - 1500 rows, 12 columns

  - Primary Key: policy_id

  - Foreign Key: customer_id (references customers.csv)

  - Contains: Policy details, vehicle info, coverage, premiums

- claims.csv - 1000 rows, 11 columns

  - Primary Key: claim_id

  - Foreign Key: policy_id (references policies.csv)

  - Foreign Key: customer_id (references customers.csv) 

  - Contains: Claim details, incident info, settlements


Relationships:

- One customer can have multiple policies (1:N)

- One policy can have multiple claims (1:N)

- Customers -> Policies -> Claims (hierarchical relationship)

## Process

- Removed any blanks.
- Removed duplicates.
- Ensured the data is in the right format.


## Analyze


Premiums and number of policies by age group.

```
SELECT 
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    
    SUM(p.annual_premium) AS total_premiums,
    ROUND(AVG(p.annual_premium),2) AS avg_premium_per_policy,
    COUNT(DISTINCT p.policy_id) AS num_policies,
    COUNT(DISTINCT c.customer_id) AS num_customers
    
FROM customers c
JOIN policies p ON c.customer_id = p.customer_id
GROUP BY 
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END
ORDER BY age_group;
```
<img width="1700" height="775" alt="Premiums and number of policies by age group " src="https://github.com/user-attachments/assets/90c676ad-380d-49b8-a9a8-204b71c372e4" />


Total customers and the total broke by age group.

```

SELECT 
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    COUNT(DISTINCT c.customer_id) AS num_customers
FROM customers c
GROUP BY 
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END
ORDER BY age_group;

```

<img width="1703" height="431" alt="Total customers and the total broke by age group" src="https://github.com/user-attachments/assets/002317a1-f532-4197-9853-c9e1356522ad" />

#### CLAIMS AND PROFIT SEGMENTATION ANALYSIS

Total Claims Amount By Age Group.
Calculating what is the amount claimed by the age group, that information would be useful to inform the pricing strategy.
```
SELECT 
    -- Demographics
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    c.occupation,
    
    -- Claims
    COUNT(cl.claim_id) AS claim_count,
    ROUND(SUM(cl.claim_amount), 2) AS total_claims_amount,
    ROUND(AVG(cl.claim_amount), 2) AS avg_claim_amount   
FROM customers c
JOIN claims cl 
    ON c.customer_id = cl.customer_id  
GROUP BY age_group, c.gender, c.marital_status, c.occupation
ORDER BY total_claims_amount DESC;
```

<img width="2748" height="1184" alt="Total Claims Amount By Age Group" src="https://github.com/user-attachments/assets/5bcc4bf5-7d01-4a2c-a678-fced6ce3554c" />

This code segments the customers by demographics policy and vehicle details as well as claims details.
Also I have taken into account the average credit score and average years licensed.  
The query pulls the following info:


Breaks down the customers by segments such us:


- Demographics
 - Age group
 - Gender
 - Marital status
 - Occupation

- Policy and vehicle details such as:
 - Coverage 
 - Vehicle make

Aggregates by segments such as 

- Number of customers
- Average credit score 
- Average licensed years
- Average credit score
- Average premiums

I also enriched that with the information from the claims data set.
This allows us to create the following:

Average Claims per Customer by Age and Occupation

```
SELECT 
    -- Demographics
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    c.occupation,
    
    -- Policy & Vehicle
    p.coverage_type,
    p.vehicle_make,
    
    -- Aggregates
    COUNT(DISTINCT c.customer_id) AS customer_count,
    ROUND(AVG(c.credit_score), 0) AS avg_credit_score,
    ROUND(AVG(c.years_licensed), 1) AS avg_years_licensed,
    ROUND(AVG(p.annual_premium), 2) AS avg_premium,
    
    -- Claims enrichment
    ROUND(COALESCE(AVG(cl.claims_count), 0), 2) AS avg_claims_per_customer
FROM customers c
JOIN policies p 
    ON c.customer_id = p.customer_id
LEFT JOIN (
    SELECT customer_id, COUNT(*) AS claims_count
    FROM claims
    GROUP BY customer_id
) cl ON c.customer_id = cl.customer_id
GROUP BY age_group, c.gender, c.marital_status, c.occupation, p.coverage_type, p.vehicle_make
ORDER BY age_group, c.gender, p.coverage_type, p.vehicle_make;
```

Average Claims per Customer by Age and Occupation

<img width="2748" height="1184" alt="Average Claims per Customer by Age and Occupation" src="https://github.com/user-attachments/assets/040a721d-e3e7-402c-bac1-c28a90220d33" />



#### PROFITABILITY
Overall Profitability
Looking at the overall profitability of the data it looks that the data set is highly unprofitable.
```
SELECT
    SUM(p.annual_premium) AS total_premiums,
    SUM(COALESCE(cl.claim_amount,0)) AS total_claims,
    (SUM(p.annual_premium) - SUM(COALESCE(cl.claim_amount,0))) AS profit,
    ROUND(SUM(COALESCE(cl.claim_amount,0)) * 1.0 / NULLIF(SUM(p.annual_premium),0), 2) AS loss_ratio
FROM policies p
LEFT JOIN claims cl ON p.policy_id = cl.policy_id
LEFT JOIN customers c ON p.customer_id = c.customer_id;

```

<img width="1693" height="818" alt="Overall Profitability" src="https://github.com/user-attachments/assets/59c329b6-2821-4726-bdd1-76c10da7871b" />




Profitability-Based Customer Profiles

Analysing further profiles ranked by profitability (e.g., average premium – claims paid), so we can determine which segments are most valuable and focus on unprofitable segments to eradicate loss of revenue.
The analysis assumes that if the claim status is closed then the settlement amount will be taken into consideration otherwise the claim amount will be used.


```
SELECT 
    -- Demographics
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    c.occupation,
    
    -- Policy & Vehicle
    p.coverage_type, 
    p.vehicle_make,
    
    -- Aggregates
    COUNT(DISTINCT c.customer_id) AS customer_count,
    ROUND(AVG(c.credit_score), 0) AS avg_credit_score,
    ROUND(AVG(c.years_licensed), 1) AS avg_years_licensed,
    
    ROUND(SUM(p.annual_premium), 2) AS total_premiums,
    ROUND(COALESCE(SUM(cl.total_claims), 0), 2) AS total_claims,
    
    
    
    CASE WHEN cl.claim_status = 'Closed' THEN ROUND(SUM(p.annual_premium) - COALESCE(SUM(cl.settlement_amount), 0), 2) 
         ELSE ROUND(SUM(p.annual_premium) - COALESCE(SUM(cl.total_claims), 0), 2) END AS profit,
    
    CASE
      WHEN cl.claim_status = 'Closed' THEN ROUND((SUM(p.annual_premium) - COALESCE(SUM(cl.settlement_amount), 0)) / 
          NULLIF(SUM(p.annual_premium), 0), 2) 
          ELSE ROUND((SUM(p.annual_premium) - COALESCE(SUM(cl.total_claims), 0)) / 
          NULLIF(SUM(p.annual_premium), 0), 2) END AS profit_margin


FROM customers c
JOIN policies p 
    ON c.customer_id = p.customer_id
LEFT JOIN (
    SELECT customer_id, SUM(claim_amount) AS total_claims,
    claim_status,
    SUM(settlement_amount) AS settlement_amount
    FROM claims
    GROUP BY customer_id, claim_status
) cl ON c.customer_id = cl.customer_id

GROUP BY age_group, c.gender, c.marital_status, c.occupation, p.coverage_type, p.vehicle_make, cl.claim_status
ORDER BY profit DESC;
```

Profit by age group

<img width="1366" height="944" alt="Profit by Age Group" src="https://github.com/user-attachments/assets/cae42cda-d68e-4dd2-ae15-9e214647ca82" />

Profit by Age and Marital Status
<img width="1366" height="944" alt="Profit by Age, Marital Status" src="https://github.com/user-attachments/assets/c7f86308-d9e9-4184-acff-5562741b3077" />

Profit by Gender

<img width="1366" height="704" alt="Profit by Gender" src="https://github.com/user-attachments/assets/63f28227-53a0-410b-8668-8d73321f4a1f" />


Profit by occupation


<img width="2748" height="944" alt="Profit by occupation" src="https://github.com/user-attachments/assets/b4eb5a53-1fea-4865-84c3-73f10425aee8" />


##  RISK 

Average Claims by Age and Occupation Code

```
SELECT 
    -- Demographics
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    c.occupation,
    
    -- Policy & Vehicle
    p.coverage_type,
    p.vehicle_make,
    
    -- Aggregates
    COUNT(DISTINCT c.customer_id) AS customer_count,
    ROUND(AVG(c.credit_score), 0) AS avg_credit_score,
    ROUND(AVG(c.years_licensed), 1) AS avg_years_licensed,
    ROUND(AVG(p.annual_premium), 2) AS avg_premium,
    
    -- Claims enrichment
    ROUND(COALESCE(AVG(cl.claims_count), 0), 2) AS avg_claims_per_customer
FROM customers c
JOIN policies p 
    ON c.customer_id = p.customer_id
LEFT JOIN (
    SELECT customer_id, COUNT(*) AS claims_count
    FROM claims
    GROUP BY customer_id
) cl ON c.customer_id = cl.customer_id
GROUP BY age_group, c.gender, c.marital_status, c.occupation, p.coverage_type, p.vehicle_make
ORDER BY age_group, c.gender, p.coverage_type, p.vehicle_make;
```
Average Claims by Age and Occupation Visuals

<img width="2748" height="1184" alt="Average Claims per Customer by Age and Occupation" src="https://github.com/user-attachments/assets/8257bb1b-ee8c-4e5a-aabd-de4fa0a1430c" />

## Segment Risk Profiling and prediction of the likelihood of a claim based on previous behaviour.

Segment Risk Profiling 

```
SELECT 
    -- Demographic segmentation
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    c.occupation,
    
    -- Policy & Vehicle segmentation
    p.coverage_type,
    p.vehicle_make,
    
    -- Aggregated risk metrics
    COUNT(DISTINCT c.customer_id) AS customer_count,
    COUNT(cl.claim_id) AS total_claims,
    COALESCE(SUM(cl.claim_amount), 0) AS total_claim_amount,
    SUM(p.annual_premium) AS total_premiums,
    
    -- Derived risk ratios
    ROUND(COALESCE(SUM(cl.claim_amount), 0) / NULLIF(SUM(p.annual_premium), 0), 2) AS loss_ratio,
    ROUND(AVG(c.credit_score), 0) AS avg_credit_score,
    ROUND(AVG(c.years_licensed), 1) AS avg_years_licensed,
    
    -- Segment-level Risk Category
    CASE 
        WHEN COUNT(cl.claim_id) > 100 
             OR (COALESCE(SUM(cl.claim_amount), 0) / NULLIF(SUM(p.annual_premium), 0)) > 1.0 
             OR AVG(c.credit_score) < 500 
        THEN 'High Risk'
        WHEN COUNT(cl.claim_id) BETWEEN 30 AND 100
             OR (COALESCE(SUM(cl.claim_amount), 0) / NULLIF(SUM(p.annual_premium), 0)) BETWEEN 0.5 AND 1.0
        THEN 'Medium Risk'
        ELSE 'Low Risk'
    END AS risk_profile

FROM customers c
JOIN policies p 
    ON c.customer_id = p.customer_id
LEFT JOIN claims cl 
    ON c.customer_id = cl.customer_id
   AND p.policy_id = cl.policy_id

GROUP BY age_group, c.gender, c.marital_status, c.occupation, p.coverage_type, p.vehicle_make
ORDER BY risk_profile DESC, total_claim_amount DESC;

```

Segment Risk Profiling Visuals


<img width="1903" height="717" alt="Segment Risk Profiling " src="https://github.com/user-attachments/assets/5ccd7a08-923f-4943-adbc-b66eea01e35f" />



Claim Likelihood by Segment Based on Historical Data

```
SELECT 
    CASE 
        WHEN c.age < 25 THEN 'Under 25'
        WHEN c.age BETWEEN 25 AND 39 THEN '25-39'
        WHEN c.age BETWEEN 40 AND 59 THEN '40-59'
        ELSE '60+'
    END AS age_group,
    c.gender,
    c.marital_status,
    p.coverage_type,
    p.vehicle_make,
    
    COUNT(DISTINCT c.customer_id) AS total_customers,
    COUNT(DISTINCT CASE WHEN cl.claim_id IS NOT NULL THEN c.customer_id END) AS customers_with_claims,
    
    -- Historical claim likelihood (empirical probability)
    ROUND(
        COUNT(DISTINCT CASE WHEN cl.claim_id IS NOT NULL THEN c.customer_id END) 
        * 1.0 / NULLIF(COUNT(DISTINCT c.customer_id), 0), 
        2
    ) AS claim_likelihood,
    
    ROUND(AVG(cl.claim_amount), 2) AS avg_claim_amount

FROM customers c
JOIN policies p ON c.customer_id = p.customer_id
LEFT JOIN claims cl ON c.customer_id = cl.customer_id 

GROUP BY age_group, c.gender, c.marital_status, p.coverage_type, p.vehicle_make
ORDER BY claim_likelihood DESC;

```


![326F5BED-58FE-47F5-AA60-7229B5A76C69](https://github.com/user-attachments/assets/883d71f1-da6f-4b32-985a-6924daa342b0)


## Share

- <a href="https://gamma.app/docs/Advanced-SQL-Analytics-for-UK-Insurance-Predictive-Risk-Profiling-jgn40euxsn2z4os" id="toc-data-preparation">Presentation</a>



## Act

Implement segment-specific pricing strategies, develop targeted risk mitigation programmes for high-risk segments, and establish continuous monitoring systems to maintain predictive accuracy.


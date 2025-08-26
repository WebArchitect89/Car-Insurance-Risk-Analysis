# Car-Insurance-Analysis
A a comprehensive SQL analytics framework using a complete UK car insurance dataset (1,200+ customers, 1,500+ policies, 1,000+ claims)

Customer segmentation by demographics, policy and vehicle details and claims information. 

## SEGMENTATION
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
Highest and Lowest Claim Segment Code
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
Total Claims Amount By Age Group

<img width="2748" height="1184" alt="Total Claims Amount By Age Group" src="https://github.com/user-attachments/assets/5bcc4bf5-7d01-4a2c-a678-fced6ce3554c" />




## PROFITABILITY

Profitability-Based Customer Profiles Code


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

Profitability-Based Customer Profiles Visuals

Profit by age group

<img width="1366" height="944" alt="Profit by Age Group" src="https://github.com/user-attachments/assets/cae42cda-d68e-4dd2-ae15-9e214647ca82" />

Profit by Age and Marital Status
<img width="1366" height="944" alt="Profit by Age, Marital Status" src="https://github.com/user-attachments/assets/c7f86308-d9e9-4184-acff-5562741b3077" />

Profit by Gender

<img width="2748" height="704" alt="Profit by Gender" src="https://github.com/user-attachments/assets/d65b667a-a525-47ca-90c9-471a08148f35" />

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

## Segment Risk Profiling 

Segment Risk Profiling Code

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




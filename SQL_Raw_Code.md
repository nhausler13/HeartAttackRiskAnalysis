CREATE DATABASE heart_attack_analysis;
USE heart_attack_analysis;
DROP TABLE heart_attack_data;
CREATE TABLE heart_attack_data (
    patient_id INT PRIMARY KEY,
    age INT,
    anaemia TINYINT(1),
    creatinine_phosphokinase FLOAT,
    diabetes TINYINT(1),
    ejection_fraction FLOAT,
    high_bp TINYINT(1),
    platelets FLOAT,
    serum_creatinine FLOAT,
    serum_sodium FLOAT,
    sex TINYINT(1),
    smoking TINYINT(1),
    death TINYINT(1),
    log_cpk FLOAT,
    log_creatinine FLOAT,
    log_platelets FLOAT,
    risk_factor_count INT,
    age_category VARCHAR(10),
    creatine_to_ef_ratio FLOAT,
    platelets_per_risk_factor FLOAT,
    high_creatinine TINYINT(1),
    low_ef TINYINT(1),
    high_serum_sodium TINYINT(1)
);

# Give a summery of numeric features for survivors (death = 0) vs non-survivors (death = 1)

SELECT 
death,
AVG(age) AS avg_age,
AVG(ejection_fraction) AS avg_ef,
AVG(serum_creatinine) AS avg_creatinine,
AVG(platelets) AS avg_platelets,
AVG(creatinine_phosphokinase) AS avg_cpk,
MIN(age) AS min_age,
MAX(age) AS max_age
FROM heart_attack_data
GROUP BY death;

# Compute the mortality rate per category

#Mortality rate for anaemia
SELECT 
anaemia, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY anaemia;

#Mortality rate for diabetes
SELECT 
diabetes, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY diabetes;

#Mortality rate for smoking
SELECT 
smoking, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY smoking;

#Mortality rate for high_creatinine
SELECT 
high_creatinine, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY high_creatinine;

#Mortality rate for low_ef
SELECT 
low_ef, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY low_ef;

SELECT 
high_serum_sodium, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY high_serum_sodium;

SELECT 
high_bp, 
AVG(death) AS mortality_rate,
COUNT(*) as num_patients
FROM heart_attack_data
GROUP BY high_bp;
# Comorbidities
SELECT 
    anaemia,
    diabetes,
    high_bp,
    smoking,
    AVG(death) AS mortality_rate,
    COUNT(*) AS num_patients
FROM heart_attack_data
GROUP BY anaemia, diabetes, high_bp,smoking;

# High-risk lab/value flags
SELECT 
	high_creatinine,
    low_ef,
    high_serum_sodium,
    AVG(death) AS mortality_rate,
    COUNT(*) AS num_patients
FROM heart_attack_data
GROUP BY high_creatinine, low_ef,high_serum_sodium;

# Age + risk factors
SELECT
	age_category,
    risk_factor_count,
    AVG(death) as mortality_rate,
    COUNT(*) AS num_patients
FROM heart_attack_data
GROUP BY age_category, risk_factor_count ORDER BY age_category ASC;

# COMBORDITIES BY SEX 
# Comorbidities
SELECT
    sex,
    COUNT(*) AS num_patients,
    AVG(death) AS mortality_rate,
    AVG(anaemia) AS avg_anaemia,
    AVG(diabetes) AS avg_diabetes,
    AVG(high_bp) AS avg_high_bp,
    AVG(smoking) AS avg_smoking,
    AVG(risk_factor_count) AS avg_risk_factors
FROM heart_attack_data
GROUP BY sex;


-- Summary Table: Comorbidities Impact on Mortality
SELECT
    anaemia,
    diabetes,
    high_bp,
    smoking,
    
    -- Mortality by risk factor count
    risk_factor_count,
    COUNT(*) AS num_patients,
    AVG(death) AS mortality_rate
    

FROM heart_attack_data
GROUP BY risk_factor_count, anaemia, diabetes, high_bp, smoking
ORDER BY risk_factor_count DESC, mortality_rate DESC;

# Comparing averages for death
SELECT
    death,
    AVG(age) AS avg_age,
    AVG(ejection_fraction) AS avg_ef,
    AVG(serum_creatinine) AS avg_creatinine,
    AVG(serum_sodium) AS avg_sodium,
    AVG(platelets) AS avg_platelets,
    AVG(creatinine_phosphokinase) AS avg_cpk
FROM heart_attack_data
GROUP BY death;

#creating correlation flags, higher flags, higher correlation rate 
SELECT 'anaemia' AS feature, anaemia AS value, AVG(death) AS mortality_rate, COUNT(*) AS n
FROM heart_attack_data
GROUP BY anaemia
UNION ALL
SELECT 'diabetes', diabetes, AVG(death), COUNT(*)
FROM heart_attack_data
GROUP BY diabetes
UNION ALL
SELECT 'high_bp', high_bp, AVG(death), COUNT(*)
FROM heart_attack_data
GROUP BY high_bp
UNION ALL
SELECT 'smoking', smoking, AVG(death), COUNT(*)
FROM heart_attack_data
GROUP BY smoking
UNION ALL
SELECT 'sex', sex, AVG(death), COUNT(*)
FROM heart_attack_data
GROUP BY sex;


# FEATURE RANKING  - Binary Features
-- Consolidated Feature Ranking Table
SELECT
    feature,
    feature_type,
    AVG(CASE WHEN death=0 THEN value END) AS avg_survived,
    AVG(CASE WHEN death=1 THEN value END) AS avg_died,
    AVG(CASE WHEN death=1 THEN value END) - AVG(CASE WHEN death=0 THEN value END) AS diff,
    STDDEV(value) AS stddev,
    COUNT(*) AS patient_count
FROM (
    -- Numeric & Engineered Features
    SELECT 'age' AS feature, 'numeric' AS feature_type, age AS value, death FROM heart_attack_data
    UNION ALL
    SELECT 'ejection_fraction', 'numeric', ejection_fraction, death FROM heart_attack_data
    UNION ALL
    SELECT 'platelets', 'numeric', platelets, death FROM heart_attack_data
    UNION ALL
    SELECT 'serum_creatinine', 'numeric', serum_creatinine, death FROM heart_attack_data
    UNION ALL
    SELECT 'log_CPK', 'engineered', log_CPK, death FROM heart_attack_data
    UNION ALL
    SELECT 'log_creatinine', 'engineered', log_creatinine, death FROM heart_attack_data
    UNION ALL
    SELECT 'log_platelets', 'engineered', log_platelets, death FROM heart_attack_data
    UNION ALL
    SELECT 'creatine_to_ef_ratio', 'engineered', creatine_to_ef_ratio, death FROM heart_attack_data
    UNION ALL
    SELECT 'platelets_per_risk_factor', 'engineered', platelets_per_risk_factor, death FROM heart_attack_data
    UNION ALL
    SELECT 'risk_factor_count', 'engineered', risk_factor_count, death FROM heart_attack_data

    -- Binary / Categorical Features (0/1 flags)
    UNION ALL
    SELECT 'anaemia', 'categorical', anaemia, death FROM heart_attack_data
    UNION ALL
    SELECT 'diabetes', 'categorical', diabetes, death FROM heart_attack_data
    UNION ALL
    SELECT 'high_bp', 'categorical', high_bp, death FROM heart_attack_data
    UNION ALL
    SELECT 'smoking', 'categorical', smoking, death FROM heart_attack_data
    UNION ALL
    SELECT 'sex', 'categorical', sex, death FROM heart_attack_data
) AS combined
GROUP BY feature, feature_type
ORDER BY diff DESC;

# Creating a risk scoring chart: 
-- Step 2: Create risk category table

SELECT
    risk_category,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM (
    SELECT *,
        CASE
            WHEN risk_factor_count <= 1 
                 AND high_creatinine = 0 AND low_ef = 0 AND high_serum_sodium = 0
            THEN 'Low'
            WHEN risk_factor_count = 2
                 OR (high_creatinine + low_ef + high_serum_sodium) = 1
            THEN 'Medium'
            ELSE 'High'
        END AS risk_category
    FROM heart_attack_data
) AS risk_table
GROUP BY risk_category
ORDER BY FIELD(risk_category,'Low','Medium','High');

# Leveraging derived features ( creatine_to_ef_ratio, platelets_per_risk_factor) to see if they highlight high-risk patients better then single features: 
SELECT 
    AVG(creatine_to_ef_ratio) AS avg_creatine_to_ef,
    MIN(creatine_to_ef_ratio) AS min_creatine_to_ef,
    MAX(creatine_to_ef_ratio) AS max_creatine_to_ef,
    STDDEV(creatine_to_ef_ratio) AS std_creatine_to_ef,
    AVG(platelets_per_risk_factor) AS avg_platelets_per_risk,
    MIN(platelets_per_risk_factor) AS min_platelets_per_risk,
    MAX(platelets_per_risk_factor) AS max_platelets_per_risk,
    STDDEV(platelets_per_risk_factor) AS std_platelets_per_risk,
    death
FROM heart_attack_data
GROUP BY death;

# Creating bins for creatine_to_ef_ratio to see mortality rate
SELECT 
    CASE
        WHEN creatine_to_ef_ratio <= 0.02 THEN 'Q1: Lowest ratio'
        WHEN creatine_to_ef_ratio <= 0.04 THEN 'Q2: Low-Medium ratio'
        WHEN creatine_to_ef_ratio <= 0.06 THEN 'Q3: Medium-High ratio'
        ELSE 'Q4: Highest ratio'
    END AS ratio_quartile,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
GROUP BY ratio_quartile
ORDER BY ratio_quartile;

SELECT
    creatine_to_ef_quartile,
    platelets_per_risk_factor_quartile,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM (
    SELECT *,
        NTILE(4) OVER (ORDER BY creatine_to_ef_ratio) AS creatine_to_ef_quartile,
        NTILE(4) OVER (ORDER BY platelets_per_risk_factor) AS platelets_per_risk_factor_quartile
    FROM heart_attack_data
) AS derived_quartiles
GROUP BY creatine_to_ef_quartile, platelets_per_risk_factor_quartile
ORDER BY creatine_to_ef_quartile, platelets_per_risk_factor_quartile;
# Creating bins for platelets_per_risk to see mortality rate
SELECT 
    CASE
        WHEN platelets_per_risk_factor <= 80000 THEN 'Q1: Lowest'
        WHEN platelets_per_risk_factor <= 150000 THEN 'Q2: Low-Medium'
        WHEN platelets_per_risk_factor <= 220000 THEN 'Q3: Medium-High'
        ELSE 'Q4: Highest'
    END AS platelets_ratio_quartile,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
GROUP BY platelets_ratio_quartile
ORDER BY platelets_ratio_quartile;

# Show which extremem combinatins have the highest moratlity and which ar the lower risk 

-- Step 1: Compute quartiles and risk score per patient
WITH ranked AS (
    SELECT
        patient_id,
        death,
        creatine_to_ef_ratio,
        platelets_per_risk_factor,
        NTILE(4) OVER (ORDER BY creatine_to_ef_ratio) AS cpk_ef_quartile,
        NTILE(4) OVER (ORDER BY platelets_per_risk_factor) AS platelets_quartile
    FROM heart_attack_data
),
quartile_labels AS (
    SELECT
        patient_id,
        death,
        creatine_to_ef_ratio,
        platelets_per_risk_factor,
        cpk_ef_quartile,
        platelets_quartile,
        CASE cpk_ef_quartile
            WHEN 1 THEN 'Q1: Lowest CPK/EF'
            WHEN 2 THEN 'Q2: Low-Medium CPK/EF'
            WHEN 3 THEN 'Q3: Medium-High CPK/EF'
            ELSE 'Q4: Highest CPK/EF'
        END AS creatine_ef_label,
        CASE platelets_quartile
            WHEN 1 THEN 'Q1: Lowest platelets per risk factor'
            WHEN 2 THEN 'Q2: Low-Medium platelets per risk factor'
            WHEN 3 THEN 'Q3: Medium-High platelets per risk factor'
            ELSE 'Q4: Highest platelets per risk factor'
        END AS platelets_label
    FROM ranked
),
risk_score_calc AS (
    SELECT
        patient_id,
        death,
        creatine_ef_label,
        platelets_label,
        -- Use the quartile numbers from ranked CTE
       (cpk_ef_quartile - 1) + (4 - platelets_quartile) AS total_risk_score
    FROM quartile_labels
)
SELECT
    creatine_ef_label,
    platelets_label,
    total_risk_score,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM risk_score_calc
GROUP BY creatine_ef_label, platelets_label, total_risk_score
ORDER BY total_risk_score DESC, mortality_rate DESC;

#SUMMARIZING KEY INSIGHTS FROM PREVIOUS ANALYSIS 

# Average ejection fraction by age group: 

SELECT
    age_category,
    AVG(ejection_fraction) AS avg_ejection_fraction,
    COUNT(*) AS patient_count
FROM heart_attack_data
GROUP BY age_category
ORDER BY age_category;

# Death rate by comorbity count ( risk_factor_count) 
SELECT
    risk_factor_count,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
GROUP BY risk_factor_count
ORDER BY risk_factor_count;

# Mortality by quartiles of dervied ratios: 
WITH patient_risk AS (
    SELECT
        patient_id,
        death,
        CASE
            WHEN creatine_to_ef_ratio <= 0.02 THEN 'Lowest CPK/EF ratio'
            WHEN creatine_to_ef_ratio <= 0.04 THEN 'Low-Medium CPK/EF ratio'
            WHEN creatine_to_ef_ratio <= 0.06 THEN 'Medium-High CPK/EF ratio'
            ELSE 'Highest CPK/EF ratio'
        END AS creatine_ef_quartile,
        CASE
            WHEN platelets_per_risk_factor <= 80000 THEN 'Lowest platelets per risk factor'
            WHEN platelets_per_risk_factor <= 150000 THEN 'Low-Medium platelets per risk factor'
            WHEN platelets_per_risk_factor <= 220000 THEN 'Medium-High platelets per risk factor'
            ELSE 'Highest platelets per risk factor'
        END AS platelets_quartile
    FROM heart_attack_data
)
SELECT
    creatine_ef_quartile,
    platelets_quartile,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM patient_risk
GROUP BY creatine_ef_quartile, platelets_quartile
ORDER BY mortality_rate DESC;

# Comorbidity + Age category cross-tab , only looking at patient_ count > 10 to reduce noise and outliers: 
SELECT
    age_category,
    risk_factor_count,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
GROUP BY age_category, risk_factor_count
HAVING COUNT(*) >= 10   -- only include groups with 10 or more patients
ORDER BY age_category, risk_factor_count;

# Feature Ranking
SELECT feature, impact
FROM (
SELECT
    'age' AS feature,
    AVG(CASE WHEN death = 1 THEN age END) - AVG(CASE WHEN death = 0 THEN age END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'ejection_fraction' AS feature,
    AVG(CASE WHEN death = 1 THEN ejection_fraction END) - AVG(CASE WHEN death = 0 THEN ejection_fraction END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'creatinine_phosphokinase' AS feature,
    AVG(CASE WHEN death = 1 THEN log_cpk END) - AVG(CASE WHEN death = 0 THEN log_cpk END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'serum_creatinine' AS feature,
    AVG(CASE WHEN death = 1 THEN log_creatinine END) - AVG(CASE WHEN death = 0 THEN log_creatinine END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'serum_sodium' AS feature,
    AVG(CASE WHEN death = 1 THEN serum_sodium END) - AVG(CASE WHEN death = 0 THEN serum_sodium END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'platelets' AS feature,
    AVG(CASE WHEN death = 1 THEN log_platelets END) - AVG(CASE WHEN death = 0 THEN platelets END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'creatine_to_ef_ratio' AS feature,
    AVG(CASE WHEN death = 1 THEN creatine_to_ef_ratio END) - AVG(CASE WHEN death = 0 THEN creatine_to_ef_ratio END) AS impact
FROM heart_attack_data
UNION ALL
SELECT
    'platelets_per_risk_factor' AS feature,
    AVG(CASE WHEN death = 1 THEN platelets_per_risk_factor END) - AVG(CASE WHEN death = 0 THEN platelets_per_risk_factor END) AS impact
FROM heart_attack_data

UNION ALL 
SELECT 'anaemia' AS feature, 
       AVG(CASE WHEN anaemia = 1 THEN death END) - AVG(CASE WHEN anaemia = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'diabetes' AS feature, 
       AVG(CASE WHEN diabetes = 1 THEN death END) - AVG(CASE WHEN diabetes = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'high_bp' AS feature, 
       AVG(CASE WHEN high_bp = 1 THEN death END) - AVG(CASE WHEN high_bp = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'smoking' AS feature, 
       AVG(CASE WHEN smoking = 1 THEN death END) - AVG(CASE WHEN smoking = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'high_creatinine' AS feature, 
       AVG(CASE WHEN high_creatinine = 1 THEN death END) - AVG(CASE WHEN high_creatinine = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'low_ef' AS feature, 
       AVG(CASE WHEN low_ef = 1 THEN death END) - AVG(CASE WHEN low_ef = 0 THEN death END) AS impact
FROM heart_attack_data
UNION ALL
SELECT 'high_serum_sodium' AS feature, 
       AVG(CASE WHEN high_serum_sodium = 1 THEN death END) - AVG(CASE WHEN high_serum_sodium = 0 THEN death END) AS impact
FROM heart_attack_data
) AS combined_features
ORDER BY ABS(impact) DESC;


-- Mortality by individual comorbidities, only for patients with 1-3 risk factors
-- Combined mortality by risk_factor_count and comorbidity
SELECT
    risk_factor_count,
    'anaemia' AS comorbidity,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
WHERE risk_factor_count BETWEEN 1 AND 3 AND anaemia = 1
GROUP BY risk_factor_count
UNION ALL
SELECT
    risk_factor_count,
    'diabetes' AS comorbidity,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
WHERE risk_factor_count BETWEEN 1 AND 3 AND diabetes = 1
GROUP BY risk_factor_count
UNION ALL
SELECT
    risk_factor_count,
    'high_bp' AS comorbidity,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
WHERE risk_factor_count BETWEEN 1 AND 3 AND high_bp = 1
GROUP BY risk_factor_count
UNION ALL
SELECT
    risk_factor_count,
    'smoking' AS comorbidity,
    COUNT(*) AS patient_count,
    AVG(death) AS mortality_rate
FROM heart_attack_data
WHERE risk_factor_count BETWEEN 1 AND 3 AND smoking = 1
GROUP BY risk_factor_count
ORDER BY risk_factor_count, comorbidity;


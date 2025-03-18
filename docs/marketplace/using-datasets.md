# Using Datasets from the Snowflake Marketplace

After acquiring datasets from the Snowflake Marketplace, the next step is to effectively use them in your analytics workflows. This section provides practical guidance on how to explore, integrate, and derive value from marketplace data.

## Exploring Acquired Datasets

### Understanding Dataset Structure

When you first access a new dataset:

1. **Examine Database Objects**: Understand tables, views, and their relationships
   ```sql
   -- List all tables in the acquired database
   SHOW TABLES IN DATABASE marketplace_database;
   
   -- List all views
   SHOW VIEWS IN DATABASE marketplace_database;
   
   -- Examine schemas
   SHOW SCHEMAS IN DATABASE marketplace_database;
   ```

2. **Review Table Schemas**: Understand column definitions and data types
   ```sql
   -- Describe a specific table
   DESCRIBE TABLE marketplace_database.schema_name.table_name;
   
   -- List columns and their data types
   SELECT column_name, data_type 
   FROM marketplace_database.information_schema.columns
   WHERE table_name = 'TABLE_NAME'
   ORDER BY ordinal_position;
   ```

3. **Sample Data**: View sample records to understand the content
   ```sql
   -- View a sample of the data
   SELECT * FROM marketplace_database.schema_name.table_name
   LIMIT 100;
   
   -- Check for null values and data quality
   SELECT 
     COUNT(*) as total_rows,
     COUNT(column1) as column1_not_null,
     COUNT(column2) as column2_not_null
   FROM marketplace_database.schema_name.table_name;
   ```

### Dataset Documentation

Most marketplace datasets include documentation:

- **Data Dictionary**: Definitions of tables and columns
- **Update Frequency**: How often the data refreshes
- **Sample Queries**: Pre-built queries to get started
- **Known Limitations**: Any caveats or constraints

Look for these resources in:

- The dataset listing page in the marketplace
- README or documentation tables in the shared database
- Provider-supplied links to external documentation

## Query Patterns for Marketplace Data

### Basic Query Patterns

Start with these fundamental query patterns:

#### Exploration Queries

```sql
-- Count records by category
SELECT category, COUNT(*) as record_count
FROM marketplace_database.schema_name.table_name
GROUP BY category
ORDER BY record_count DESC;

-- Get min, max, avg for numeric columns
SELECT 
  MIN(numeric_column) as min_value,
  MAX(numeric_column) as max_value,
  AVG(numeric_column) as avg_value
FROM marketplace_database.schema_name.table_name;

-- Check date ranges
SELECT 
  MIN(date_column) as earliest_date,
  MAX(date_column) as latest_date
FROM marketplace_database.schema_name.table_name;
```

#### Filtering and Segmentation

```sql
-- Filter by specific attributes
SELECT *
FROM marketplace_database.schema_name.table_name
WHERE attribute = 'value'
  AND numeric_value > 100
  AND date_column BETWEEN '2023-01-01' AND '2023-12-31';

-- Segmentation analysis
SELECT 
  segment_column,
  COUNT(*) as count,
  AVG(metric_column) as avg_metric
FROM marketplace_database.schema_name.table_name
GROUP BY segment_column
ORDER BY avg_metric DESC;
```

### Advanced Query Patterns

For deeper analysis, try these advanced techniques:

#### Time Series Analysis

```sql
-- Trend analysis by month
SELECT 
  DATE_TRUNC('month', date_column) as month,
  COUNT(*) as record_count,
  SUM(value_column) as total_value,
  AVG(value_column) as avg_value
FROM marketplace_database.schema_name.table_name
GROUP BY month
ORDER BY month;

-- Year-over-year comparison
WITH current_year AS (
  SELECT month, SUM(value) as value
  FROM marketplace_database.schema_name.table_name
  WHERE YEAR(date_column) = 2023
  GROUP BY month
),
previous_year AS (
  SELECT month, SUM(value) as value
  FROM marketplace_database.schema_name.table_name
  WHERE YEAR(date_column) = 2022
  GROUP BY month
)
SELECT 
  c.month,
  c.value as current_value,
  p.value as previous_value,
  (c.value - p.value) / p.value as yoy_growth
FROM current_year c
JOIN previous_year p ON c.month = p.month
ORDER BY c.month;
```

#### Geospatial Analysis

Many marketplace datasets include location data:

```sql
-- Regional aggregation
SELECT 
  region,
  COUNT(*) as record_count,
  SUM(value_column) as total_value
FROM marketplace_database.schema_name.table_name
GROUP BY region
ORDER BY total_value DESC;

-- Distance calculations (if coordinates are available)
SELECT 
  id,
  location_name,
  SQRT(POW(latitude - 40.7128, 2) + POW(longitude - 74.0060, 2)) as distance_from_nyc
FROM marketplace_database.schema_name.location_table
ORDER BY distance_from_nyc
LIMIT 10;
```

## Integrating with Internal Data

### Data Joining Strategies

#### Common Keys and Attributes

Identify columns that can join marketplace data with internal data:

- **Geographic Identifiers**: Zip codes, state/province codes, country codes
- **Time Dimensions**: Dates, months, quarters, years
- **Industry Classifications**: NAICS codes, SIC codes, custom categorizations
- **Product Identifiers**: UPCs, SKUs, ISBNs
- **Financial Metrics**: Currency codes, exchange rates

#### Join Examples

```sql
-- Join marketplace demographic data with internal customer data
SELECT 
  c.customer_id,
  c.customer_name,
  c.zip_code,
  d.median_household_income,
  d.population,
  d.average_age
FROM internal_database.customers c
JOIN marketplace_database.demographics d ON c.zip_code = d.zip_code;

-- Enrich transaction data with economic indicators
SELECT 
  t.transaction_date,
  t.transaction_amount,
  t.product_category,
  e.gdp_growth,
  e.unemployment_rate,
  e.consumer_sentiment
FROM internal_database.transactions t
JOIN marketplace_database.economic_indicators e 
  ON DATE_TRUNC('quarter', t.transaction_date) = e.quarter_date
  AND t.region = e.region;
```

### Creating Integrated Views

Views simplify access to joined datasets:

```sql
-- Create a view combining internal and marketplace data
CREATE OR REPLACE VIEW integrated_customer_view AS
SELECT 
  c.customer_id,
  c.customer_name,
  c.email,
  c.zip_code,
  d.median_income,
  d.education_level,
  d.homeownership_rate
FROM internal_database.customers c
LEFT JOIN marketplace_database.demographics d ON c.zip_code = d.zip_code;

-- Create a secure view with row-level access control
CREATE OR REPLACE SECURE VIEW sales_with_market_data AS
SELECT 
  s.sale_id,
  s.product_id,
  s.sale_amount,
  s.sale_date,
  m.market_size,
  m.competitor_price,
  m.market_share
FROM internal_database.sales s
JOIN marketplace_database.market_intelligence m 
  ON s.product_id = m.product_id
  AND s.region = m.region;
```

## Data Transformation and Enrichment

### Preprocessing Marketplace Data

Sometimes marketplace data requires preprocessing:

```sql
-- Standardize geographic names
CREATE OR REPLACE TABLE standardized_regions AS
SELECT 
  id,
  CASE 
    WHEN region = 'NY' THEN 'New York'
    WHEN region = 'CA' THEN 'California'
    -- Add other mappings as needed
    ELSE region
  END as standardized_region,
  value
FROM marketplace_database.schema_name.table_name;

-- Convert units of measurement
CREATE OR REPLACE TABLE converted_metrics AS
SELECT 
  id,
  date,
  -- Convert from imperial to metric
  value * 2.54 as value_cm,
  value_fahrenheit - 32 * 5/9 as value_celsius
FROM marketplace_database.schema_name.table_name;
```

### Creating Derived Datasets

Generate new insights from marketplace data:

```sql
-- Create aggregated dataset
CREATE OR REPLACE TABLE regional_metrics AS
SELECT 
  region,
  COUNT(*) as entity_count,
  AVG(value1) as avg_value1,
  SUM(value2) as total_value2,
  MAX(date_column) as latest_date
FROM marketplace_database.schema_name.table_name
GROUP BY region;

-- Create composite score
CREATE OR REPLACE TABLE opportunity_score AS
SELECT 
  zip_code,
  (income_score * 0.4) + 
  (growth_score * 0.3) + 
  (stability_score * 0.3) as opportunity_index
FROM (
  SELECT 
    zip_code,
    (median_income / 100000) * 100 as income_score,
    population_growth * 20 as growth_score,
    (1 - unemployment_rate) * 100 as stability_score
  FROM marketplace_database.demographics
) scores;
```

## Analysis Techniques for Marketplace Data

### Descriptive Analytics

Extract patterns and summaries:

```sql
-- Distribution analysis
SELECT 
  FLOOR(value / 100) * 100 as value_bucket,
  COUNT(*) as frequency
FROM marketplace_database.schema_name.table_name
GROUP BY value_bucket
ORDER BY value_bucket;

-- Correlation analysis
SELECT 
  CORR(metric1, metric2) as correlation_coefficient
FROM marketplace_database.schema_name.table_name;
```

### Predictive Analytics

Use marketplace data for forecasting:

```sql
-- Simple moving average forecast
SELECT 
  future_date,
  AVG(value) OVER (
    ORDER BY date_column 
    ROWS BETWEEN 6 PRECEDING AND CURRENT ROW
  ) as forecast_value
FROM marketplace_database.time_series_data;

-- Linear regression using SQL (simplified)
WITH regression_calc AS (
  SELECT 
    COUNT(*) as n,
    SUM(x) as sum_x,
    SUM(y) as sum_y,
    SUM(x*x) as sum_x_squared,
    SUM(x*y) as sum_xy
  FROM marketplace_database.regression_data
)
SELECT 
  (n * sum_xy - sum_x * sum_y) / (n * sum_x_squared - sum_x * sum_x) as slope,
  (sum_y - ((n * sum_xy - sum_x * sum_y) / (n * sum_x_squared - sum_x * sum_x)) * sum_x) / n as intercept
FROM regression_calc;
```

## Data Quality and Governance

### Validating Marketplace Data

Always verify the quality of marketplace data:

```sql
-- Check for missing values
SELECT 
  column_name,
  COUNT(*) as total_rows,
  SUM(CASE WHEN column_value IS NULL THEN 1 ELSE 0 END) as null_count,
  SUM(CASE WHEN column_value IS NULL THEN 1 ELSE 0 END) / COUNT(*) as null_percentage
FROM marketplace_database.schema_name.table_name
GROUP BY column_name;

-- Identify outliers
SELECT *
FROM marketplace_database.schema_name.table_name
WHERE numeric_column > (
  SELECT AVG(numeric_column) + 3 * STDDEV(numeric_column)
  FROM marketplace_database.schema_name.table_name
);
```

### Documenting Marketplace Data Usage

Maintain documentation for governance:

```sql
-- Create a data catalog table
CREATE TABLE IF NOT EXISTS data_catalog (
  source_database STRING,
  source_schema STRING,
  source_table STRING,
  source_column STRING,
  business_definition STRING,
  data_steward STRING,
  refresh_frequency STRING,
  last_refresh_date DATE,
  usage_restrictions STRING
);

-- Insert marketplace dataset information
INSERT INTO data_catalog VALUES
  ('marketplace_database', 'schema_name', 'table_name', 'column_name',
   'Business definition here', 'Data Steward Name',
   'Daily', CURRENT_DATE(), 'Internal analysis only');
```

## Sharing and Collaboration

### Creating Data Products

Package marketplace and internal data into reusable assets:

```sql
-- Create a shareable dataset combining marketplace and internal data
CREATE OR REPLACE SECURE VIEW shared.customer_insights AS
SELECT 
  c.customer_segment,
  d.region,
  COUNT(DISTINCT c.customer_id) as customer_count,
  AVG(c.lifetime_value) as avg_ltv,
  AVG(d.market_potential) as market_potential,
  SUM(c.revenue) / SUM(d.market_size) as market_penetration
FROM internal.customers c
JOIN marketplace_database.market_data d ON c.region_id = d.region_id
GROUP BY c.customer_segment, d.region;
```

### Collaboration Best Practices

When sharing marketplace data insights:

- **Cite Sources**: Clearly attribute marketplace data providers
- **Document Methodology**: Explain how data was processed and analyzed
- **Version Control**: Track changes to queries and derived datasets
- **Access Controls**: Ensure proper permissions for marketplace data

## Industry-Specific Use Cases

### Financial Services

```sql
-- Risk assessment with credit data
SELECT 
  c.customer_id,
  c.loan_amount,
  m.default_probability,
  c.loan_amount * m.default_probability as expected_loss
FROM internal.loan_applications c
JOIN marketplace_database.credit_risk m ON c.zip_code = m.zip_code
WHERE c.application_date > CURRENT_DATE() - 30;
```

### Retail and Consumer Goods

```sql
-- Market basket analysis with demographic enrichment
SELECT 
  p.product_category,
  d.income_bracket,
  COUNT(DISTINCT o.order_id) as order_count,
  AVG(o.order_value) as avg_order_value
FROM internal.orders o
JOIN internal.customers c ON o.customer_id = c.customer_id
JOIN internal.order_items i ON o.order_id = i.order_id
JOIN internal.products p ON i.product_id = p.product_id
JOIN marketplace_database.demographics d ON c.zip_code = d.zip_code
GROUP BY p.product_category, d.income_bracket
ORDER BY p.product_category, d.income_bracket;
```

### Healthcare

```sql
-- Population health analysis
SELECT 
  h.treatment_type,
  d.age_group,
  d.gender,
  COUNT(DISTINCT h.patient_id) as patient_count,
  AVG(h.treatment_cost) as avg_cost,
  AVG(d.health_index) as avg_health_index
FROM internal.healthcare_records h
JOIN marketplace_database.population_health d 
  ON h.zip_code = d.zip_code
  AND h.age_group = d.age_group
  AND h.gender = d.gender
GROUP BY h.treatment_type, d.age_group, d.gender;
```

## Advanced Integration Topics

### Real-time Integration

For frequently updated marketplace data:

```sql
-- Create a task to refresh integrated views
CREATE OR REPLACE TASK refresh_market_insights
  WAREHOUSE = compute_wh
  SCHEDULE = 'USING CRON 0 */6 * * * UTC'
AS
TRUNCATE TABLE derived.market_insights;

INSERT INTO derived.market_insights
SELECT 
  current_timestamp() as snapshot_time,
  i.product_category,
  m.region,
  SUM(i.sales_amount) as internal_sales,
  m.market_size,
  SUM(i.sales_amount) / m.market_size as market_share
FROM internal.sales i
JOIN marketplace_database.market_size m 
  ON i.product_category = m.product_category
  AND i.region = m.region
  AND i.date = m.date
WHERE i.date >= CURRENT_DATE() - 30
GROUP BY i.product_category, m.region, m.market_size;
```

### Multi-provider Integration

Combining data from multiple marketplace providers:

```sql
-- Integrate demographic, economic, and weather data with sales
SELECT 
  s.sale_date,
  s.store_id,
  s.sales_amount,
  d.population,
  d.median_income,
  e.unemployment_rate,
  w.temperature,
  w.precipitation
FROM internal.daily_sales s
JOIN marketplace_provider1.demographics d ON s.zip_code = d.zip_code
JOIN marketplace_provider2.economic e 
  ON s.region = e.region
  AND DATE_TRUNC('month', s.sale_date) = e.month
JOIN marketplace_provider3.weather w
  ON s.zip_code = w.zip_code
  AND s.sale_date = w.date;
```

!!! info "Next Steps"
    Continue to [Example Projects](example-projects.md) to see complete, real-world applications using Snowflake Marketplace data.

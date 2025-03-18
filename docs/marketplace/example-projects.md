# Example Projects Using Snowflake Marketplace Data

This section provides complete, practical examples of projects that leverage Snowflake Marketplace data. Each example includes step-by-step instructions, SQL code, and visualization guidance to help you implement similar solutions in your organization.

## Project 1: Customer Segmentation with Demographic Data

This project demonstrates how to enrich internal customer data with demographic information from the Snowflake Marketplace to create more sophisticated customer segments.

### Business Objective

- Create customer segments based on both purchase behavior and demographic characteristics
- Target marketing campaigns more effectively using enhanced segmentation
- Improve customer lifetime value by tailoring offerings to segment needs

### Required Datasets

- **Internal Data**: Customer purchase history, customer location data
- **Marketplace Data**: Demographic data by geographic area (e.g., from providers like Acxiom, Experian, or SafeGraph)

### Implementation Steps

#### 1. Acquire Demographic Data

First, acquire appropriate demographic data from the Snowflake Marketplace:

1. Navigate to the Snowflake Marketplace
2. Search for demographic data providers
3. Request access to a dataset that includes income levels, age ranges, education levels, etc.
4. Once approved, the data will appear as a new database in your Snowflake account

#### 2. Prepare Internal Customer Data

Ensure your internal customer data includes geographic identifiers that can be matched with the marketplace data:

```sql
-- Create a view of customer data with standardized zip codes
CREATE OR REPLACE VIEW customer_data_prepared AS
SELECT
  customer_id,
  first_name,
  last_name,
  email,
  LPAD(zip_code, 5, '0') AS zip_code,  -- Ensure 5-digit format
  city,
  state,
  SUM(purchase_amount) AS total_spent,
  COUNT(DISTINCT order_id) AS order_count,
  MAX(purchase_date) AS last_purchase_date
FROM customers c
JOIN orders o ON c.customer_id = o.customer_id
WHERE purchase_date >= DATEADD(year, -1, CURRENT_DATE())
GROUP BY 1, 2, 3, 4, 5, 6, 7;
```

#### 3. Join Customer and Demographic Data

Create an enriched customer view by joining internal and marketplace data:

```sql
-- Join customer data with demographic information
CREATE OR REPLACE VIEW customer_enriched AS
SELECT
  c.*,
  d.median_household_income,
  d.median_age,
  d.population_density,
  d.education_level_bachelors_or_higher,
  d.homeownership_rate
FROM customer_data_prepared c
LEFT JOIN demographic_database.zip_demographics d ON c.zip_code = d.zip_code;
```

#### 4. Create Customer Segments

Use both internal and demographic data to define customer segments:

```sql
-- Create customer segments
CREATE OR REPLACE TABLE customer_segments AS
WITH segment_calc AS (
  SELECT
    customer_id,
    first_name,
    last_name,
    email,
    zip_code,
    city,
    state,
    total_spent,
    order_count,
    last_purchase_date,
    median_household_income,
    median_age,
    population_density,
    education_level_bachelors_or_higher,
    homeownership_rate,
    NTILE(3) OVER (ORDER BY total_spent) AS spend_tertile,
    NTILE(3) OVER (ORDER BY median_household_income) AS income_tertile,
    CASE
      WHEN DATEDIFF(day, last_purchase_date, CURRENT_DATE()) <= 30 THEN 'Active'
      WHEN DATEDIFF(day, last_purchase_date, CURRENT_DATE()) <= 90 THEN 'Recent'
      ELSE 'Inactive'
    END AS recency_segment
  FROM customer_enriched
)
SELECT
  *,
  CASE
    WHEN spend_tertile = 3 AND income_tertile = 3 THEN 'High Value'
    WHEN spend_tertile = 3 AND income_tertile = 2 THEN 'Growth Potential'
    WHEN spend_tertile = 2 AND income_tertile = 3 THEN 'Underperforming'
    WHEN spend_tertile = 1 AND income_tertile = 1 AND recency_segment = 'Active' THEN 'New Potential'
    ELSE 'Standard'
  END AS customer_segment
FROM segment_calc;
```

#### 5. Analyze Segments

Analyze the distribution and characteristics of each segment:

```sql
-- Segment distribution
SELECT
  customer_segment,
  COUNT(*) AS customer_count,
  ROUND(AVG(total_spent), 2) AS avg_spend,
  ROUND(AVG(order_count), 1) AS avg_orders,
  ROUND(AVG(median_household_income), 0) AS avg_income,
  ROUND(AVG(median_age), 1) AS avg_age,
  ROUND(AVG(education_level_bachelors_or_higher) * 100, 1) AS pct_college_educated,
  ROUND(AVG(homeownership_rate) * 100, 1) AS pct_homeowners
FROM customer_segments
GROUP BY customer_segment
ORDER BY avg_spend DESC;
```

#### 6. Visualize Segment Insights

Visualize the segments using your preferred BI tool (Tableau, Power BI, etc.):

1. Create a dashboard with the following visualizations:
   - Segment distribution pie chart
   - Average spend by segment bar chart
   - Geographic distribution of segments (map)
   - Demographic composition of each segment (radar chart)
   - Segment migration over time (for returning customers)

### Business Value

This segmentation approach provides several advantages:

- **Enhanced Targeting**: Marketing campaigns can target "Growth Potential" customers differently than "High Value" customers
- **Resource Allocation**: Customer service resources can be prioritized for "High Value" segments
- **Product Development**: Product offerings can be tailored to the demographic characteristics of key segments
- **Location Planning**: New locations or services can be planned based on geographic segment distribution

## Project 2: Market Expansion Analysis

This project demonstrates how to use market intelligence data from the Snowflake Marketplace to identify optimal locations for business expansion.

### Business Objective

- Identify high-potential geographic markets for expansion
- Quantify market opportunity size in candidate locations
- Prioritize expansion targets based on multiple factors

### Required Datasets

- **Internal Data**: Current store locations, sales performance by location
- **Marketplace Data**: 
  - Population and demographic data by geography
  - Business location data (e.g., from SafeGraph, Precisely, or Foursquare)
  - Economic indicators by region (e.g., from Knoema or Moody's)

### Implementation Steps

#### 1. Acquire Market Intelligence Data

First, acquire relevant market intelligence data from the Snowflake Marketplace:

1. Search for providers offering demographic, business location, and economic data
2. Request access to datasets that cover your regions of interest
3. Once approved, the data will appear as new databases in your Snowflake account

#### 2. Prepare Current Performance Data

Create a view that summarizes your current location performance:

```sql
-- Summarize current store performance
CREATE OR REPLACE VIEW store_performance AS
SELECT
  store_id,
  store_name,
  city,
  state,
  zip_code,
  latitude,
  longitude,
  DATE_TRUNC('year', opening_date) AS opening_year,
  SUM(sales_amount) AS annual_sales,
  COUNT(DISTINCT transaction_id) AS transaction_count,
  SUM(sales_amount) / COUNT(DISTINCT transaction_id) AS avg_transaction_value,
  SUM(profit_amount) AS annual_profit,
  SUM(profit_amount) / SUM(sales_amount) AS profit_margin
FROM stores s
JOIN transactions t ON s.store_id = t.store_id
WHERE transaction_date BETWEEN DATEADD(year, -1, CURRENT_DATE()) AND CURRENT_DATE()
GROUP BY 1, 2, 3, 4, 5, 6, 7, 8;
```

#### 3. Identify Target Market Characteristics

Analyze your successful locations to identify patterns:

```sql
-- Identify characteristics of successful stores
WITH store_tiers AS (
  SELECT
    store_id,
    store_name,
    city,
    state,
    zip_code,
    annual_sales,
    annual_profit,
    profit_margin,
    NTILE(4) OVER (ORDER BY annual_profit DESC) AS profit_quartile
  FROM store_performance
)
SELECT
  s.profit_quartile,
  COUNT(*) AS store_count,
  ROUND(AVG(s.annual_profit), 0) AS avg_profit,
  ROUND(AVG(s.profit_margin) * 100, 1) AS avg_margin,
  ROUND(AVG(d.median_household_income), 0) AS avg_income,
  ROUND(AVG(d.population_density), 1) AS avg_pop_density,
  ROUND(AVG(d.median_age), 1) AS avg_age,
  ROUND(AVG(e.unemployment_rate) * 100, 1) AS avg_unemployment,
  ROUND(AVG(b.competitor_count), 1) AS avg_competitors,
  ROUND(AVG(b.foot_traffic), 0) AS avg_foot_traffic
FROM store_tiers s
JOIN demographic_database.zip_demographics d ON s.zip_code = d.zip_code
JOIN economic_database.regional_indicators e ON s.state = e.state
JOIN business_database.location_intelligence b ON s.zip_code = b.zip_code
GROUP BY s.profit_quartile
ORDER BY s.profit_quartile;
```

#### 4. Score Potential Markets

Create a scoring model for potential new locations:

```sql
-- Score potential markets based on success factors
CREATE OR REPLACE TABLE market_expansion_targets AS
WITH success_factors AS (
  -- Extract top quartile store characteristics
  SELECT
    AVG(d.median_household_income) AS target_income,
    AVG(d.population_density) AS target_density,
    AVG(d.median_age) AS target_age,
    AVG(e.unemployment_rate) AS target_unemployment,
    AVG(b.competitor_count) AS target_competition,
    AVG(b.foot_traffic) AS target_traffic
  FROM store_performance s
  JOIN demographic_database.zip_demographics d ON s.zip_code = d.zip_code
  JOIN economic_database.regional_indicators e ON s.state = e.state
  JOIN business_database.location_intelligence b ON s.zip_code = b.zip_code
  WHERE annual_profit > (SELECT PERCENTILE_CONT(0.75) WITHIN GROUP (ORDER BY annual_profit) FROM store_performance)
),
current_markets AS (
  -- Get list of current zip codes
  SELECT DISTINCT zip_code FROM stores
),
candidate_markets AS (
  -- Calculate similarity score for markets without current stores
  SELECT
    d.zip_code,
    d.city,
    d.state,
    d.population,
    d.median_household_income,
    d.population_density,
    d.median_age,
    e.unemployment_rate,
    b.competitor_count,
    b.foot_traffic,
    -- Calculate similarity scores (higher is better)
    (1 - ABS(d.median_household_income - sf.target_income) / sf.target_income) * 100 AS income_score,
    (1 - ABS(d.population_density - sf.target_density) / sf.target_density) * 100 AS density_score,
    (1 - ABS(d.median_age - sf.target_age) / sf.target_age) * 100 AS age_score,
    (1 - ABS(e.unemployment_rate - sf.target_unemployment) / NULLIF(sf.target_unemployment, 0)) * 100 AS unemployment_score,
    (1 - ABS(b.competitor_count - sf.target_competition) / NULLIF(sf.target_competition, 0)) * 100 AS competition_score,
    (1 - ABS(b.foot_traffic - sf.target_traffic) / sf.target_traffic) * 100 AS traffic_score
  FROM demographic_database.zip_demographics d
  JOIN economic_database.regional_indicators e ON d.state = e.state
  JOIN business_database.location_intelligence b ON d.zip_code = b.zip_code
  CROSS JOIN success_factors sf
  WHERE d.zip_code NOT IN (SELECT zip_code FROM current_markets)
    AND d.population > 10000  -- Minimum population threshold
)
SELECT
  zip_code,
  city,
  state,
  population,
  median_household_income,
  population_density,
  median_age,
  unemployment_rate,
  competitor_count,
  foot_traffic,
  -- Weighted combined score (customize weights based on business priorities)
  ROUND(
    income_score * 0.25 +
    density_score * 0.20 +
    age_score * 0.15 +
    unemployment_score * 0.10 +
    competition_score * 0.15 +
    traffic_score * 0.15,
    1
  ) AS market_opportunity_score,
  RANK() OVER (ORDER BY 
    income_score * 0.25 +
    density_score * 0.20 +
    age_score * 0.15 +
    unemployment_score * 0.10 +
    competition_score * 0.15 +
    traffic_score * 0.15 DESC
  ) AS opportunity_rank
FROM candidate_markets;
```

#### 5. Visualize Expansion Opportunities

Create a dashboard to analyze expansion opportunities:

1. Map visualization showing:
   - Current store locations
   - Top 20 expansion targets with opportunity score
   - Heat map of market potential by region
2. Scatter plot comparing:
   - Market size (population) vs. opportunity score
   - Expected competition vs. expected foot traffic
3. Detail table of top expansion candidates with all metrics

### Business Value

This expansion analysis provides several benefits:

- **Data-Driven Decisions**: Replace gut feeling with objective metrics for expansion
- **Risk Reduction**: Target markets with similar characteristics to proven successes
- **Resource Optimization**: Focus expansion efforts on highest-potential locations
- **Timeline Planning**: Sequence expansion based on opportunity ranking

## Project 3: Predictive Sales Forecasting with Weather Data

This project demonstrates how to enhance sales forecasting by incorporating weather data from the Snowflake Marketplace.

### Business Objective

- Improve sales forecast accuracy by accounting for weather factors
- Optimize inventory planning based on weather-influenced demand
- Create store-specific forecasts that consider local weather patterns

### Required Datasets

- **Internal Data**: Historical sales data by product, store, and date
- **Marketplace Data**: Historical and forecast weather data (e.g., from Weather Source, AccuWeather, or Tomorrow.io)

### Implementation Steps

#### 1. Acquire Weather Data

First, acquire historical weather data from the Snowflake Marketplace:

1. Search for weather data providers in the marketplace
2. Request access to historical weather data covering your store locations
3. Once approved, the data will appear as a new database in your Snowflake account

#### 2. Prepare Sales and Weather Data

Create a view that combines sales and weather data:

```sql
-- Join historical sales with corresponding weather data
CREATE OR REPLACE VIEW sales_weather_combined AS
SELECT
  s.sale_date,
  s.store_id,
  s.store_name,
  s.store_zip_code,
  s.product_category,
  s.product_id,
  s.product_name,
  s.units_sold,
  s.sales_amount,
  w.temperature_max,
  w.temperature_min,
  w.temperature_avg,
  w.precipitation,
  w.snowfall,
  w.wind_speed,
  w.cloud_cover,
  w.relative_humidity,
  -- Derive weather condition categories
  CASE
    WHEN w.precipitation > 0.5 THEN 'Heavy Rain'
    WHEN w.precipitation > 0.1 THEN 'Light Rain'
    WHEN w.snowfall > 1.0 THEN 'Snow'
    WHEN w.temperature_max > 85 THEN 'Hot'
    WHEN w.temperature_min < 32 THEN 'Freezing'
    WHEN w.cloud_cover > 70 THEN 'Cloudy'
    ELSE 'Clear'
  END AS weather_condition,
  -- Add date attributes
  DAYOFWEEK(s.sale_date) AS day_of_week,
  MONTH(s.sale_date) AS month,
  CASE WHEN DAYNAME(s.sale_date) IN ('Sat', 'Sun') THEN 1 ELSE 0 END AS is_weekend
FROM daily_sales s
JOIN store_locations l ON s.store_id = l.store_id
JOIN weather_database.historical_weather w 
  ON l.store_zip_code = w.zip_code
  AND s.sale_date = w.date
WHERE s.sale_date BETWEEN DATEADD(year, -2, CURRENT_DATE()) AND CURRENT_DATE();
```

#### 3. Analyze Weather Impact on Sales

Analyze how different weather conditions affect sales across product categories:

```sql
-- Calculate average sales lift/drop by weather condition and product category
CREATE OR REPLACE TABLE weather_sales_impact AS
WITH base_sales AS (
  -- Calculate average daily sales by store, product category, month, and day of week
  -- This represents the "expected" sales baseline
  SELECT
    store_id,
    product_category,
    month,
    day_of_week,
    is_weekend,
    AVG(sales_amount) AS avg_baseline_sales,
    AVG(units_sold) AS avg_baseline_units
  FROM sales_weather_combined
  GROUP BY store_id, product_category, month, day_of_week, is_weekend
),
weather_sales AS (
  -- Calculate average sales by weather condition
  SELECT
    s.store_id,
    s.product_category,
    s.month,
    s.day_of_week,
    s.is_weekend,
    s.weather_condition,
    AVG(s.sales_amount) AS avg_weather_sales,
    AVG(s.units_sold) AS avg_weather_units
  FROM sales_weather_combined s
  GROUP BY store_id, product_category, month, day_of_week, is_weekend, weather_condition
)
SELECT
  w.store_id,
  w.product_category,
  w.month,
  w.day_of_week,
  w.is_weekend,
  w.weather_condition,
  b.avg_baseline_sales,
  w.avg_weather_sales,
  w.avg_weather_sales / NULLIF(b.avg_baseline_sales, 0) - 1 AS sales_impact_pct,
  b.avg_baseline_units,
  w.avg_weather_units,
  w.avg_weather_units / NULLIF(b.avg_baseline_units, 0) - 1 AS units_impact_pct
FROM weather_sales w
JOIN base_sales b 
  ON w.store_id = b.store_id
  AND w.product_category = b.product_category
  AND w.month = b.month
  AND w.day_of_week = b.day_of_week
  AND w.is_weekend = b.is_weekend
ORDER BY ABS(sales_impact_pct) DESC;
```

#### 4. Build Weather-Enhanced Forecast Model

Create a forecast model that incorporates weather factors:

```sql
-- Create a table with the forecast model
CREATE OR REPLACE TABLE sales_forecast AS
WITH date_series AS (
  -- Generate a series of future dates to forecast
  SELECT DATEADD(day, seq, CURRENT_DATE()) AS forecast_date
  FROM TABLE(GENERATOR(ROWCOUNT => 30)) -- 30-day forecast
),
store_products AS (
  -- Get distinct store and product category combinations
  SELECT DISTINCT store_id, product_category 
  FROM daily_sales
  WHERE sale_date >= DATEADD(month, -3, CURRENT_DATE())
),
forecast_base AS (
  -- Join dates with stores and products to create forecast rows
  SELECT
    d.forecast_date,
    sp.store_id,
    sp.product_category,
    DAYOFWEEK(d.forecast_date) AS day_of_week,
    MONTH(d.forecast_date) AS month,
    CASE WHEN DAYNAME(d.forecast_date) IN ('Sat', 'Sun') THEN 1 ELSE 0 END AS is_weekend
  FROM date_series d
  CROSS JOIN store_products sp
),
weather_forecast AS (
  -- Get forecast weather data
  SELECT
    f.forecast_date,
    w.zip_code,
    w.temperature_max,
    w.temperature_min,
    w.temperature_avg,
    w.precipitation,
    w.snowfall,
    CASE
      WHEN w.precipitation > 0.5 THEN 'Heavy Rain'
      WHEN w.precipitation > 0.1 THEN 'Light Rain'
      WHEN w.snowfall > 1.0 THEN 'Snow'
      WHEN w.temperature_max > 85 THEN 'Hot'
      WHEN w.temperature_min < 32 THEN 'Freezing'
      WHEN w.cloud_cover > 70 THEN 'Cloudy'
      ELSE 'Clear'
    END AS weather_condition
  FROM date_series f
  JOIN weather_database.forecast_weather w
    ON f.forecast_date = w.forecast_date
),
base_sales AS (
  -- Get baseline sales expectations
  SELECT
    store_id,
    product_category,
    month,
    day_of_week,
    is_weekend,
    AVG(sales_amount) AS avg_baseline_sales,
    AVG(units_sold) AS avg_baseline_units
  FROM sales_weather_combined
  WHERE sale_date >= DATEADD(month, -12, CURRENT_DATE())
  GROUP BY store_id, product_category, month, day_of_week, is_weekend
)
SELECT
  f.forecast_date,
  f.store_id,
  s.store_name,
  s.store_zip_code,
  f.product_category,
  f.day_of_week,
  f.month,
  f.is_weekend,
  w.temperature_max,
  w.temperature_min,
  w.temperature_avg,
  w.precipitation,
  w.snowfall,
  w.weather_condition,
  b.avg_baseline_sales AS baseline_sales_forecast,
  b.avg_baseline_units AS baseline_units_forecast,
  -- Apply weather impact factor
  CASE 
    WHEN i.sales_impact_pct IS NOT NULL 
    THEN b.avg_baseline_sales * (1 + i.sales_impact_pct)
    ELSE b.avg_baseline_sales
  END AS weather_adjusted_sales_forecast,
  CASE 
    WHEN i.units_impact_pct IS NOT NULL 
    THEN b.avg_baseline_units * (1 + i.units_impact_pct)
    ELSE b.avg_baseline_units
  END AS weather_adjusted_units_forecast,
  i.sales_impact_pct,
  i.units_impact_pct
FROM forecast_base f
JOIN store_locations s ON f.store_id = s.store_id
JOIN weather_forecast w ON f.forecast_date = w.forecast_date AND s.store_zip_code = w.zip_code
JOIN base_sales b 
  ON f.store_id = b.store_id
  AND f.product_category = b.product_category
  AND f.month = b.month
  AND f.day_of_week = b.day_of_week
  AND f.is_weekend = b.is_weekend
LEFT JOIN weather_sales_impact i
  ON f.store_id = i.store_id
  AND f.product_category = i.product_category
  AND f.month = i.month
  AND f.day_of_week = i.day_of_week
  AND f.is_weekend = i.is_weekend
  AND w.weather_condition = i.weather_condition
ORDER BY f.forecast_date, f.store_id, f.product_category;
```

#### 5. Visualize the Weather-Enhanced Forecast

Create a dashboard comparing baseline and weather-adjusted forecasts:

1. Time series chart showing:
   - Historical sales
   - Baseline forecast
   - Weather-adjusted forecast
   - Actual weather conditions

2. Heat map showing:
   - Product categories (rows)
   - Weather conditions (columns)
   - Color intensity based on sales impact

3. Geographic map showing:
   - Store locations
   - Forecast weather conditions
   - Expected sales impact

### Business Value

This weather-enhanced forecasting provides several benefits:

- **Improved Accuracy**: Sales forecasts that account for weather patterns are typically 5-20% more accurate than baseline forecasts
- **Operational Planning**: Schedule staff and inventory based on weather-driven demand fluctuations
- **Marketing Optimization**: Time promotions to coincide with favorable weather conditions
- **Financial Planning**: Better predict cash flow and revenue based on seasonal weather patterns

## Resources for Further Learning

To continue building your skills with Snowflake Marketplace datasets:

- [Snowflake Marketplace Documentation](https://docs.snowflake.com/en/user-guide/data-marketplace.html)
- [Snowflake Hands-On Labs](https://quickstarts.snowflake.com/)
- [Snowflake Community Forum](https://community.snowflake.com/)
- [Snowflake Blog - Marketplace Use Cases](https://www.snowflake.com/blog/category/marketplace/)

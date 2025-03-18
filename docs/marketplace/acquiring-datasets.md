# Acquiring Datasets from the Snowflake Marketplace

After finding relevant datasets in the Snowflake Marketplace, the next step is to acquire and set them up in your Snowflake account. This section guides you through the dataset acquisition process, from requesting access to configuring the shared data.

## Prerequisites for Acquiring Datasets

Before you begin the acquisition process, ensure you have:

- **Appropriate Permissions**: You need ACCOUNTADMIN role or similar privileges
- **Valid Snowflake Account**: An active account in good standing
- **Budget Approval**: For paid listings, ensure you have spending authorization
- **Network Configuration**: Some listings require specific IP allowlisting

## The Dataset Acquisition Process

### 1. Requesting Access

From a dataset's listing page:

1. Click the "Get" or "Request" button
2. For free listings, you may get immediate access
3. For paid listings, you'll see pricing details and payment options
4. For restricted listings, you may need to complete an application form

### 2. Provider Approval

Some datasets require explicit provider approval:

- The provider reviews your request
- They may ask for additional information
- Approval times vary (hours to days)
- You'll receive notification when approved

### 3. Acceptance and Configuration

Once approved:

1. You'll receive a notification in Snowflake
2. Accept the terms and conditions
3. Select a warehouse for query processing
4. Choose database name (or accept the default)

### 4. Access Confirmation

After configuration:

1. The provider's data appears as a database in your account
2. Snowflake creates necessary objects and permissions
3. You can immediately begin querying the data

## Understanding Acquisition Options

### One-Time vs. Subscription

Datasets can be acquired through different models:

- **One-Time Purchase**: Pay once for a static dataset
- **Subscription**: Ongoing access to continuously updated data
- **Trial Period**: Limited-time access to evaluate the data
- **Usage-Based**: Cost depends on query volume or data accessed

### Access Control Options

When acquiring data, consider access management:

- **Reader Account**: Limit access to specific users or roles
- **Database Name**: Choose a descriptive name for easy identification
- **Usage Monitoring**: Set up monitoring for consumption tracking

## Financial Considerations

### Pricing Models

Marketplace datasets use various pricing structures:

- **Free**: No cost (often for public or sample data)
- **Flat Fee**: Fixed recurring subscription cost
- **Tiered Pricing**: Cost based on usage levels
- **Pay-Per-Query**: Charges based on compute resources used
- **Custom Pricing**: Negotiated rates for specific needs

### Cost Management

To manage costs effectively:

1. **Review Terms**: Understand payment frequency and commitment periods
2. **Monitor Usage**: Track consumption, especially for usage-based pricing
3. **Optimize Queries**: Efficient queries reduce costs for usage-based models
4. **Set Alerts**: Create budget alerts to avoid unexpected charges

## Technical Setup

### Database Configuration

After acquisition, you may need to:

1. **Grant Access**: Provide appropriate roles access to the new database
2. **Set Up Views**: Create views that join marketplace data with internal data
3. **Create Aliases**: Consider creating friendly aliases for complex table names
4. **Document Structure**: Map and document the acquired data structure

### Testing Access

Verify successful acquisition with these steps:

```sql
-- List all databases to confirm the new database exists
SHOW DATABASES;

-- List tables in the acquired database
SHOW TABLES IN DATABASE your_acquired_database;

-- Run a sample query to verify data access
SELECT * FROM your_acquired_database.schema_name.table_name LIMIT 10;
```

## Troubleshooting Acquisition Issues

### Common Problems and Solutions

| Problem | Possible Solution |
|---------|------------------|
| Access Denied | Verify you have the ACCOUNTADMIN role or proper permissions |
| Database Not Visible | Check that the provider has approved your request |
| Empty Tables | Confirm the correct schema and table names |
| Query Failures | Ensure your warehouse has adequate resources |
| Billing Issues | Contact your Snowflake account representative |

### Getting Support

If you encounter issues:

1. **Snowflake Support**: For account or platform issues
2. **Provider Support**: For data-specific questions or concerns
3. **Marketplace Documentation**: For general acquisition guidance
4. **Community Forums**: For peer advice and experiences

## Managing Acquired Datasets

### Monitoring Usage

Track how your team uses marketplace data:

```sql
-- Query history for specific marketplace database
SELECT *
FROM TABLE(INFORMATION_SCHEMA.QUERY_HISTORY())
WHERE QUERY_TEXT ILIKE '%your_acquired_database%'
ORDER BY START_TIME DESC;
```

### Refreshing Data

For subscription datasets:

- Data typically updates automatically
- Check provider documentation for refresh schedule
- Set up monitoring to verify updates are occurring

### Canceling or Changing Subscriptions

If your needs change:

1. Navigate to your list of acquired datasets
2. Select the dataset you wish to modify or cancel
3. Follow the prompts to adjust or terminate access
4. Preserve any derived data before cancellation

## Enterprise Considerations

### Data Governance

Incorporate marketplace data into your governance framework:

- **Data Catalog**: Register external data sources
- **Lineage Tracking**: Document how marketplace data flows through your systems
- **Usage Policies**: Define appropriate use cases for third-party data
- **Compliance Checks**: Ensure usage complies with license terms

### Change Management

Prepare for changes to marketplace datasets:

- **Schema Evolution**: Providers may add, remove, or modify columns
- **Deprecation Notices**: Be alert for dataset retirement announcements
- **Version Management**: Some providers offer multiple versions

## Next Steps After Acquisition

After successfully acquiring a dataset:

1. Explore the data structure thoroughly
2. Develop a strategy for integrating with internal data
3. Create documentation for your team
4. Build initial queries and analyses

!!! info "Next Steps"
    Continue to [Using Datasets](using-datasets.md) to learn how to effectively work with acquired marketplace data.

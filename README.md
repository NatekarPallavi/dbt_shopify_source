<p align="center">
    <a alt="License"
        href="https://github.com/fivetran/dbt_shopify_source/blob/main/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" /></a>
    <a alt="Fivetran-Release"
        href="https://fivetran.com/docs/getting-started/core-concepts#releasephases">
        <img src="https://img.shields.io/badge/Fivetran Release Phase-_Beta-orange.svg" /></a>
    <a alt="dbt-core">
        <img src="https://img.shields.io/badge/dbt_Core™_version->=1.0.0_<2.0.0-orange.svg" /></a>
    <a alt="Maintained?">
        <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" /></a>
    <a alt="PRs">
        <img src="https://img.shields.io/badge/Contributions-welcome-blueviolet" /></a>
</p>

# Shopify Source dbt Package ([Docs](https://fivetran.github.io/dbt_shopify_source/))
# 📣 What does this dbt package do?
- Materializes [Shopify staging tables](https://fivetran.github.io/dbt_shopify_source/#!/overview/github_source/models/?g_v=1) which leverage data in the format described by [this ERD](https://fivetran.com/docs/applications/shopify/#schemainformation). These staging tables clean, test, and prepare your Shopify data from [Fivetran's connector](https://fivetran.com/docs/applications/shopify) for analysis by doing the following:
  - Name columns for consistency across all packages and for easier analysis
  - Adds freshness tests to source data
  - Adds column-level testing where applicable. For example, all primary keys are tested for uniqueness and non-null values.
- Generates a comprehensive data dictionary of your Shopify data through the [dbt docs site](https://fivetran.github.io/dbt_shopify_source/).
- These tables are designed to work simultaneously with our [Shopify transformation package](https://github.com/fivetran/dbt_shopify).

# 🎯 How do I use the dbt package?
## Step 1: Prerequisites
To use this dbt package, you must have the following:
- At least one Fivetran Shopify connector syncing data into your destination. 
- A **BigQuery**, **Snowflake**, **Redshift**, **Databricks**, or **PostgreSQL** destination.

## Step 2: Install the package
Include the following shopify_source package version in your `packages.yml` file.
> TIP: Check [dbt Hub](https://hub.getdbt.com/) for the latest installation instructions or [read the dbt docs](https://docs.getdbt.com/docs/package-management) for more information on installing packages.
```yaml
packages:
  - package: fivetran/shopify_source
    version: [">=0.7.0", "<0.8.0"]
```
## Step 3: Define database and schema variables
By default, this package runs using your destination and the `shopify` schema. If this is not where your Shopify data is (for example, if your Shopify schema is named `shopify_fivetran` and your `issue` table is named `usa_issue`), add the following configuration to your root `dbt_project.yml` file:

```yml
vars:
    shopify_database: your_destination_name
    shopify_schema: your_schema_name 
```

## Step 4: Disabling models
This package was designed with the intention that users have all relevant Shopify tables being synced by Fivetran. However, if you are a Shopify user that does not operate on returns or adjustments then you will not have the related source tables. As such, you may use the below variable configurations to disable the respective downstream models. All variables are `true` by default. To disable those models, add the below configuration in  your root `dbt_project.yml` file:

```yml
# dbt_project.yml

...
vars:
  shopify__using_order_adjustment:  false  # true by default
  shopify__using_order_line_refund: false  # true by default
  shopify__using_refund:      false  # true by default
```

## (Optional) Step 5: Additional configurations
<details><summary>Expand to view configurations</summary>
    
### Changing the Build Schema
By default this package will build the Shopify staging models within a schema titled (<target_schema> + `_stg_shopify`) in your target database. If this is not where you would like your staging Shopify data to be written to, add the following configuration to your `dbt_project.yml` file:

```yml
# dbt_project.yml

...
models:
  shopify_source:
    +schema: my_new_schema_name # leave blank for just the target_schema
```
### Change the source table references
If an individual source table has a different name than the package expects, add the table name as it appears in your destination to the respective variable:
> IMPORTANT: See this project's [`dbt_project.yml`](https://github.com/fivetran/dbt_shopify_source/blob/main/dbt_project.yml) variable declarations to see the expected names.
    
```yml
vars:
    shopify_<default_source_table_name>_identifier: your_table_name 
```
### Union Multiple Shopify Connectors
If you have multiple Shopify connectors in Fivetran and would like to use this package on all of them simultaneously, we have provided functionality to do so. The package will union all of the data together and pass the unioned table into the transformations. You will be able to see which source it came from in the `source_relation` column of each model. To use this functionality, you will need to set either the `shopify_union_schemas` or `shopify_union_databases` variables:

```yml
# dbt_project.yml

...
config-version: 2

vars:
    shopify_union_schemas: ['shopify_usa','shopify_canada'] # use this if the data is in different schemas/datasets of the same database/project
    shopify_union_databases: ['shopify_usa','shopify_canada'] # use this if the data is in different databases/projects but uses the same schema name
```
### Add Passthrough Columns
This package includes all source columns defined in the staging_columns.sql macro. To add additional columns to this package, do so using our pass-through column variables. This is extremely useful if you'd like to include custom fields to the package.

```yml
# dbt_project.yml

...
config-version: 2

vars:
  shopify_source:
    customer_pass_through_columns: []
    order_line_refund_pass_through_columns: []
    order_line_pass_through_columns: []
    order_pass_through_columns: []
    product_pass_through_columns: []
    product_variant_pass_through_columns: []
```


</details>

## (Optional) Step 6: Orchestrate your models with Fivetran Transformations for dbt Core™
<details><summary>Expand to view details</summary>
<br>
    
Fivetran offers the ability for you to orchestrate your dbt project through [Fivetran Transformations for dbt Core™](https://fivetran.com/docs/transformations/dbt). Learn how to set up your project for orchestration through Fivetran in our [Transformations for dbt Core setup guides](https://fivetran.com/docs/transformations/dbt#setupguide).
</details>
    
# 🔍 Does this package have dependencies?
This dbt package is dependent on the following dbt packages. Please be aware that these dependencies are installed by default within this package. For more information on the following packages, refer to the [dbt hub](https://hub.getdbt.com/) site.
> IMPORTANT: If you have any of these dependent packages in your own `packages.yml` file, we highly recommend that you remove them from your root `packages.yml` to avoid package version conflicts.
```yml
packages:
    - package: fivetran/fivetran_utils
      version: [">=0.3.0", "<0.4.0"]

    - package: dbt-labs/dbt_utils
      version: [">=0.8.0", "<0.9.0"]
```
          
# 🙌 How is this package maintained and can I contribute?
## Package Maintenance
The Fivetran team maintaining this package _only_ maintains the latest version of the package. We highly recommend that you stay consistent with the [latest version](https://hub.getdbt.com/fivetran/jira_source/latest/) of the package and refer to the [CHANGELOG](https://github.com/fivetran/dbt_jira_source/blob/main/CHANGELOG.md) and release notes for more information on changes across versions.

## Contributions
A small team of analytics engineers at Fivetran develops these dbt packages. However, the packages are made better by community contributions! 

We highly encourage and welcome contributions to this package. Check out [this dbt Discourse article](https://discourse.getdbt.com/t/contributing-to-a-dbt-package/657) to learn how to contribute to a dbt package!

# 🏪 Are there any resources available?
- If you have questions or want to reach out for help, please refer to the [GitHub Issue](https://github.com/fivetran/dbt_jira_source/issues/new/choose) section to find the right avenue of support for you.
- If you would like to provide feedback to the dbt package team at Fivetran or would like to request a new dbt package, fill out our [Feedback Form](https://www.surveymonkey.com/r/DQ7K7WW).
- Have questions or want to just say hi? Book a time during our office hours [on Calendly](https://calendly.com/fivetran-solutions-team/fivetran-solutions-team-office-hours) or email us at solutions@fivetran.com.

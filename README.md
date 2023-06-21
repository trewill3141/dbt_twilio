<p align="center">
    <a alt="License"
        href="https://github.com/fivetran/dbt_twilio_source/blob/main/LICENSE">
        <img src="https://img.shields.io/badge/License-Apache%202.0-blue.svg" /></a>
    <a alt="dbt-core">
        <img src="https://img.shields.io/badge/dbt_Core™_version->=1.3.0_,<2.0.0-orange.svg" /></a>
    <a alt="Maintained?">
        <img src="https://img.shields.io/badge/Maintained%3F-yes-green.svg" /></a>
    <a alt="PRs">
        <img src="https://img.shields.io/badge/Contributions-welcome-blueviolet" /></a>
</p>


# Twilio Transformation dbt Package ([Docs](https://fivetran.github.io/dbt_twilio/))

- Produces modeled tables that leverage Twilio data from [Fivetran's connector](https://fivetran.com/docs/applications/twilio) in the format described by [this ERD](https://fivetran.com/docs/applications/twilio#schemainformation) and builds off the output of our [Twilio source package](https://github.com/fivetran/dbt_twilio_source).

- Enables you to better understand the efficacy of your email and SMS marketing efforts. It achieves this by:
  - Performing last-touch attribution on events in order to properly credit campaigns and flows with conversions
  - Enriching the core event table with data regarding associated users, flows, and campaigns
  - Aggregating key metrics, such as associated revenue, related to each user's interactions with individual campaigns and flows (and organic actions)
  - Aggregating these metrics further, to the grain of campaigns, flows, and individual users

The following table provides a detailed list of all models materialized within this package by default. 

| **Model**                | **Description**                                                                                                                                |
| ------------------------ | ---------------------------------------------------------------------------------------------------------------------------------------------- |
| [Twilio__events](https://github.com/fivetran/dbt_twilio/blob/main/models/twilio__events.sql)             | Each record represents a unique event in twilio, enhanced with a customizable last-touch attribution model associating events with flows and campaigns. Also includes information about the user who triggered the event. Materialized incrementally by default. |
| [Twilio__person_campaign_flow](https://github.com/fivetran/dbt_twilio/blob/main/models/twilio__person_campaign_flow.sql)             | Each record represents a unique person-campaign or person-flow combination, enriched with sums of the numeric values (i.e. revenue) associated with each kind of conversion, and counts of the number of triggered conversion events. |
| [Twilio__campaigns](https://github.com/fivetran/dbt_twilio/blob/main/models/twilio__campaigns.sql)             | Each record represents a unique campaign, enriched with user interaction metrics, any revenue attributed to the campaign, and other conversions. |
| [Twilio__flows](https://github.com/fivetran/dbt_twilio/blob/main/models/twilio__flows.sql)             | Each record represents a unique flow, enriched with user interaction metrics, any revenue attributed to the flow, and other conversions. |
| [Twilio__persons](https://github.com/fivetran/dbt_twilio/blob/main/models/twilio__persons.sql)             | Each record represents a unique user, enriched with metrics around the campaigns and flows they have interacted with, any associated revenue (organic as well as attributed to flows/campaigns), and their recent activity. |

# 🎯 How do I use the dbt package?

## Step 1: Prerequisites
To use this dbt package, you must have the following:

- At least one Fivetran Twilio connector syncing data into your destination.
- A **BigQuery**, **Snowflake**, **Redshift**, **PostgreSQL**, or **Databricks** destination.

### Databricks Dispatch Configuration
If you are using a Databricks destination with this package you will need to add the below (or a variation of the below) dispatch configuration within your `dbt_project.yml`. This is required in order for the package to accurately search for macros within the `dbt-labs/spark_utils` then the `dbt-labs/dbt_utils` packages respectively.
```yml
dispatch:
  - macro_namespace: dbt_utils
    search_order: ['spark_utils', 'dbt_utils']
```


## Step 2: Install the package
Include the following Twilio package version in your `packages.yml` file:
> TIP: Check [dbt Hub](https://hub.getdbt.com/) for the latest installation instructions or [read the dbt docs](https://docs.getdbt.com/docs/package-management) for more information on installing packages.
```yaml
packages:
  - package: fivetran/twilio
    version: [">=0.1.0", "<0.2.0"]
```
## Step 3: Define database and schema variables
By default, this package runs using your destination and the `Twilio` schema. If this is not where your Twilio data is (for example, if your Twilio schema is named `twilio_fivetran`), add the following configuration to your root `dbt_project.yml` file:

```yml
vars:
  twilio_database: your_database_name
  twilio_schema: your_schema_name
```
## (Optional) Step 4: Additional configurations

<details><summary>Expand for configurations</summary>

### Changing the Build Schema

By default, this package will build the Twilio final models within a schema titled (`<target_schema>` + `_twilio`), intermediate models in (`<target_schema>` + `_int_twilio`), and staging models within a schema titled (`<target_schema>` + `_stg_twilio`) in your target database. If this is not where you would like your modeled Twilio data to be written to, add the following configuration to your `dbt_project.yml` file:

```yml
# dbt_project.yml

...
models:
  twilio:
    +schema: my_new_schema_name # leave blank for just the target_schema
    intermediate:
      +schema: my_new_schema_name # leave blank for just the target_schema
  twilio_source:
    +schema: my_new_schema_name # leave blank for just the target_schema
```

> Note that if your profile does not have permissions to create schemas in your warehouse, you can set each `+schema` to blank. The package will then write all tables to your pre-existing target schema.

### Change the source table references
If an individual source table has a different name than the package expects, add the table name as it appears in your destination to the respective variable:

> IMPORTANT: See this project's [`dbt_project.yml`](https://github.com/fivetran/dbt_twilio/blob/main/dbt_project.yml) variable declarations to see the expected names.

```yml
vars:
    twilio_<default_source_table_name>_identifier: your_table_name 
```

</details>

## (Optional) Step 5: Orchestrate your models with Fivetran Transformations for dbt Core™  
<details><summary>Expand for more details</summary>

Fivetran offers the ability for you to orchestrate your dbt project through [Fivetran Transformations for dbt Core™](https://fivetran.com/docs/transformations/dbt). Learn how to set up your project for orchestration through Fivetran in our [Transformations for dbt Core setup guides](https://fivetran.com/docs/transformations/dbt#setupguide).

</details>

# 🔍 Does this package have dependencies?
This dbt package is dependent on the following dbt packages. Please be aware that these dependencies are installed by default within this package. For more information on the following packages, refer to the [dbt hub](https://hub.getdbt.com/) site.
> IMPORTANT: If you have any of these dependent packages in your own `packages.yml` file, we highly recommend that you remove them from your root `packages.yml` to avoid package version conflicts.
    
```yml
packages:
    - package: fivetran/twilio_source
      version: [">=0.1.0", "<0.2.0"]

    - package: fivetran/fivetran_utils
      version: [">=0.4.0", "<0.5.0"]

    - package: dbt-labs/dbt_utils
      version: [">=1.0.0", "<2.0.0"]
```
# 🙌 How is this package maintained and can I contribute?
## Package Maintenance
The Fivetran team maintaining this package _only_ maintains the latest version of the package. We highly recommend you stay consistent with the [latest version](https://hub.getdbt.com/fivetran/twilio_source/latest/) of the package and refer to the [CHANGELOG](https://github.com/fivetran/dbt_twilio_source/blob/main/CHANGELOG.md) and release notes for more information on changes across versions.

## Contributions
A small team of analytics engineers at Fivetran develops these dbt packages. However, the packages are made better by community contributions! 

We highly encourage and welcome contributions to this package. Check out [this dbt Discourse article](https://discourse.getdbt.com/t/contributing-to-a-dbt-package/657) on the best workflow for contributing to a package!

# 🏪 Are there any resources available?
- If you have questions or want to reach out for help, please refer to the [GitHub Issue](https://github.com/fivetran/dbt_twilio/issues/new/choose) section to find the right avenue of support for you.
- If you would like to provide feedback to the dbt package team at Fivetran or would like to request a new dbt package, fill out our [Feedback Form](https://www.surveymonkey.com/r/DQ7K7WW).
- Have questions or want to be part of the community discourse? Create a post in the [Fivetran community](https://community.fivetran.com/t5/user-group-for-dbt/gh-p/dbt-user-group) and our team along with the community can join in on the discussion!
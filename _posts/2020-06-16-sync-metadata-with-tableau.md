---
published: false
---
## Part 1: Metadata synchronisation between Tableau and BigQuery (or other dbs)

### Overview of of we're trying to address

_Note: This blog uses BigQuery as the datasource, but the concepts could be applied to any other datasource. BigQuery was chosen simply because a client recently asked me about it._

Recently a client who was building a data platform with BigQuery had a query about synchronising metadata (Column Descriptions in this case) between the BigQuery and Tableau's table assets. This gave me a good excuse to do some self-learning about using our Metadata API (part of Data Catalog) and REST API.

This is Part 1 which looks syncronisation from BigQuery to Tableau. When I write Part 2 I'll cover the other direction which is the other direction; from Tableau to BigQuery.

Currently the metadata API only allows for updates of table assets, not of Tableau Data Sources. I'm hopeful that Data Sources metadata updates are supported via the API in a future release.

The aim in this is to introduce & show the concepts / API endpoints involved and not to provide production quality code.

First thing is to introduce what we're focused on. In Tableau Server assets are structured into a lineage, so you can see the end to end linkage between Dashboards & underlying Databases. In this case we're looking at the Tables section of the lineage (highlighted with an arrow), which map to tables from the underlying database. The node selected below this is a [Tableau Data Source](https://help.tableau.com/current/pro/desktop/en-us/publish_datasources_about.htm#:~:text=A%20Tableau%20data%20source%20consists,tables%20from%20different%20data%20types.) which is similar to a semantic layer or reusable data model on top of one or more databases - we'll look at these in Part 2.

![lineage.png]({{site.baseurl}}/_posts/lineage.png)

In this example we'll look at a table called Orders, which comes from my BigQuery instance.

**It currently doesn't have any metadata descibing each column:**

![orders_table.png]({{site.baseurl}}/_posts/orders_table.png)

**But the metadata for Description exists within BigQuery:**

![bq_schema.png]({{site.baseurl}}/_posts/bq_schema.png)

We want to address this by syncing from BigQuery to Tableau.

### What we need to approach this

We'll need a few things to approach this and I've chosen Python to do this due to the great Tableau Server Client library:

1. A way to query metadata from BigQuery. For this we can use the [BigQuery API Client Libraries](https://cloud.google.com/bigquery/docs/reference/libraries).
2. A way to query metadata from Tableau Server or Online. This is done via the Tableau REST API which we can query more easily with the [Tableau Server Client library](https://tableau.github.io/server-client-python/docs/).
3. We'll want a way to run the synchronisation regularly, in this case a Google Cloud Function is used but it could really be implemented anywhere you can schedule your code or script to run. 

### Querying for Tableau metadata

For this we need to rely on the Metadata API which allows you to use GraphiQL to query our data catalog for lineage information. You can read more about it [here](https://help.tableau.com/current/api/metadata_api/en-us/index.html).

This is a query that will find all tables connected to a BigQuery database and for each table will retrieve a number of different attributes:

	query getTables {
        databaseTables (filter:{connectionType:"bigquery"}) {
            id
            luid
            connectionType
            fullName
            schema
            isCertified
        }
    }

asdf

	{
 	 "data": {
      "databaseTables": [
        {
          "id": "c91df9a7-e3e4-7edb-ba05-17e7a06d2cc3",
          "luid": "16ae4890-3a29-4c42-aaba-ea5b4f312a6e",
          "connectionType": "bigquery",
          "fullName": "[google-project-id.google-project-id].[orders]",
          "schema": "superstore",
          "isCertified": true
        }
        ]
      }
  	}





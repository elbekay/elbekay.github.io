---
published: true
---
## Part 1: Metadata synchronisation between Tableau and BigQuery (or other dbs)

### Overview of of we're trying to address

_Note: This blog uses BigQuery as the datasource, but the concepts could be applied to any other datasource. BigQuery was chosen simply because a client recently asked me about it._

Recently a client who was building a data platform with BigQuery had a query about synchronising metadata (Column Descriptions in this case) between the BigQuery and Tableau's table assets. This gave me a good excuse to do some self-learning about using our Metadata API (part of Data Catalog) and REST API.

This is Part 1 which looks synchronisation from BigQuery to Tableau. When I write Part 2 I'll cover the other direction which is the other direction; from Tableau to BigQuery.

Currently the metadata API only allows for updates of table assets, not of Tableau Data Sources. I'm hopeful that Data Sources metadata updates are supported via the API in a future release.

The aim in this is to introduce & show the concepts / API endpoints involved and not to provide production quality code.

First thing is to introduce what we're focused on. In Tableau Server assets are structured into a lineage, so you can see the end to end linkage between Dashboards & underlying Databases. In this case we're looking at the Tables section of the lineage (highlighted with an arrow), which map to tables from the underlying database. The node selected below this is a [Tableau Data Source](https://help.tableau.com/current/pro/desktop/en-us/publish_datasources_about.htm#:~:text=A%20Tableau%20data%20source%20consists,tables%20from%20different%20data%20types.) which is similar to a semantic layer or reusable data model on top of one or more databases - we'll look at these in Part 2.

![lineage.png]({{site.baseurl}}/images/lineage.png)

In this example we'll look at a table called Orders, which comes from my BigQuery instance.

**It currently doesn't have any metadata descibing each column:**

![orders_table.png]({{site.baseurl}}/images/orders_table.png)

**But the metadata for Description exists within BigQuery:**

![bq_schema.png]({{site.baseurl}}/images/bq_schema.png)

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

This will produce an output like below.
Two key pieces of information we need are
1. The value under **fullName** to link to BigQuery is the table id.
2. The **luid** which is the used to uniquely identify the Table when using the Tableau REST API metadata methods.


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

You can query Tableau Server with GraphiQL using the Tableau Server Client library using the _metadata.query_ function which will return JSON just like above.

    server.metadata.query(query=metadata_query)

We also need to ask populate the Columns (Name, ID, Description etc.) for each table, for this we can use the _tables.populatecolumns_ method using the _luid_ value from the metadata api:

    #get table by id
    table = server.tables.get_by_id(table_id)
    
    # populate columns
    server.tables.populate_columns(table)

Finally we need a way to update the metadata (once we retrieve it from BigQuery) for this we can use _server.tables.updatecolumn_:

	tableau_column.description = "My New Description"
    server.tables.update_column(tableau_table, tab_column)


### Querying BigQuery

For BigQuery metadata queries we're using the BigQuery python library and all we need is the table_id (which we got from the Tableau Metadata API, from there we can access the schema (which contains the metadata).

	# get the BigQuery table by ID in format [google-project-id].[google-project-id].[orders]
    bigquery_table = client.get_table(table_id)
    # access the schema
	schema = bigquery_table.schema
    # loop through columns 
    for column in schema:
    	# do stuff
        description = column.description
        
### Putting it together

We have the basic building blocks now so we can:

1. Query & iterate through tables on Tableau Server that connect to BigQuery
2. For each of the tables from (1.) query BigQuery for the table.
3. Match the columns names between BigQuery & Tableau columns, and update the description for each column in Tableau.
4. Push the metadata back to Tableau.

After running the rough sample code below we find the descriptions populated in Tableau. You can see a [video of this here](https://www.youtube.com/watch?v=l6_uL7GVFS0).

![metadata_populated.png]({{site.baseurl}}/images/metadata_populated.png)


Rough sample code:

      from google.cloud import bigquery
      import tableauserverclient as TSC

      tableau_auth = TSC.PersonalAccessTokenAuth('tsc', 'my_persona_token', 'mysite')
      server = TSC.Server('https://tableauserver', use_server_version=True)

      # sign into Tableau server
      tab_serv = server.auth.sign_in(tableau_auth)

      # create bigquery client
      client = bigquery.Client()

      def sync_metadata(request):
          # get tables which connect to bigquery
          tables = getTables()

          # loop through tables from tableau
          #   get bigquery table by id
          #   sync metadata between them
          for table in tables:
              tab_table = getColsByTableID( table['luid'] )
              bq_dataset_id = getBQDatasetID( table )
              bq_table = get_table( bq_dataset_id )
              updateColumns(bq_table, tab_table)

      # get a bigquery table by bigquery table id
      def get_table(table_id):
          return client.get_table(table_id)

      # get tables from tableau which connect to bigquery
      #   connection type is a filter, could be snowflake/redshift/other dbs
      def getTables(connectionType = "bigquery"):

          # GraphiQL metadata query for tableau 
          md_query = '''
          query getTables {
              databaseTables (filter:{connectionType:"%s"}) {
                  id
                  luid
                  connectionType
                  fullName
                  schema
                  isCertified
              }
          }
          ''' % format(connectionType)

          return server.metadata.query(query=md_query)['data']['databaseTables']

      # tidy bigquery table id
      def getBQDatasetID(md_bq_table):
          return md_bq_table['fullName'].replace('[','').replace(']','')

      # sync metadata
      def updateColumns(db_table, tableau_table):
          # for each column in the db_table
          #   check for a matching column in the tableau_table
          #   then update the description of the tableau_table
          for db_col in db_table.schema:
              for tab_col in tableau_table.columns:
                  if db_col.name == tab_col.name:
                      if db_col.description is None or len(db_col.description) > 0:
                          tab_col.description = db_col.description
                      else:
                          tab_col.description = " "

                      server.tables.update_column(tableau_table, tab_col)

      # populate columns for tableau tables
      def getColsByTableID(table_id):
          #get table by id
          table = server.tables.get_by_id(table_id)

          # populate columns
          server.tables.populate_columns(table)

          return table

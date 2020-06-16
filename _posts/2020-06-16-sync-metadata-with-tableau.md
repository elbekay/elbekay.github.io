---
published: false
---
## Part 1: Metadata synchronisation between Tableau and BigQuery (or other dbs)

_Note: This blog uses BigQuery as the datasource, but the concepts could be applied to any other datasource. BigQuery was chosen simply because a client recently asked me about it._

Recently a client who was building a data platform with BigQuery had a query about synchronising metadata between the BigQuery and Tableau's table assets. This gave me a good excuse to do some self-learning about using our Metadata API (part of Data Catalog) and REST API.

This is Part 1 which covers syncronisation from BigQuery to Tableau. When I write Part 2 I'll cover the other direction which is the other direction; from Tableau to BigQuery.

Currently the metadata API only allows for updates of table assets, not of Tableau Data Sources. I'm hopeful that Data Sources metadata updates are supported via the API in a future release.


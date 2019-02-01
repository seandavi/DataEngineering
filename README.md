# DataEngineering
Random notes on big data technologies focused on engineering

## BigQuery

- How can I dump the schema from a BigQuery table as json?
  See [StackOverflow answer](https://stackoverflow.com/a/50921844/459633) and [schema docs]
  ```
  bq show --schema --format=prettyjson [PROJECT_ID]:[DATASET].[TABLE] > [SCHEMA_FILE]
  bq show --schema --format=prettyjson mydataset.mytable > /tmp/myschema.json
  ```
[schema docs]: https://cloud.google.com/bigquery/docs/managing-table-schemas

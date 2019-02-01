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
- How do I load a [json-lines format](http://jsonlines.org/) file into BigQuery from google cloud storage?

  Note that it is not a good idea to gzip files to load into BigQuery, as it precludes reading in parallel. See [docs](https://cloud.google.com/bigquery/docs/loading-data-cloud-storage-json) for details.
  ```
  bq --location=US load --source_format=NEWLINE_DELIMITED_JSON mydataset.mytable gs://mybucket/mydata.json ./myschema.json
  ```
  or with autodetect:
  ```
  bq --location=US load --autodetect --source_format=NEWLINE_DELIMITED_JSON mydataset.mytable gs://mybucket/mydata.json
  ```
  

# API Overview

<info>
- **Title**: Opendatasoft Explore API
- **Version**: v2.1
- **Description**: REST API providing access to platform datasets. Only HTTP `GET` is supported and responses are JSON. Includes links for navigation and uses Opendatasoft Query Language (ODSQL) for most parameters.
</info>

<servers>
- `https://data.stad.gent/api/explore/v2.1`
- `https://documentation-resources.opendatasoft.com/api/explore/v2.1` (example portal)
</servers>

<security>
- API key is passed in the `apikey` query parameter.
</security>

<tags>
- **Catalog** – enumerate datasets
- **Dataset** – work on records
</tags>

<endpoints>
- **GET /catalog/datasets** – list datasets
- **GET /catalog/exports** – list available catalog export formats
- **GET /catalog/exports/{format}** – export catalog in given format
- **GET /catalog/exports/csv** – export catalog in CSV
- **GET /catalog/exports/dcat{dcat_ap_format}** – export catalog in RDF/DCAT
- **GET /catalog/facets** – list catalog facets
- **GET /catalog/datasets/{dataset_id}/records** – query dataset records
- **GET /catalog/datasets/{dataset_id}/exports** – list dataset export formats
- **GET /catalog/datasets/{dataset_id}/exports/{format}** – export dataset in given format
- **GET /catalog/datasets/{dataset_id}/exports/csv** – export dataset in CSV
- **GET /catalog/datasets/{dataset_id}/exports/parquet** – export dataset in Parquet
- **GET /catalog/datasets/{dataset_id}/exports/gpx** – export dataset in GPX
- **GET /catalog/datasets/{dataset_id}** – dataset information
- **GET /catalog/datasets/{dataset_id}/facets** – dataset facets
- **GET /catalog/datasets/{dataset_id}/attachments** – dataset attachments
- **GET /catalog/datasets/{dataset_id}/records/{record_id}** – retrieve a single record
</endpoints>

<parameters>
Common query parameters include:
- `select` – choose fields or expressions
- `where` – filter expression using ODSL
- `order_by` – sorting expression
- `limit`/`offset` – pagination controls
- `refine`/`exclude` – facet filters
- `lang` – language of returned texts
- `timezone` – timezone for date fields (default `UTC`)
- `group_by` – grouping expression for aggregations
- `include_links` – add HATEOAS links (boolean)
- `include_app_metas` – include application metadata (boolean)
</parameters>


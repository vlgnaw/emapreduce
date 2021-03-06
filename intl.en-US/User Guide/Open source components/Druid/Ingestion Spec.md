# Ingestion Spec {#concept_hw3_drd_z2b .concept}

This section briefly introduces Ingestion Spec, the description file of the index data.

Ingestion Spec is a unified description of the format of the data to be indexed and how that data format is indexed by Druid. It is a JSON file, which consists of three parts:

```
{
    "dataSchema" : {...},
    "ioConfig" : {...},
    "tuningConfig" : {...}
}
```

|Key|Format|Description|Required|
|:--|:-----|:----------|:-------|
|dataSchema|JSON object|Describes the schema information of the data you want to consume. dataSchema is fixed and does not change with the way in which data is consumed|Yes|
|ioConfig|JSON object|Describe the source and destination of the data you want to consume. If the consumption method of the data is different, ioConfig is also different.|Yes|
|tuningConfig|JSON object|Configure the parameters of the data you want to consume. If the consumption method of the data is different, the adjustable parameters are also different.|No|

## dataSchema {#section_vqj_jrd_z2b .section}

dataSchema describes the format of the data and how to parse the data. The typical structure is as follows:

```
{
    "dataSrouce": <name_of_dataSource>,
    "parser": {
        "type": <>,
        "parseSpec": {
            "format": <>,
            "timestampSpec": {},
            "dimensionsSpec": {}
        }
    },
    "metricsSpec": {},
    "granularitySpec": {}
}
```

|Key|Format|Description|Required|
|:--|:-----|:----------|:-------|
|dataSource|String|Name of the data source|Yes|
|parser|JSON object|How the data is parsed|Yes|
|metricsSpec|Array of JSON objects|Aggregator list|Yes|
|granularitySpec|JSON object|Data aggregation settings, such as creating segments and aggregation granularity|Yes|

-   **parser**

    parser determines how your data is parsed correctly. metricsSpec defines how the data is clustered for calculation. granularitySpec defines the granularity of the data fragmentation and the granularity of the query.

    For the parser, there are two types: string and hadoopstring. The latter is used for Hadoop index jobs. ParseSpec is a specific definition of data format resolution.

    |Key|Format|Description|Required|
    |:--|:-----|:----------|:-------|
    |type|String|The data format can be "json", "jsonLowercase", "csv", or "tsv".|Yes|
    |timestampSpec|JSON object|Timestamp and timestamp type|Yes|
    |dimensionsSpec|JSON object|The dimension of the data \(which columns are included\)|Yes|

    For different data formats, additional parseSpec options may exist. The following table describes timestampSpec and dimensionsSpec.

    |Key|Format|Description|Required|
    |:--|:-----|:----------|:-------|
    |column|String|Columns corresponding to the timestamp|Yes|
    |format|String|The timestamp type can be "ISO", "millis", "POSIX ", "auto", or that supported by [joda time](http://joda-time.sourceforge.net/apidocs/org/joda/time/format/DateTimeFormat.html).|Yes|

    |Key|Format|Description|Required|
    |:--|:-----|:----------|:-------|
    |dimensions|JSON array|Describes which dimensions the data contains. Each dimension can be just a string. In addition, you can specify the attribute for the dimension. For example, the type of “dimensions”: \[“dimenssion1”, “dimenssion2”, “\{“type”: “long”, “name”: “dimenssion3”\}\] is string by default.|Yes|
    |dimensionExclusions|Array of JSON strings|Dimension to be deleted when data is consumed|No|
    |spatialDimensions|Array of JSON objects|Spatial dimension|No|

-   **metricsSpec**

    MetricsSpec is an array of JSON objects. It defines several aggregators. Aggregators typically have the following structures:

    ```
    ```json
    {
        "type": <type>,
        "name": <output_name>,
        "fieldName": <metric_name>
    }
    ```
    ```

    The following commonly used aggregators are provided by the authority:

    |Type|Type optional|
    |:---|:------------|
    |count|count|
    |sum|longSum, doubleSum, floatSum|
    |min/max|longMin/longMax, doubleMin/doubleMax, floatMin/floatMax|
    |first/last|longFirst/longLast, doubleFirst/doubleLast, floatFirst/floatLast|
    |javascript|javascript|
    |cardinality|cardinality|
    |hyperUnique|hyperUnique|

    **Note:** The last three are the advanced aggregators. For information about how to use them, see [Druid official documents](http://druid.io/docs/0.11.0/querying/aggregations.html).

-   **granularitySpec**

    Two aggregation modes are supported: “uniform” and “arbitrary”. The uniform mode aggregates data with a fixed interval of time. The arbitrary mode tries to make sure that each of the segments has the same size, but the time interval for aggregation is not fixed. “Uniform” is the default option at present.

    |Key|Format|Description|Required|
    |:--|:-----|:----------|:-------|
    |segmentGranularity|String|Segments granularity Uniform type|No, the default is “DAY”|
    |queryGranularity|String|Minimum data aggregation granularity for query|No|
    |rollup|Bool value|Aggregate or not|No, the default is “true”|
    |intervals|String|Time interval of data consumption|It is Yes for batch and No for realtime.|


## ioConfig {#section_v53_gsd_z2b .section}

ioConfig describes the data source. Here is an example of Hadoop index:

```
{
    "type": "hadoop",
    "inputSpec": {
        "type": "static",
        "paths": "hdfs://emr-header-1.cluster-6789:9000/druid/quickstart/wikiticker-2015-09-16-sampled.json"
    }
}
```

This part is not required for streaming data that is processed through Tranquility.

## Tunning Config {#section_fqd_3sd_z2b .section}

TuningConfig refers to some additional settings. For example, you can specify some MapReduce parameters to use Hadoop to create index for batch data. The contents of tunningConfig may vary based on the data source. For details about examples, see the example file or official document of this service.


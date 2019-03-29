# Ingest HTTP Plugin

The ingest HTTP plugin lets Elasticsearch to call an external web service and supplies the response to the document via the Ingest Node pipeline.

## Downloads

[ingest-http-5.2.0.zip](https://github.com/kosho/ingest-http/releases/download/5.2.0-url-encode/ingest-http-5.2.0.zip)

## How to build

### IntelliJ IDEA

This repository is an IntelliJ IDEA project with `.idea` directory. In order to make a zipped plugin archive, goto **Build** > **Build Artifacts**, then choose **ingest-http:zip**. The gradle wrapper is used to make a build.

### Dependencies

- Elasticsearch 5.2.0
- org.apache.httpcomponents:httpclient:4.5.3

## Usages

### Installation

Stop the Elasticsearch service, then use the `elasticsearch-plugin` command to install the plugin.

```shell
$ bin/elasticsearch-plugin install file:///<path_to>/ingest-http.zip

```

### Options

| Name             | Required | Default  | Description                                                   |
|------------------|:--------:|:--------:|---------------------------------------------------------------|
| `field`          | yes      |          | The field name to be passed.                                  |
| `target_field`   | no       | response | The target JSON entity where the response to be.              |
| `url_prefix`     | yes      |          | The request URL where `{}` is repalced with `field`.          |
| `extra_header`   | no       |          | An extra header to be added to the HTTP request.              |
| `ignore_missing` | no       | false    | `true` to ignore when `field` is empty or missing, otherwise `IllegalArgumentException` will be thrown.  |


### Using the plugin

Before registering the pipeline, you can use the [Simulate Pipeline API](https://www.elastic.co/guide/en/elasticsearch/reference/master/simulate-pipeline-api.html) to call the `search` processor with the request body.


**Supply location information from the ip address with freegeoip.net**

One of the typical use case is calling an external web service such as converting IP address to geolocation. In this example, `ip` field is passed to freegeoip.net and the response goes to the `response` field of the ingesting document.


```
POST _ingest/pipeline/_simulate
{
  "pipeline" :
  {
    "processors": [
      {
        "http" : {
          "field": "ip",
          "target_field": "response",
          "ignore_missing": true,
          "url_prefix" : "http://freegeoip.net/json/{}"
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "ip": "8.8.8.8"
      }
    }
  ]
}
```

**Query local/remote Elasticsearch cluster to get rating information for web sites**

Another example is issuing a URI search upon an Elasticsearch cluster either local or remote. The `host` field fulfills the query string in URL and the whole search result is going to be stored under the `_source` field. The authorization string is constructed with base64 encoded pair of user name and password which can be generated by `echo -n user_name:password | base64` on most of operating systems. 

```
POST _ingest/pipeline/_simulate
{
  "pipeline" :
  {
    "processors": [
      {
        "http" : {
          "field": "host",
          "url_prefix" : "http://localhost:9200/malicious-website-list/_search?q=host:{}&size=1",
          "extra_header": "Authorization: Basic ZWxhc3RpYzpjaGFuZ2VtZQ=="
        }
      }
    ]
  },
  "docs": [
    {
      "_index": "index",
      "_type": "type",
      "_id": "id",
      "_source": {
        "host": "elastic.co"
      }
    }
  ]
}
```
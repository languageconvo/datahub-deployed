Screenshots: see the /screenshots folder for a few screenshots of how to set up API calls in Postman!

URL: If you used elastic.co on AWS to create an Elasticsearch instance, the URL will be similar to
`https://your-es-app.us-west-2.aws.found.io/_index_template/datahub_usage_event_index_template` Find the URL in the elastic.co console. If you're using a different cloud service for Elasticsearch, the important part bit here is the `/_index_template/datahub_usage_event_index_template`; just add that to the end of your instance's URL.

Authorization: when making an API calls to elastic.co you can user username/password authentication. Get the username and password for your instance from the elastic.co console

Type: PUT

Body:
````
{
  "index_patterns": ["*datahub_usage_event*"],
  "data_stream": { },
  "priority": 500,
  "template": {
    "mappings": {
      "properties": {
        "@timestamp": {
          "type": "date"
        },
        "type": {
          "type": "keyword"
        },
        "timestamp": {
          "type": "date"
        },
        "userAgent": {
          "type": "keyword"
        },
        "browserId": {
          "type": "keyword"
        }
      }
    },
    "settings": {
      "index.lifecycle.name": "datahub_usage_event_policy"
    }
  }
}
````

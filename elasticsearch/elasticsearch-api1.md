Screenshots: see the /screenshots folder for a few screenshots of how to set up API calls in Postman!

URL: If you used elastic.co on AWS to create an Elasticsearch instance, the URL you'll need for this API call will be similar to 
`https://your-es-app.us-west-2.aws.found.io/_ilm/policy/datahub_usage_event_policy` Find the URL in the elastic.co console. If you're using a different cloud service for Elasticsearch, the important part bit here is the `/_ilm/policy/datahub_usage_event_policy`; just add that to the end of your instance's URL.

Authorization: when making an API calls to elastic.co you can user username/password authentication. Get the username and password for your instance from the elastic.co console

Type: PUT

Body:
````
{
  "policy": {
    "phases": {
      "hot": {
        "actions": {
          "rollover": {
            "max_age": "7d"
          }
        }
      },
      "delete": {
        "min_age": "60d",
        "actions": {
          "delete": {}
        }
      }
    }
  }
}
````

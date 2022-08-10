# Overview
DataHub uses Elasticsearch heavily. Primarily to make upgrading DataHub versions as easy as possible, we've removed the Elasticsearch instance (along with the MySQL database) from DataHub's quickstart Docker container and host it in the cloud.

Elastic.co is a good choice for hosted Elasticseach.

**Note**: AWS does not provide a hosted Elasticsearch service. Their service, OpenSearch, is _not_ Elasticsearch compatible in a number of ways. DataHub does have special steps in their docs for using OpenSearch, but we don't cover them here.

We need to do main things:
1. Create an Elasticsearch instance somewhere in the clouds
2. Make two API calls to this instance, to create an index template and lifecycle policy

# Steps
1. In the elastic.co console, click "Create deployment"
2. Choose your provide (AWS, GCP, etc.) and region
3. **Important**: choose to edit the Advanced Settings. Mainly you'll want to look at the instance size; larger instances obviously cost more. A small instance with 2 GB of RAM, in a single availability zone, is working for us. That said, you'll want to consider the tradeoffs; only having one availability zone means that if that single instance goes down, your DataHub app will go down. And the more users and daily interaction you have on your app, the more RAM you'll want.
4. Once the instance has been created (elastic.co calls this a "deployment"), go to the Edit page of the deployment in elastic.co and check the monthly cost. Make sure you're ok with it!
5. Next, go to the Security tab of the deployment and get the username and password.
6. Now we need to make two API calls to this instance. A good tool to do this is Postman. For instructions, see the [elasticsearch-api1.md](elasticsearch-api1.md) file for details about the first API call, and the second in  [elasticsearch-api2.md](elasticsearch-api2.md)

That's it! Record the URL, username, and password of your Elasticsearch instance, we'll need them in future steps.

**Note**: the original policy and index setups can be found here: https://github.com/datahub-project/datahub/tree/master/metadata-service/restli-servlet-impl/src/main/resources/index/usage-event

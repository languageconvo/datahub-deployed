# Updating DataHub Versions

When DataHub releases a new version, you'll usually only have to follow these simple steps:

1. Wait for us to release a new version of this project, we'll try to be quick about it (or you can go to the /admin folder and follow the instructions, it's fairly simple)
2. Get the `/elasticbeanstalk/docker-compose.yml` file and replace the {{{ $vars }}} with the connection details of your MySQL database and Elasticsearch instance
3. Go to AWS Elastic Beanstalk and upload the new `docker-compose.yml` file. **Importantly**, choose the Immutable deployment option.
   1. Why the Immutable option? When you update DataHub versions, new Docker images are downloaded, some of which are quite large. If you choose the normal deployment option Elastic Beanstalk will attempt to deploy the new version of DataHub to  your existing EC2 instance. This will usually fail with an 'out of storage' error, because the EC2 instance that Elastic Beanstalk provisions doesn't have a ton of storage, and much of that storage is being used by your previous deployment (no, unfortunately Elastic Beanstalk doesn't automatically purge those old downloaded images). With an Immutable deployment, Elastic Beanstalk provisions an entirely new EC2 instance which will have plenty of storage available. Then it points your application to this new EC2 instance.
   2. Note that using Immutable deployments requires the Enhanced Health be turned on in Elastic Beanstalk. This does add some small bit of cost.

That's it! You should be able to reload your DataHub console and see the new version listed.

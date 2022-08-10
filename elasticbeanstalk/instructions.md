# In Progress!
These instructions are still in progress, we hope to finish them by mid-August. We're working on one last problem with password resets.

# Overview
MySQL stores DataHub's data, Elasticsearch makes it fast to get that data...what about the actual DataHub app? That's what we need to create in this last major step, the frontend DataHub app as well as a number of important backend services.  

Fortunately, we have a `docker-compose.yml` file we can use to help us do this. You can deploy that compose file to any cloud service you want. The first steps below will discuss editing that file so that it has the credentials for your MySQL and Elasticsearch instances. After it has those, you can take the compose file and deploy it anywhere you want, even on your own computer for local testing. Keep in mind though it connects to your production MySQL and Elasticsearch instances, so any changes you make in the DataHub app will actually change your production work.

After editing the docker-compose file, we're going to walk you through some steps to deploy to an AWS service called Elastic Beanstalk. It's a relatively easy way to get a full-fledged web application up in the cloud from just a docker-compose file.

# Steps
1. First, we need to edit the [docker-compose.yml](docker-compose.yml) file so that it has the details it needs to connect to the MySQL and Elasticsearch instances you created in previous steps
   1. Take a look at the [docker-compose.yml](docker-compose.yml) file and you'll find numerous `{{{ $varname }}}` These are things that you need to replace!
   2. Take a look at [vars-to-replace.md](vars-to-replace.md) for details on what each variable is, along with an example value. When you replace the variables, make sure you delete from the first { to the last }, nothing more, nothing less.
2. Create an Elastic Beanstalk application
   1. At this point, you can choose to deploy your compose file wherever you want, and you're all set. You should be able to visit a URL and see the DataHub app up and running! For example if you run the compose file on your local machine, you should be able to visit localhost and see the DataHub app running. You can log in with using the DataHub master username and password, then start using the app. We're going to walk you through deploying this docker-compose file to Elastic Beanstalk
   2. First, head to Elastic Beanstalk in the AWS console. Create an "Application"; this is just basically a folder, so the name isn't very intuitive. 
   3. Inside the application, click to create a new environment -- this is where and how our DataHub app really gets created. There are quite a few options, we'll try to go through the important ones, and add to this soon with more detail and screenshots:
      1. Web server environment (not worker)
      2. Platform = Docker
      3. Application code = Sample application (we'll upload our custom application after the initial startup succeeds, in a later step)
      4. Click on "Configure more options"
         1. Configuration presets = single instance (yes, this means your app will be hosted on one single EC2 instance; if it goes down, your DataHub app will go down. we haven't yet tested)
         2. Capacity - EC2 instance types = this is the important setting, it dictates how large or small of an EC2 instance is used to house your DataHub app. DataHub recommends having 8GB of RAM when running their Docker QuickStart app, so hopefully an instance with that much RAM will suffice considering we removed Elasticsearch and MySQL dependencies
         3. Monitoring = enable enhanced health reporting. This is needed in order to deploy new versions of DataHub using Elastic Beanstalk's immutable deploy option, which will be needed in most cases (more details explained in the [/updating.md](/updating.md) instructions in this repo)
   4. Once the environment has finished creating, the Elastic Beanstalk console will give you a URL that you can visit; you should see the sample application up and running!
   5. Now the magic part -- click the "Upload and deploy" button in the Elastic Beanstalk console, and upload the `docker-compose.yml` file. Once it finishes uploading, go to the URL that Elastic Beanstalk gave us and you should see DataHub running in the cloud. You should be able to log in with the DataHub master username (which is `datahub`) and the `{{{ $datahub_master_password }}}` password that you set in the docker-compose file! 
      1. Note that the URL is `http`, not `https`. Unfortuantely Elastic Beanstalk doesn't give us `https` by default for some reason, so we'll need to rectify that; sending web traffic over `http` isn't a good idea.
3. Set up SSL
   1. [ details coming soon! ]
4. Optional - set up a custom domain e.g. `myawesomedatadocs.com`
    1. [ details coming soon! ]

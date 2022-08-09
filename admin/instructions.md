# Overview

**Note**: if you're trying to get DataHub set up in production, you don't need to read this. This /admin folder describes the steps admins need to take to update and maintain this project.

When a new version of DataHub comes out, the admins of this project need to take a few steps. Those steps are described below in detail. If we have been slacking and you want to attempt to update to the new DataHub version before we're able to update this repository, you can go through these steps to determine how to update your project.

# Steps
1. In the /admin/datahub-versions folder, create a new folder named the same as DataHub's tag for their new version, e.g. `v0.8.42`
2. From the DataHub git repo, get `docker-compose-without-neo4j.quickstart.yml` from /docker/quickstart and paste it into the folder you created in step 1. This docker-compose file is what we use as a base for creating our own docker-compose file that's used for the Elastic Beanstalk application.
3. Read the new version's release notes. Fingers crossed that no changes need to be made to mysql or elasticsearch! If that's the case, the only thing that production deployments of DataHub will need to update is the docker container. If there *are* changes needed to mysql or elasticsearch, we should note that clearly in our own release notes of this repo. 
4. Check https://github.com/datahub-project/datahub/blob/master/docker/quickstart/docker-compose-without-neo4j.quickstart.yml to find the MySQL version that DataHub is using. It'll look like `image: mysql:5.7`. In our `/mysql/instructions.md` make sure that that version is listed.
5. Check https://github.com/datahub-project/datahub/tree/master/metadata-service/restli-servlet-impl/src/main/resources/index/usage-event to see if any recent updates have been made to either `policy.json` or `index_template.json`. If so, you may need to update /elasticsearch/ap1.md or api2.md
6. Check https://github.com/datahub-project/datahub/tree/efc5602493e66c83fa0ffe8cf9f9998fe9ec72bd/docker/mysql-setup `init.sql` to see if any recent updates have been made. If so, you may need to update /mysql/mysql.sql
7. Now build we need to build the new `docker-compose.yml` file that's used to create the Elastic Beanstalk application
    1. Diff the changes between the previous version and new version of `docker-compose-without-neo4j.quickstart.yml` In most cases the changes are very minor, and thus we can simply make a copy of `our-docker-compose.yml` file from the previous version, modify it to include the new changes, and voila we're done. If major changes have been made by DataHub, you'll need to consider whether to modify the previous `our-docker-compose.yml` file so that it contains these new changes, or to start with the new `docker-compose-without-neo4j.quickstart.yml` file supplied by DataHub and modify it to include our changes. Typically modifying `our-docker-compose.yml` will be easier, fewer steps.
    2. Once you're done (and you tested that it works, right?) put the resulting compose file in two locations: first, here in the appropriate /datahub-versions/ folder and name it `our-docker-compose.yml`. Second, in the /elasticbeanstalk folder and name it `docker-compose.yml`; that folder is where users of this repo get it from.
8. Create a PR in GitHub to `main`. Review
9. Try out the new version on our own cloud instance
10. If all goes well, merge the PR. Create a new release named the same version as DataHub.
    1. If any changes to MySQL or ElasticSearch were required, note them
    2. If any mistakes were made and a new version needs to be rolled, simply name it as `v0.8.2r2`, `v0.8.2r3` etc.

# Detailed Steps to Modifying `docker-compose-without-neo4j.quickstart.yml`
Starting with DataHub's `docker-compose-without-neo4j.quickstart.yml` which is designed as a local quickstart for folks new to DataHub to test out, we need to make a number of changes. (We chose this quickstart file over others as it doesn't include neo4j, a dependency which is not needed. It's the "cleanest" Docker setup.) As an overview:
- Remove everything related to mysql and elasticsearch, since in production we've moved those to dedicated services
- Create `{{{ $var }}}` variables so that end users of this repo know what they need to replace themselves

**Note:** take a look at the v0.8.41 folder for a few files that go step by step through the change process.

That said, the detailed steps:
1. Delete the following services: elasticsearch, elasticsearch-setup, mysql-setup, mysql and the following volumes: esdata, mysqldata
   1. These are all related to mysql and elasticsearch, which we use hosted services for
2. Make the following edits to the datahub-frontend-react service. This service, as you might guess, houses DataHub's front end.
   1. In the `JAVA_OPTS` var set the port to `-Dhttp.port=80`. This helps allow the frontend to be exposed on http in our Elastic Beanstalk application.
   2. Set `ELASTIC_CLIENT_HOST={{{ $elastic_host }}}` 
   3. Set `ELASTIC_CLIENT_PORT=443` for SSL
   4. Below that add `ELASTIC_CLIENT_USE_SSL=true` for SSL
   5. Add `ELASTIC_CLIENT_USERNAME={{{ $elastic_username }}}` and `ELASTIC_CLIENT_PASSWORD={{{ $elastic_password }}}` so that the frontend knows how to connect to the elasticsearch instance
   6. Set `image: linkedin/datahub-frontend-react:v0.8.41` making sure the version is the DataHub version you're working on. Here we're obviously hardcoding the version so that each version of this repo is always tied directly to a specific DataHub version
   7. Set `ports` to `- 80:80` so that this service can be exposed to http on Elastic Beanstalk
   8. Add one volume: `- frontend:/datahub-frontend/conf/`. This allows our custom `datahub-frontend-react-setup` service to share a volume with this service, to transfer a file to it (the user.props file explained later)
3. Make the following edits to the datahub-gms service (DataHub's "generalized metadata service).
   1. Remove the `depends_on` mysql, since we move that to an external service
   2. Set `DATAHUB_TELEMETRY_ENABLED=true`
   3. Set `EBEAN_DATASOURCE_USERNAME={{{ $mysql_username }}}`, `EBEAN_DATASOURCE_PASSWORD={{{ $mysql_password }}}`, `EBEAN_DATASOURCE_HOST={{{ $mysql_host }}}:3306`
   4. In the `EBEAN_DATASOURCE_URL` replace `//mysql` with `//{{{ $mysql_host }}}`
   5. Set `ELASTICSEARCH_HOST={{{ $elastic_host }}}`
   6. Set `ELASTICSEARCH_PORT=443` for SSL
   7. Set `image: linkedin/datahub-gms:v0.8.41` ensuring you get the correct version
   8. Add the following three lines, needed so the gms service knows how to connect with elasticsearch: `- ELASTICSEARCH_USE_SSL=true`, `ELASTICSEARCH_USERNAME={{{ $elastic_username }}}`, `ELASTICSEARCH_PASSWORD={{{ $elastic_password }}}`
4. In the kafka-setup servie set `image: linkedin/datahub-kafka-setup:v0.8.41` ensuring you get the correct version
5. Add our custom datahub-frontend-react-setup service
   1. What is this service for? The only reason this service exists is to set the datahub frontend master user's password. For some reason, this can only be done by modifiying a `user.props` file in the datahub-frontend-react container. In order to do that, the best solution we could come up with was generating this `user.props` file in its own container, then using a shared volume (named frontend) to get the file into the datahub-frontend-react container. This setup container will automatically stop after 180 seconds.
6. Add a volume `frontend:`. This is what allows us to transfer the `user.props` file from the datahub-frontend-react-setup container to the datahub-frontend-react

# Detailed steps to modifying `mysql.sql`

For this file, there were just two simple steps:
1. Took the `init.sql` file from DataHub's git repo, which is located in the /docker/mysql-setup folder, and pasted the contents into our `/mysql/mysql.sql` file
2. Within the sql file replaced `DATAHUB_DB_NAME` with `datahub`



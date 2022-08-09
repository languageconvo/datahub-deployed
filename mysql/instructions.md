# Overview
DataHub utilizes a MySQL database as its primary data store. That means, for example, if you create a new Glossary Term in DataHub, the title, description, etc. are all stored in a MySQL database.

**Important**: if in the future your MySQL database gets deleted...that means all of your data that was in DataHub is gone. All of the glossary terms, descriptions of tables, fields, BI dashboards, they'll all be gone forever. So make backups of this database! We'll discuss this more below.

To get MySQL set up, there will be two main steps:
1. Create a MySQL database in a cloud service
2. Run the SQL from our `mysql.sql` file in that database to set up the tables that DataHub needs

If you've never set up a database in the cloud before, we'd recommend consulting someone with devops experience. The following are *general* steps that can help you do this; we'll reference AWS RDS, a managed cloud database service, but we'll write generally for any cloud service you want to use.

# Steps
1. Create a MySQL database in the cloud, version **5.7**
   1. Verify MySQL version. We'll hopefully remember to update ^ that when DataHub does, but if you want to be sure it' s the version that DataHub is using visit https://github.com/datahub-project/datahub/blob/master/docker/quickstart/docker-compose-without-neo4j.quickstart.yml and look for the `image:mysql` line of code 
   2. Username and password. During setup you'll likely be asked to set a master username and password. Record these, you'll need them to connect to the database.
   3. Size. For small-ish use cases, we've found AWS's `t3.medium` to work fine so far. It has 4GB of RAM and 2 vCPUs.
   4. Storage. 100GB is the recommended minimum for production use cases on AWS. If you expect to have a lot of DataHub data you might increase this.
   5. Automated backups. AWS allows you to set up automated backups be retained for up to 35 days; we'd recommend you set it to that. Note there is a cost to the backup storage.
   6. The /screenshots/mysqlconfig.png file shows a typical RDS configuration with the above settings
2. Special step for AWS RDS - create a Security Group
   1. With AWS RDS, a security group is required to manage access to your database. In this security group, you'll need to set an "ingress" rule to allow your own IP address. (We'll also need this security group in the near future, to allow our Elastic Beanstalk app to access the database)
   2. Attach this security group to the RDS instance. See the /screenshots/rdsbasic.png for an example of what you're aiming for, the "VPC security groups" section
3. Now we need to run some SQL on this database, to set up the tables and other things DataHub needs.
   1. Connect to the MySQL instance you created above. You can use an IDE like DataGrip or MySQL Workbench to connect; you'll need the database URL, username and password.
   2. Get the contents from the `mysql.sql` file that's in this folder, and run it on your database. Make sure all of it runs (some IDEs only run the first command in a SQL file by default)! When that's done you should see the tables as shown in /screenshots/tables.png

# Backups
As noted above, RDS can make automated backups of your MySQL database, but they can only be retained for around a month. We highly recommend looking at making longer-term backups if your DataHub data is at all important to you. We'd recommend taking a look at a service simplebackups.com; as the name implies, setting up an ongoing daily backup is pretty easy. We're not affiliated with them in any way, and of course there are many other options out there for easily backing up a MySQL database.

A once-daily backup, and retaining that backup for a very long time, might be a good starting point. This way, in the event someone on your team accidentally deletes an important piece of information in DataHub, you can go back and restore it.

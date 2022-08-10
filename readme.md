# Deploy DataHub!
[DataHub](https://datahubproject.io/) is an awesome metadata platform, a tool to document, discover, share, and collaborate on your data. It's open-source, and progressing rapidly.

This project attempts to make it *somewhat* easy for small teams to deploy a production instance of DataHub to the cloud. If you have some devops experience, you can probably get through this in an hour or two. If you don't have devops experience it'll take longer, but our aim is to make the steps here clear enough that anyone can use them.

# Quickstart
1. Read through this readme
2. Follow `instructions.md` in the /mysql folder to set up a MySQL database
3. Follow `instructions.md` in the /elasticsearch folder to set up an Elasticsearch instance
4. Follow `instructions.md` in the /elasticbeanstalk folder to get your DataHub app up and running the cloud, ready for your team to use!

# You Should Probably Use Acryl Data
[Acryl Data](https://www.acryldata.io/) is the team behind DataHub, and they're building a managed solution. Once it's available, you should probably use it if your organization can afford to. Would you rather spend your time building your own product, or managing the infrastructure behind your team's DataHub setup? Here's a fingers-crossed hope that Acryl is able to put together usage-based pricing that will be affordable for teams of any size! An open-source project of this magnitude often does better if there is a team behind it that is paid to think and work on it every day, so support that team financially if you can. 

**Who might want to use this project for a production DataHub setup?**

That said, this might be a good option for:
- Small teams, you have one or two "data people" and not a ton of data
- Academia, non-profits, etc.

# Infrastructure Overview

There are three main pieces of infrastructure you'll be creating:

1. A managed MySQL instance [we chose AWS RDS, but any will suffice]
2. A managed Elasticsearch instance [we chose elastic.co]
3. Everything else (DataHub frontend, Kafka, etc.) on a single instance via docker-compose [we chose AWS Elastic Beanstalk]

**Why?**

Our goals were:
- Make it as easy as possible to deploy DataHub to the cloud for a small production use case
- Make it as easy as possible to update DataHub when a new version comes out
- Ensure we don't lose our DataHub data
- Make it as affordable as possible

By separating MySQL and Elasticsearch, we can usually update DataHub by simply deploying a new `docker-compose.yml` file that has the newest version of DataHub listed. Having MySQL in its own separate, managed service means that backing up our data is easy and reliable. Having the bulk of the application in a single instance means affordability.

Obviously, having much of the bulk of DataHub in a single instance won't be performant past a certain point. So again, this is probably only a good idea for small teams. Larger teams should look to Kubernetes, until Acryl Data has their managed product available.

**How?**

We started with DataHub's Docker quickstart `docker-compose-without-neo4j.yml`, removed MySQL and Elasticsearch, set those two up as separate managed services leaving the rest of DataHub's front and back end in the docker-compose.

# Approximate Costs

We haven't done enough testing yet to determine how stable this setup is going to be, meaning that larger instance sizes may be needed. You may be able to get away with smaller instances too, we'll just have to test and find out (let us know if you do). The costs below **don't include everything**, for example doesn't include database backup storage; these are just the primary costs of the 3 main pieces of infrastructure. We're in the us-west region, other regions have different costs. You should look carefully into costs if you're considering using this as a guide.

That said, for our small team so far the following has worked:

- AWS RDS t3.medium = $50/month
- Elastic.co 2GB instance = $41/month
- AWS Elastic Beanstalk t3.large = $60/month
- AWS NAT Gateway = $35/month

# Updating DataHub Versions

When DataHub releases a new version, updating your instance to the new version only takes a few simple steps, typically less than 5 minutes of work. See `updating.md` for the step-by-step guide.

# Problems & Things to Consider

- The current setup steps do not include how to set up a static outbound IP address with Elastic Beanstalk. That means on your data sources, you would have to allow *all* IP addresses access. Obviously that's not a good idea. Setting up a static outbound IP address is not very difficult, see https://aws.amazon.com/premiumsupport/knowledge-center/elastic-beanstalk-static-IP-address/ but there is an additional cost for the NAT gateway, and our own instructions don't yet include how to set this up.

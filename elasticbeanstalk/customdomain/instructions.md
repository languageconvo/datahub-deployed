## Overview
The URL you get from Elastic Beanstalk will look like `datahub-mycompany.us-west-2.elasticbeanstalk.com`  ... not exactly the easiest to remember. Follow these steps to set up a custom domain such as `mycompanydata.com` to host your DataHub application.

## Steps
**Buy/Register your new domain**

First we need to buy the domain name you want. We'll use "myawesomedata.com" as an example  throughout these steps.

1. In AWS go to Route53 -> Registered domains, and click the button "Register Domain". Follow the steps to purchase your new domain name, "myawesomedata.com". Exciting stuff!
2. Check your email, AWS will email you a confirmation that you need to complete. After doing that, check your AWS account; in Route53, wait until you see your new domain name listed on the "Registered domains" page. Once it's there, click on the domain and then set the "Transfer lock" option to "Enabled"
    1. Why? This will make it impossible to accidentally (or otherwise) transfer the domain to some other domain host. If you do need to transfer the domain in the future, you'll need to first disable this setting.

**Create an SSL certificate**

So that we can visit your DataHub website securely using HTTPS, we need to set up something called an SSL certificate.

1. In AWS, go to Certificate Manager. Click Request. In the "Fully qualified domain name" box, type your domain name "myawesomedata.com"  (without the quotation marks!) We'd also recommend adding another name to the certificate, "*.myawesomedata.com"  (again without those quotation marks). For the validation method option, leave "DNS validation - recommended" selected.
     1. Why? The * asterisk means "any subdomain on myawesomedata.com can be considered to be secure". So if in future you set up something like "api.myawesomedata.com" this SSL certificate will apply to it.
2. In a new tab, go to Route53. In the "Hosted zones" section you'll see a zone that has your domain name "myawesomedata.com"; click on it. We need to create one CNAME record:
    1. Under "Record type" choose "CNAME"
    2. Back in the AWS Certificate Manager, on the SSL certificate page, you'll see 2 domains. Look for the CNAME name and CNAME value. It shouldn't matter which one you get, but get the name and value and put those into the name and value boxes for the CNAME record
    3. After the record is created you should see it on the Hosted zones page. The "Record name" should be like this: `_e25s1e1et.myawesomedata.com`, the Type should be CNAME, and the "Value/Route traffic to" should look like `_abc123.aslke3jf.acm-validations.aws`
    4. Refresh the AWS Certificate Manager page and it should show that your certificate has a green checkmark and says Issued. Congrats, you have an SSL certificate for your new domain!
3. Next, head to Route53 Hosted zones, and click on your zone. Click "Create record" and make the following choices:
    1. First, select Alias
    2. In the "Route traffic to" option, chose "Alieas to Elastic Beanstalk environment", and chose the region you created the Elastic Beanstalk app in
    3. Then choose the environment, and create the record

Note: at this point you should be able to go to your custom domain "myawesomedata.com" and see DataHub! But you'll also notice it's insecure, HTTP, so let's rectify that.

4. 


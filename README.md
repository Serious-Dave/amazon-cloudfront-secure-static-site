# Amazon CloudFront Secure Static Website

Use this solution to create a secure static website for your registered domain name. With this solution, your website:

- Is hosted on [Amazon S3](https://aws.amazon.com/s3/)
- Is distributed by [Amazon CloudFront](https://aws.amazon.com/cloudfront/)
- Uses an SSL/TLS certificate from [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/)
- Uses [CloudFront Response Header Policies](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-response-headers.html) to add security headers to every server response
- Is deployed with [AWS CloudFormation](https://aws.amazon.com/cloudformation/)

For more information about each of these components, see the **Solution details** section on this page.

## Solution overview

The following is an overview of how the solution works:

1. The viewer requests the website at www.example.com.
2. If the requested object is cached, CloudFront returns the object from its cache to the viewer.
3. If the object is not in CloudFront’s cache, CloudFront requests the object from the origin (an S3 bucket).
4. S3 returns the object to CloudFront
5. CloudFront caches the object.
6. The object is returned to the viewer. Subsequent responses for the object are served from the CloudFront cache.

## Solution details

### S3 configuration

This solution creates an S3 bucket that hosts your static website’s assets. The website is only accessible via CloudFront, not directly from S3.

### CloudFront configuration

This solution creates a CloudFront distribution to serve your website to viewers. The distribution is configured with a CloudFront [origin access identity](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/private-content-restricting-access-to-s3.html) to make sure that the website is only accessible via CloudFront, not directly from S3. The distribution is also configured with a [CloudFront Response Header Policy](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/adding-response-headers.html) that adds security headers to every response.

### ACM configuration

This solution creates an SSL/TLS certificate in ACM, and attaches it to the CloudFront distribution. This enables the distribution to serve your domain’s website using HTTPS.

### CloudFront Response Header Policy

The CloudFront Response Header Policy adds security headers to every response served by CloudFront.

The security headers can help mitigate some attacks, as explained in the [Amazon CloudFront - Understanding response header policies documentation](https://docs.aws.amazon.com/AmazonCloudFront/latest/DeveloperGuide/understanding-response-headers-policies.html#understanding-response-headers-policies-security). Security headers are a group of headers in the web server response that tell web browsers to take extra security precautions. This solution adds the following headers to each response:

- [Strict-Transport-Security](https://infosec.mozilla.org/guidelines/web_security#http-strict-transport-security)
- [Content-Security-Policy](https://infosec.mozilla.org/guidelines/web_security#content-security-policy)
- [X-Content-Type-Options](https://infosec.mozilla.org/guidelines/web_security#x-content-type-options)
- [X-Frame-Options](https://infosec.mozilla.org/guidelines/web_security#x-frame-options)
- [X-XSS-Protection](https://infosec.mozilla.org/guidelines/web_security#x-xss-protection)
- [Referrer-Policy](https://infosec.mozilla.org/guidelines/web_security#referrer-policy)

For more information, see [Mozilla’s web security guidelines](https://infosec.mozilla.org/guidelines/web_security).

## Prerequisites

You must have a registered domain name, such as example.com, and point it to a Route 53 hosted zone in the same AWS account in which you deploy this solution. For more information, see [Configuring Amazon Route 53 as your DNS service](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/dns-configuring.html).

## Deploy the solution

> :⚠️ This template can only be deployed in the `us-east-1` region

To deploy the solution, you use [AWS CloudFormation](https://aws.amazon.com/cloudformation).

> **Note:** You must have IAM permissions to launch CloudFormation templates that create IAM roles, and to create all the AWS resources in the solution. Also, you are responsible for the cost of the AWS services used while running this solution. For more information about costs, see the pricing pages for each AWS service.

## Customizing the Solution

### Update the website content locally

**To customize the website with your own content before deploying the solution**

1. Install npm. For more information, go to https://www.npmjs.com/get-npm.
2. Clone or download this project from https://github.com/awslabs/aws-cloudformation-templates.
3. Run the following command to package a build artifact.

   ```shell
   make package-static
   ```

4. Copy your website content into the **www** folder.
5. If you don’t have one already, create an S3 bucket to store the CloudFormation artifacts. To create one, use the following AWS CLI command:

   ```shell
   aws s3 mb s3://<S3 bucket name>
   ```

6. Run the following AWS CLI command to package the CloudFormation template. The template uses the [AWS Serverless Application Model](https://aws.amazon.com/about-aws/whats-new/2016/11/introducing-the-aws-serverless-application-model/), so it must be transformed before you can deploy it.

   ```shell
   aws --region us-east-1 cloudformation package \
       --template-file templates/main.yaml \
       --s3-bucket <your S3 bucket name> \
       --output-template-file packaged.template
   ```

7. Run the following command to deploy the packaged CloudFormation template to a CloudFormation stack. To optionally deploy the stack with a domain apex skip this section and proceed to [Step 8] below.

   ```shell
   aws --region us-east-1 cloudformation deploy \
       --stack-name <your CloudFormation stack name> \
       --template-file packaged.template \
       --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
       --parameter-overrides  DomainName=<your domain name> SubDomain=<your website subdomain> HostedZoneId=<hosted zone id>
   ```

8. [Optional] Run the following command to deploy the packaged CloudFormation template to a CloudFormation stack with a domain apex.

   ```shell
   aws --region us-east-1 cloudformation deploy \
       --stack-name <your CloudFormation stack name> \
       --template-file packaged.template \
       --capabilities CAPABILITY_NAMED_IAM CAPABILITY_AUTO_EXPAND \
       --parameter-overrides  DomainName=<your domain name> SubDomain=<your website subdomain> HostedZoneId=<hosted zone id> CreateApex=yes
   ```

### Updating the site Content Security Policy

To change the Content Security Policy of the site:

1. Make your changes to the header values by editing `source/secured-headers/index.js`.
1. Deploy the solution by following the steps in [Update the website content locally](#update-the-website-content-locally)

## License Summary

This project is licensed under the Apache-2.0 License.

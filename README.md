# tafalk-basic-web-app-cloudformation-template

The CloudFormation template for the AWS resources of tafalk-basic-web-app


It has a [main stack](main.yaml) and the rest are nested stacks which are:
1- [IAM Layer](iam.yaml) for IAM roles and policies
2- [Network Layer](network.yaml) for the VPC, subnet, security groups and NAT instance. The Default NAT instance AMI name is available for Frankfurt(eu-central-1) region.
3- [Data layer](data.yaml) mainly for DynamoDB resources
4- [CICD Pipeline for Lambda functions](build.yaml) for build pipelines of the lambda functions
5- [Application Authentication Layer](cognito-auth.yaml) for managing application users (i.e. signin, login)
6- [AppSync configuration](appsync.yaml) for Appsync API and schemas
7- [Serving](webserving.yaml) for hosting S3 buckets, CloudFormation and Route53 hosted zones and record sets.

## Prework

The user is expected to supply some parameters and some of those parameters requires a pre-definition:

1- Grab an [AWS account](https://aws.amazon.com/) if you do not have one.
2- Get a [Google Places API key](https://developers.google.com/places/web-service/get-api-key)
3- Register your site for [Google Recaptcha V3](https://www.google.com/recaptcha/admin/create)
4- Create an EC2 key-pair for assigning to the NAT instance.
5- Create a AWS CodeCommit repository for the Lambda function triggered when a user confirms his/her authentication. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/cognito-user-exporter-function/browse?region=eu-central-1)
6- Create a AWS CodeCommit repository for the Lambda function used for resolving place keys of Google Places API. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/place-getter-function/browse?region=eu-central-1)
7- Create a AWS CodeCommit repository for the Lambda function used for getting Google Recaptcha V3 token. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/recaptcha-token-resolver-function/browse?region=eu-central-1)
8- Create a AWS CodeCommit repository for the Lambda function for achieving complex queries for resources. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/resource-searcher-function/browse?region=eu-central-1)
9 - Get a domain for your website. [Amazon Route53](https://aws.amazon.com/getting-started/tutorials/get-a-domain/) can be used to do that
10- Get a site-specific email address from ZohoMail. [CNAME for mail verification](https://www.zoho.com/mail/help/adminconsole/domain-verification.html#alink4) is also needed in parameters
11- Create an S3 bucket and copy all the `yaml` files and the graphql folder (the bucket also should have a `graphql` folder) within this repository. We are going to address that location for stack creation.

You should also add ACM (AWS Certificate Manager) certificates as Route53. They are requested in the templates but, adding the created certs to Route53 cannot be done automatically now: [Forum post 1](https://forums.aws.amazon.com/thread.jspa?messageID=821952), [Forum post 2](https://forums.aws.amazon.com/ann.jspa?annID=6076)

## Creating the stack

### Console

### AWS-CLI

## See also

- [CloudFormation editing on VSCode](https://hodgkins.io/up-your-cloudformation-game-with-vscode)

- [Building a Continuous Delivery Pipeline for a Lambda Application with AWS CodePipeline](https://docs.aws.amazon.com/lambda/latest/dg/build-pipeline.html)

- [Lambda Build Pipeline sample](https://github.com/widdix/aws-velocity/blob/master/deploy/pipeline.yml)

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
6- Create a AWS CodeCommit repository for the Lambda function used for getting Google Recaptcha V3 token. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/recaptcha-token-resolver-function/browse?region=eu-central-1)
7- Create a AWS CodeCommit repository for the Lambda function for achieving complex queries for resources. A sample is [here](https://eu-central-1.console.aws.amazon.com/codesuite/codecommit/repositories/resource-searcher-function/browse?region=eu-central-1)
8 - Get a domain for your website. [Amazon Route53](https://aws.amazon.com/getting-started/tutorials/get-a-domain/) can be used to do that
9- Get a site-specific email address from ZohoMail. [CNAME for mail verification](https://www.zoho.com/mail/help/adminconsole/domain-verification.html#alink4) is also needed in parameters
10- Create an S3 bucket and copy all the `yaml` files and the graphql folder (the bucket also should have a `graphql` folder) within this repository. We are going to address that location for stack creation.

## Creating the stack

### Console

- Sign in to AWS Console.
  
- Go to CloudFormation page and click **Create Stack** button.

  - Choose **Template is ready** option for *Prerequisite - Prepare template*

  - Choose **Amazon S3** for *Template Source*.

- We are going to enter the HTTP URL of the `main.yaml` file we have uploaded among the other ones. It should look like:

```url
https://s3.{{aws-region}}.amazonaws.com/{{bucket-name-of-the-templates}}/main.yaml
```

- Click next and enter parameters for your case.

- Next, next, next...

### AWS-CLI

I personally favor CLI usage whenever possible, but creating this stack require a dozen of parameters and managing them with a command is not always as clear as the console screen. Still, this command should work:

```sh
aws cloudformation create-stack \
  --stack-name {{your-stack-name}} \
  --template-body https://s3.{{aws-region}}.amazonaws.com/{{bucket-name-of-the-templates}}/main.yaml \
  --parameters \
    ParameterKey=KeyPairName,ParameterValue=TestKey \
    ParameterKey=SubnetIDs,ParameterValue=SubnetID1\\,SubnetID2
```

### Manual work

#### Registered Domain Nameservers

The created Route 53 hosted zone may arbitrarily create the name servers (NS). And this may lead to problems in certificate validation.

Not to encounter such cases, from the AWS Console, you have to update *Registered Domain name servers*:

- Sign in to the [console](https://console.aws.amazon.com) and go to **Route 53** service.

- From the left pane, click *Hosted Zones*.

- In the next window, click on your domain. Note down the value for the *record set* with type `NS` -typically four name servers. 

- From the left pane, click *Registered Domains*.

- In the next window, click on your domain.

- Click **Add or edit name servers**.

- Enter the hosted zone name servers you have previously noted down from scratch.

- An email will be received telling the registered domain name servers are updated.

See the [docs](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html#migrate-dns-update-domain) for further information.

#### Certificate Validation

This template features a SSL certificate creation for [AWS Certificate Manager (ACM)](https://aws.amazon.com/certificate-manager/). However, validating that certificate blocks the Hosted Zone and CDN creations which can be investigated in [webserving.yaml](webserving.yaml) file and cannot be completed unless a corresponding CNAME record is added to the Route53 Hosted Zone.

To the date, AWS do not supply an automated way for this.

> There are custom solutions around the net (like [binxio's solution](https://github.com/binxio/cfn-certificate-provider)); however, they are not implemented in this repo for the sake of simplicity.

To manually do that after stack creation is initiated, check the AWS Certificate Manager page in AWS console.

When an entry appears there, under *Validation* tab, click **Create Record in Route 53** (for a more pictured guide, see [here](https://aws.amazon.com/blogs/security/easier-certificate-validation-using-dns-with-aws-certificate-manager/)) and leave the stack creation in rest until validation is done and other steps are ready to continue.

> The newly created hosted zone should be ready for the certificate to be verified.
> To check it, run:
> `dig tafalk.com`
> And it should contain a `;; ANSWER SECTION:`

## See also

- [CloudFormation editing on VSCode](https://hodgkins.io/up-your-cloudformation-game-with-vscode)

- [Building a Continuous Delivery Pipeline for a Lambda Application with AWS CodePipeline](https://docs.aws.amazon.com/lambda/latest/dg/build-pipeline.html)

- [Lambda Build Pipeline sample](https://github.com/widdix/aws-velocity/blob/master/deploy/pipeline.yml)

# tafalk-basic-web-app-cloudformation-template

The CloudFormation template for the AWS resources of tafalk-basic-web-app

It has a [main stack](main.yaml) and the rest are nested stacks which are:

1. [IAM Layer](iam.yaml) for IAM roles and policies

2. [Network Layer](network.yaml) for the VPC, subnet, security groups and NAT instance. The Default NAT instance AMI name is available for Dublin(eu-westr-1) region.

3. [Data layer](data.yaml) mainly for DynamoDB resources

4. [CICD Pipeline for Lambda functions](build.yaml) for build pipelines of the lambda functions

5. [Application Authentication Layer](cognito-auth.yaml) for managing application users (i.e. signin, login)

6. [AppSync configuration](appsync.yaml) for Appsync API and schemas

7. [RDS Bootstrapping](rds-bootstrap.yaml) for creating RDS tables

8. [Serving](webserving.yaml) for hosting S3 buckets, CloudFormation and Route53 hosted zones and record sets.

## Prework

The user is expected to supply some parameters and some of those parameters requires a pre-definition:

1. Grab an [AWS account](https://aws.amazon.com/) if you do not have one.

2. Get a [Google Places API key](https://developers.google.com/places/web-service/get-api-key)

3. Register your site for [Google Recaptcha V3](https://www.google.com/recaptcha/admin/create)

4. Create an EC2 key-pair for assigning to the NAT instance.

5. [Create](https://help.github.com/en/github/authenticating-to-github/creating-a-personal-access-token-for-the-command-line#creating-a-token) and note down a Github personal access token: we will use it for `GitHubOAuthToken`. The token should have scopes:

   - repo
   - repo:status
   - admin:repo_hook

6. Create:

   1. A GitHub repository for the Lambda function triggered when a user confirms xer authentication. A sample is [here](https://github.com/tafalk/cognito-user-exporter-function/)

   2. A GitHub repository for the Lambda function used for getting Google Recaptcha V3 token. A sample is [here](https://github.com/tafalk/recaptcha-token-resolver-function/)

   3. A GitHub repository for the Lambda function for achieving complex queries for resources. A sample is [here](https://github.com/tafalk/resource-searcher-function/)

   4. A GitHub repository for the static html files for keeping privacy policy and terms of use html files. A sample is [here](https://github.com/tafalk/site-policies/)

   5. A GitHub repository for creating RDS tables. A sample is [here](https://github.com/tafalk/rds-bootstrap-function/)

   6. A GitHub repository for handling certificate validation. A sample is [here](https://github.com/tafalk/certificate-delegating-function/)

7. Get a domain for your website. [Amazon Route53](https://aws.amazon.com/getting-started/tutorials/get-a-domain/) can be used to do that

8. Get a site-specific email address from ZohoMail. [CNAME for mail verification](https://www.zoho.com/mail/help/adminconsole/domain-verification.html#alink4) is also needed in parameters

9. Create an S3 bucket and copy all the `yaml` files and the graphql folder (the bucket also should have a `graphql` folder) within this repository. We are going to address that location for stack creation.

## Creating the stack

### Console

- Sign in to AWS Console.

- Go to CloudFormation page and click **Create Stack** button.

  - Choose **Template is ready** option for _Prerequisite - Prepare template_

  - Choose **Amazon S3** for _Template Source_.

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

#### Testing validity of templates

In project root, run:

```sh
aws cloudformation validate-template --template-body file://{yaml_file_name}
```

### Manual work

#### Creating RDS resources

Manually run `tafalk-prod-rdsbootstrap` with empty json object (`{}`) as body.

This will create UncloggerPrompt and Flag tables

#### Registered Domain Nameservers

The created Route 53 hosted zone may arbitrarily create the name servers (NS). And this may lead to problems in certificate validation.

Not to encounter such cases, from the AWS Console, you have to update _Registered Domain name servers_:

- Sign in to the [console](https://console.aws.amazon.com) and go to **Route 53** service.

- From the left pane, click _Hosted Zones_.

- In the next window, click on your domain. **Note down** the value for the _record set_ with type `NS` -typically four name servers.

- From the left pane, click _Registered Domains_.

- In the next window, click on your domain.

- Click **Add or edit name servers**.

- Enter the hosted zone name servers you have previously noted down from scratch.

- An email will be received telling the registered domain name servers are updated.

See the [docs](https://docs.aws.amazon.com/Route53/latest/DeveloperGuide/migrate-dns-domain-in-use.html#migrate-dns-update-domain) for further information.

#### [Not anymore] ACM certificate validation

Thanks to rhboyd's great [Custom Resource ACM](https://github.com/rhboyd/CustomResourceACM/) certificae validation (i.e. adding CNAME to domain) can be done automatically now.

## See also

- [CloudFormation editing on VSCode](https://hodgkins.io/up-your-cloudformation-game-with-vscode)

- [Building a Continuous Delivery Pipeline for a Lambda Application with AWS CodePipeline](https://docs.aws.amazon.com/lambda/latest/dg/build-pipeline.html)

- [Lambda Build Pipeline sample](https://github.com/widdix/aws-velocity/blob/master/deploy/pipeline.yml)

- [Custom Resource ACM](https://github.com/rhboyd/CustomResourceACM/)

# tafalk-basic-web-app-cloudformation-template
The CloudFormation template for the AWS resources of tafalk-basic-web-app

Create the stacks **strictly** with the order
 1- [IAM Layer](#iam-layer)
 2- [Network Layer](#network-layer)
 2- [Cognito Layer](#cognito-layer)

## IAM Layer
`iam.yaml` contains specs for IAM Roles

Run the following to craete a stack of its:

```sh
aws cloudformation create-stack --stack-name ${iamStackName} \
--template-body file://./iam.yaml \
--parameters ProjectIdentifierName=${projectName},EnvironmentName=${enviromentName}
```

## Network Layer
`network.yaml` contains specs for VPC, Subnet etc.

Run the following to craete a stack of its:

```sh
aws cloudformation create-stack --stack-name ${networkStackName} \
--template-body file://./network.yaml \
--parameters ProjectIdentifierName=${projectName},EnvironmentName=${enviromentName}
```

# See also
- [CloudFormation editing on VSCode](https://hodgkins.io/up-your-cloudformation-game-with-vscode)
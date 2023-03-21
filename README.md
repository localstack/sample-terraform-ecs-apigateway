# Serverless Container-based APIs with Amazon ECS and Amazon API Gateway

| Key          | Value                                                                                 |
| ------------ | ------------------------------------------------------------------------------------- |
| Environment  | LocalStack, AWS                                                                       |
| Services     | S3, DynamoDB, Elastic Container Service, API Gateway, Cognito, IAM                    |
| Integrations | Terraform, CloudFormation, AWS CLI                                                    |
| Categories   | Serverless; Containers; Security, Identity, and Compliance                            |
| Level        | Intermediate                                                                          |
| GitHub       | [Repository link](https://github.com/localstack/serverless-api-ecs-apigateway-sample) |

## Introduction

The Serverless Container-based APIs with Amazon ECS and Amazon API Gateway application sample demonstrate how you can launch and test a sample container-based API. This application sample implements an example API with two services — “Food store” to `PUT` & `GET` foods, and “Pet store” to `PUT` & `GET` pets. The application client is implemented using ReactJS, which allows unauthenticated users to use only `GET` requests, while authenticated users can utilize `GET` and `PUT` requests. Users can deploy this application sample on AWS & LocalStack using CloudFormation & Terraform with minimal changes. To test this application sample, we will demonstrate how you use LocalStack to deploy the infrastructure on your developer machine and your CI environment.

## Architecture diagram

The following diagram shows the architecture that this sample application builds and deploys:

![Architecture diagram for Serverless Container-based APIs with Amazon ECS and Amazon API Gateway sample application](./images/serverless-container-api.png)

We are using the following AWS services and their features to build our infrastructure:

- [Elastic Container Service](https://docs.localstack.cloud/user-guide/aws/elastic-container-service/) to create and deploy our containerized application.
- [DynamoDB](https://docs.localstack.cloud/user-guide/aws/dynamodb/) as a key-value and document database to persist our data.
- [API Gateway](https://docs.localstack.cloud/user-guide/aws/apigatewayv2/) to expose the containerized services to the user through HTTP APIs.
- [Cognito User Pools](https://docs.localstack.cloud/user-guide/aws/cognito/) for user authentication and authorizing requests to container APIs.
- [Amplify](https://docs.localstack.cloud/user-guide/aws/amplify/) to create the user client with ReactJS to send requests to container APIs.
- [S3](https://docs.localstack.cloud/user-guide/aws/s3/) to deploy the Amplify application to make the web application available to users.
- [IAM](https://docs.localstack.cloud/user-guide/aws/iam/) to create policies to specify roles and permissions for various AWS services.

## Prerequisites

- LocalStack Pro
- [AWS CLI](https://docs.localstack.cloud/user-guide/integrations/aws-cli/) with the `awslocal` wrapper.
- [Terraform](https://docs.localstack.cloud/user-guide/integrations/terraform/) with the `tflocal` wrapper.
- [Node.js](https://nodejs.org/en/download/) with `npm` package manager.

Start LocalStack Pro with the appropriate configuration to enable the S3 website to send requests to the container APIs:

```shell
export LOCALSTACK_API_KEY=<your-api-key>
EXTRA_CORS_ALLOWED_ORIGINS=http://sample-app.s3.localhost.localstack.cloud:4566 DISABLE_CUSTOM_CORS_APIGATEWAY=1 localstack start -d
```

The `DISABLE_CUSTOM_CORS_APIGATEWAY` configuration variable disables CORS override by API Gateway. The `EXTRA_CORS_ALLOWED_ORIGINS` configuration variable allows our website to send requests to the container APIs.

## Instructions

You can build and deploy the sample application on LocalStack by running our `Makefile` commands. Run `make terraform-setup` or `make cloudformation-setup` to create the infrastructure on LocalStack. Run `make run` to deploy the S3 Website and get a URL to access the application. Run `make stop` to delete the infrastructure by stopping LocalStack.

Here are instructions to deploy it manually step-by-step.

## Creating the infrastructure

To create the AWS infrastructure locally, you can either use Terraform or CloudFormation.

### Terraform

To create the infrastructure using Terraform, run the following commands:

```shell
cd terraform
tflocal init
tflocal apply --auto-approve
```

We are using the `tflocal` wrapper to configure the local service endpoints, and send the API requests to LocalStack, instead of AWS. You can use the same Terraform configuration to deploy the infrastructure on AWS as well.

#### CloudFormation

To create the infrastructure using CloudFormation, run the following commands:

```shell
cd cloudformation
export STACK="stack1"
awslocal cloudformation create-stack --stack-name $STACK --template-body file://ecsapi-demo-cloudformation.yaml
```

Wait for a few seconds for the infrastructure to be created. You can check the status of the stack using the following command:

```shell
awslocal cloudformation describe-stacks --stack-name $STACK | grep StackStatus
```

If the `StackStatus` is `CREATE_COMPLETE`, you can proceed to the next step.

## Building the web application

To build the web application, navigate to the root directory of the sample application and run the following commands:

```shell
cd client-application-react
yarn
yarn build
```

Ensure a `build` directory is created in the `client-application-react` directory.

## Deploying the web application

To deploy the web application, we will make an S3 bucket and sync the `build` directory to the S3 bucket. Run the following commands from the `client-application-react` directory:

```shell
awslocal s3 mb s3://sample-app
awslocal s3 sync build s3://sample-app
```

To access the web application, you can run the following commands:

```shell
export API_ID=$(awslocal apigatewayv2 get-apis | jq -r '.Items[] | select(.Name=="ecsapi-demo") | .ApiId')
export POOL_ID=$(awslocal cognito-idp list-user-pools --max-results 1 | jq -r '.UserPools[0].Id')
export CLIENT_ID=$(awslocal cognito-idp list-user-pool-clients --user-pool-id $POOL_ID | jq -r '.UserPoolClients[0].ClientId')
export URL="http://sample-app.s3.localhost.localstack.cloud:4566/index.html?stackregion=us-east-1&stackhttpapi=$API_ID&stackuserpool=$POOL_ID&stackuserpoolclient=$CLIENT_ID"
echo $URL
```

## Testing the web application

TODO

## GitHub Action

This application sample hosts an example GitHub Action workflow that starts up LocalStack, deploys the infrastructure, and checks the created resources using `awslocal`. You can find the workflow in the `.github/workflows/main.yml` file. To run the workflow, you can fork this repository and push a commit to the `main` branch.

Users can adapt this example workflow to run in their own CI environment. LocalStack supports various CI environments, including GitHub Actions, CircleCI, Jenkins, Travis CI, and more. You can find more information about the CI integration in the [LocalStack documentation](https://docs.localstack.cloud/user-guide/ci/).

## Learn more

The sample application is based on a public  [AWS sample app](https://github.com/aws-samples/ecs-apigateway-sample)  that deploys ECS containers with API Gateway to. See this AWS blog post for more details:  [Field Notes: Serverless Container-based APIs with Amazon ECS and Amazon API Gateway.](https://aws.amazon.com/blogs/architecture/field-notes-serverless-container-based-apis-with-amazon-ecs-and-amazon-api-gateway/)


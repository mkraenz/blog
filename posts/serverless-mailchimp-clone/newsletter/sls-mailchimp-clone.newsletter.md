---
title: Build a Serverless Mailchimp Clone with AWS Step Functions and Amazon Simple Email Service Part 2 - Sending Notifications
description: TODO
tags: "aws, serverless, stepfunctions, lambda"
cover_image: ./newsletter_stepfunctions_graph.png
canonical_url: null
published: false
series: serverless mailchimp clone
---

Today, we learn how to automate new-blog-post email notifications for a blog. We will extract the information from the blog's RSS feed by using an AWS Step Functions state machine with AWS Lambda functions and then send the email with Amazon Simple Email Service. Unlike [last time](TODO add link) where we used the AWS CLI we will use AWS SAM to deploy our application. This is so we can work more easily with TypeScript in our AWS Lambda functions.

At the end of this article, you will feel confident using the AWS SAM, Step Functions, RSS feeds, and Lambda. You will also have a scheduled service that will send email notifications to your email list subscribers.

While this is the second part in a series, you **can follow along** with this article **without having read the first part**. I will include very brief summaries of the relevant points from the first part. If you want to read the first part, you can find it [here](TODO add link).

You can find the code for the current article in this [Github Repository](https://github.com/mkraenz/blog-serverless-mailchimp-clone).

## Target Workflow

TODO

## Contents

- [Target Workflow](#target-workflow)
- [Contents](#contents)
- [Prerequisites](#prerequisites)
- [Pricing](#pricing)
- [What is AWS SAM?](#what-is-aws-sam)
- [Setup the Project](#setup-the-project)
- [Understanding `template.yaml`](#understanding-templateyaml)
- [Cleanup](#cleanup)
- [References](#references)

## Prerequisites

- AWS account. If you don't have an account yet, you can create one here via the [AWS website](https://aws.amazon.com/).
- [AWS CLI](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html) installed and configured (I tested with `aws-cli/2.7.31 Python/3.9.11` on Ubuntu Linux)
- Access permissions on your user (I tested with `AdministratorAccess` policy.)
- default profile and default region set in your AWS CLI config
  - (alternatively, pass `--profile YOUR_PROFILE` and `--region YOUR_REGION` flags to each AWS CLI commands)
- [SAM CLI](https://aws.amazon.com/serverless/sam/#Install_SAM_CLI:~:text=Started%20with%20SAM-,Install%20SAM%20CLI) (I tested with version `1.56.1`)

## Pricing

tl;dr: Following this article will cost you less than 10 cents during development.

As with the first part in this blog series, pricing is close to zero thanks to the [AWS Free Tier](https://aws.amazon.com/free/). You will only be charged for a few emails that you send. The cost of sending 1000 email is about $0.10. You can find the exact pricing [here](https://aws.amazon.com/ses/pricing/). During development you will only send a few emails to yourself, so it's essentially free.

## What is AWS SAM?

AWS SAM, or AWS Serverless Application Model, is a tool for building serverless applications on AWS. You write a yaml file called the _SAM template_ containing the resources you want to deploy, like a Lambda function and corresponding API endpoint, and then SAM will deploy it for you with 2 simple commands: `sam build` and `sam deploy`. SAM will also compile our TypeScript code for the Lambda functions into JavaScript

Under the hood, it builts on top of AWS CloudFormation so if you are familiar with CloudFormation you will feel right at home with SAM. In fact, you can write CloudFormation code inside the SAM template file. If not, don't worry, SAM is fairly easy to start with thanks to the command line tool and the [serverless patterns collection](https://s12d.com/patterns).

## Setup the Project

To make our lives easier, we start with a project template.

```sh
sam init \
    --runtime nodejs16.x \
    --dependency-manager npm \
    --app-template hello-world-typescript \
    --name sls-mailchimp-clone
cd sls-mailchimp-clone
```

Let's take a look at the generated files:

- `template.yaml` - the heart of every SAM application. This is where we define our AWS resources and their properties.
- `hello-world/` - the directory containing an example Lambda function `app.ts` written in TypeScript. Inside the directory, you find a usual nodejs project setup including `package.json`, linter setup (eslint), unit test setup (jest), and `tsconfig.json` containing our TypeScript compiler config.
- `README.md` - a short description of the project. Feel free to read it to learn more about SAM.

That's it. We have a working project. Let's deploy to AWS.

First, let's ensure we have all node modules installed:

```sh
cd hello-world
npm install
cd ..
```

Then, we can deploy the application with the following command:

```sh
sam build
```

This command will turn our TypeScript code into JavaScript and package any `node_modules` dependencies, too. The output will be in the `.aws-sam/build` directory and is fairly readable.

It seems to also be necessary to run this command even when you have not changed anything in the TypeScript code but only inside the sam `template.yaml`. I am not sure why this is the case but it is not a big deal.

The next step is to deploy to AWS. Since this is the first deployment, we use the `--guided` flag to be guided through the deployment process. Every following deployment will omit the `--guided` flag.

```sh
sam deploy --guided
```

The wizard guides us through a few questions. Make sure to enable saving the configuration to `samconfig.toml` so you can easily redeploy later. It's always a good idea to have everything in version control for replicability.

I used the following settings:

```log
Stack Name [sam-app]: sls-mailchimp-clone
AWS Region [us-east-1]: us-east-1
Confirm changes before deploy [y/N]: y
Allow SAM CLI IAM role creation [Y/n]: Y
Disable rollback [y/N]: N
HelloWorldFunction may not have authorization defined, Is this okay? [y/N]: y
Save arguments to configuration file [Y/n]: Y
SAM configuration file [samconfig.toml]: samconfig.toml
SAM configuration environment [default]: default
```

- `Stack Name` is the underlying name for our deployment in CloudFormation. Stacks are the units of deployment in CloudFormation. Each stack changes atomically, and if something goes wrong it will also be rolled back to the previous working state.
- `Confirm changes before deploy` is a good idea to make sure you are deploying what you expect. On each deploy, you will first see a list of changes that will be applied. If you are happy with them, you can confirm them. In a CI/CD pipeline, you likely want to disable this manual confirmation.
- `HelloWorldFunction may not have authorization defined` is about our Lambda being publically available on the internet. It shouldn't be much of a problem as the function is not doing anything dangerous and the endpoint is essentially unknown to anyone. We will change this later.

After this step, the actual deployment occurs. You may need to confirm your deployment. The resources to be deployed should look like this (except for the `1111aaaa` id):

| Operation | LogicalResourceId                          | ResourceType               | Replacement |
| --------- | ------------------------------------------ | -------------------------- | ----------- |
|           |                                            |                            |             |
| + Add     | HelloWorldFunctionHelloWorldPermissionProd | AWS::Lambda::Permission    | N/A         |
| + Add     | HelloWorldFunctionRole                     | AWS::IAM::Role             | N/A         |
| + Add     | HelloWorldFunction                         | AWS::Lambda::Function      | N/A         |
| + Add     | ServerlessRestApiDeployment1111aaaa        | WS::ApiGateway::Deployment | N/A         |
| + Add     | ServerlessRestApiProdStage                 | AWS::ApiGateway::Stage     | N/A         |
| + Add     | ServerlessRestApi                          | AWS::ApiGateway::RestApi   | N/A         |

In essence, we deploy a Lambda function that is publically callable via HTTP using API Gateway. We further have an IAM role to allow our Lambda function to push logs to Cloud Watch and a some roles and permissions to allow

It will take a minute for everything to be provisioned on AWS so let's use that time to brew fresh tea. ;)

Congratulations! You have deployed your first app to AWS using SAM.

To verify it worked, check your console logs for the following line and open the URL in the browser to invoke your Lambda function.

```log
--------
Outputs
--------
...

Key           HelloWorldApi
Description   API Gateway endpoint URL for Prod stage for Hello World function
Value         https://YOUR_API_ID.execute-api.us-east-1.amazonaws.com/Prod/hello/
```

Your browser should show

```json
{
  "message": "hello world"
}
```

## Understanding `template.yaml`

Let's check `template.yaml` to find out how we deployed all of this.

Beginning with the most important part, we have the `Resources` section.

```yaml
Resources:
  HelloWorldFunction:
    Type: AWS::Serverless::Function
    Properties:
      CodeUri: hello-world/
      Handler: app.lambdaHandler
      # ...
```

This is where we define what we want to deploy on AWS. In this case, we deploy a Lambda function ([AWS::Serverless::Function](https://docs.aws.amazon.com/serverless-application-model/latest/developerguide/sam-resource-function.html) is basically a more comfortable definition of a Lambda function available in SAM). Pretty much every resource has a `Type` field and a `Properties` field. The `Type` defines it's SAM resource type or CloudFormation resource type.

> Googling for a resource's `Type` may also be the best way to find the documentation including what `Properties` are available.

For the properties, we specify where the code is located and which module export to run when the Lambda function is invoked. Inside of `hello-world/app.ts` you will find

```ts
export const lambdaHandler = // ...
```

> The Lambda's `Handler` property and the module export's name must match. So always change them together.

## Cleanup

One great thing about infrastructure as code is that we can easily tear down our resources. Compare this to the list of about 20 commands we had to manually track and delete in the first part of this series.

```sh
sam delete
```

## References

- [AWS SAM](https://aws.amazon.com/serverless/sam/)
- [Serverless Pattern Collection](https://s12d.com/patterns)
- [Building TypeScript Projects with SAM](https://aws.amazon.com/blogs/compute/building-typescript-projects-with-aws-sam-cli/)

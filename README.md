# uphone-deployment
Instruction on how to deploy the trinimbus web application assignments

## Introduction
This repo contains instruction on how to deploy a simple single page application built with react + AWS Lambda and that allows for the verification of 
users phone numbers. This work was produced for **Byron Packwood** as part of the [Trinumbus](https://www.trinimbus.com/) hiring process for solutions architect.

## Live Demo
An already deployed live demo of the application can be found here []()

## Deployment
This sections details the step required to deploy the solution on your on AWS account.

### Requirements

#### IAM permissions
The successful deployment of the solution will require the credential of an IAM user who has the permissions found in the iam_user_policy.json file 

#### Dependencies
I will assume that you have the following dependencies installed on your deploy machine:

1. Python3.6 + pip3
2. virtualenv
3. nodeenv
4. awscli
5. git

### Setup
Follow these steps to deploy the solution to your AWS account.

#### Backend Deployment

```bash
# Checkout and build the code for the backend
git clone https://github.com/magnus1984/muphone.git
cd muphone
virtualenv -p python3.6 .runtimeenv
source .runtimeenv/bin/activate
pip install -r requirements.txt -t muphone/build
cp muphone/*.py muphone/build

# Deploy the backend infrastructure
aws s3api create-bucket --bucket jomagnus1984-trinimbus-deploy-bucket --region ca-central-1 --create-bucket-configuration LocationConstraint=ca-central-1
aws cloudformation package --template-file template.yaml --s3-bucket jomagnus1984-trinimbus-deploy-bucket --output-template-file packaged.yaml
aws cloudformation deploy --template-file packaged.yaml --stack-name jomagnus1984-trinimbus-backend --capabilities CAPABILITY_IAM
```

Wait for the deployment to finish and query your stack for the API_ENDPOINT

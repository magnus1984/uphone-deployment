# uphone-deployment
Instruction on how to deploy the trinimbus web application assignments

## Introduction
This repo contains instruction on how to deploy a simple single page application built with React + AWS Lambda and that allows for the verification of 
users phone numbers. This work was produced for **Byron Packwood** as part of the [Trinumbus](https://www.trinimbus.com/) hiring process for solutions architect.

## Live Demo
An already deployed live demo of this application can be found here []()

## Deployment
This sections details the step required to deploy the solution on your on AWS account.

### Requirements

#### Operating System
Ubuntu 16.04 LTS

#### IAM user and permissions
The successful deployment of the solution will require the credential of an IAM user who has the permissions found in the iam_user_policy.json file.

#### Dependencies
I will assume that you have the following dependencies installed on your deploy machine:

1. [Python3.6 + pip3](https://www.python.org/)
2. [virtualenv](https://github.com/pypa/virtualenv)
3. [nodeenv](https://github.com/ekalinin/nodeenv)
4. [awscli](https://docs.aws.amazon.com/cli/latest/userguide/cli-chap-welcome.html)
5. [git](https://git-scm.com/)

for installation and setup instructions, please refer to the respective projects documentation.

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

Wait for the deployment to finish and query your stack for the api endpoint

```bash
# get the api endpoint url
REACT_APP_API_ENDPOINT_URL=`aws cloudformation describe-stacks --stack-name jomagnus1984-trinimbus-backend --query 'Stacks[0].Outputs[0].OutputValue' --profile jomagnus1984 --region ca-central-1 | sed -e 's/\"//g'`
# cleanup a bit
deactivate
cd ..
rm -rf muphone
```

#### Frontend Deployment

```bash
# Checkout and build the code for the frontend


```

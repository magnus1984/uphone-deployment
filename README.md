# uphone-deployment
Instruction on how to deploy the trinimbus web application assignments

## Introduction
This repo contains instructions on how to deploy a simple single-page application built with React + AWS Lambda that allows for the verification of 
users phone numbers, as is commonly seen on many websites. This work was produced for **Byron Packwood** as part of 
the [Trinimbus](https://www.trinimbus.com/) hiring process for solutions architect.

## Live Demo
An already deployed live demo of this application can be found [here](http://jomagnus1984-trinimbus-frontend-hostingbucket-1pmtg54qeouiu.s3-website.ca-central-1.amazonaws.com/)

## Solution architecture overview
The web application is a 3-tier application using the following components for each tiers:

1. AWS S3 for the presentation layer (highly-available).
2. AWS Lambda for the application layer (highly-available).
3. AWS DynamoDB for the data layer (highly-available).

Every layer is deployed on multiple AZ, as part of AWS managed services.
The solution conforms to modern idioms
of web development such as single-page applications, serverless architecture and highly-available NoSQL databases 
like DynamoDB. 

The solution will be assembled from 2 projects I have built. the repos for the projects can be found here:

1. https://github.com/magnus1984/muphone.git
2. https://github.com/magnus1984/muphone-frontend.git

## Deployment
This sections details the step required to deploy the solution on your on AWS account.

### Important note on https
In order to support https, we need to have a CloudFront distribution in front of our S3 bucket. In the interest
of saving time, I have not included this as part of my solution. I have blogged about using AWS Certificate Manager
with S3 and CloudFront to support SSL and you can read the details 
[here](https://hedgenet.info/posts/static-s3-cloudformation.html) if you are interested.

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
Please see the documentation for the backend [here](https://github.com/magnus1984/muphone.git) and then proceed
with the installation.

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
REACT_APP_API_ENDPOINT_URL=`aws cloudformation describe-stacks --stack-name jomagnus1984-trinimbus-backend --query 'Stacks[0].Outputs[0].OutputValue' | sed -e 's/\"//g'`
# cleanup a bit
deactivate
cd ..
rm -rf muphone
```

#### Frontend Deployment
Please see the documentation for the frontend [here](https://github.com/magnus1984/muphone-frontend.git) and then proceed
with the installation.
```bash
# Checkout and build the code for the frontend
git clone https://github.com/magnus1984/muphone-frontend.git
cd muphone-frontend
nodeenv .frontenv
source .frontenv/bin/activate
npm install

# Build the application
REACT_APP_API_ENDPOINT_URL=$REACT_APP_API_ENDPOINT_URL npm run build

# Deploy the frontend infrastructure
aws cloudformation deploy --template-file src/infrastructure.yaml --stack-name jomagnus1984-trinimbus-frontend
```

Wait for the deployment to finish and query your stack for it's deployment bucket

```bash
# Get the bucket name were to deploy the static assets
DEPLOY_BUCKET=`aws cloudformation describe-stacks --stack-name jomagnus1984-trinimbus-frontend --query 'Stacks[0].Outputs[1].OutputValue' | sed -e 's/\"//g'`

# sync the build folder with the bucket
aws s3 sync build s3://$DEPLOY_BUCKET
```

The application is now fully deployed. To access it, check the last stack output for the CloudFront distribution URL:

```bash
aws cloudformation describe-stacks --stack-name jomagnus1984-trinimbus-frontend --query 'Stacks[0].Outputs[0].OutputValue'
```

Type that address in your browser and test the application for yourself !

## Author
Jonathan Pelletier (jonathan.pelletier1@gmail.com)
Website: [https://hedgenet.info](https://hedgenet.info)
Other work of interest: [Static Website on S3 + SSl support using AWS Certificate Manager](https://hedgenet.info/posts/static-s3-cloudformation.html)

---
pagination_next: null
pagination_prev: null
sidebar_position: 2
---

# AWS

For Cloudwhistle to monitor your AWS account's spending in real-time, a cross-account IAM role in your AWS account needs to be created.

This role allows access to certain AWS APIs, for instance, to query the list of EC2 instances or how many invocations your AWS Lambda has received recently.

You don't have to set up the cross-account IAM role by hand, we provide a CloudFormation template that automates the process.

### CloudFormation template

When you set up a new AWS account, the instructions will guide you through the option of installing the CloudFormation template via the AWS Console or the AWS CLI. Either way, the end result is the same.

The latest version of the Cloudformation template can be found here:
https://cloudwhistle-bridge.s3.amazonaws.com/template.yaml - it has been annotated extensively to explain why each resource is required.

Almost all AWS policy actions Cloudwhistle requires are read-only; the only exception is the ability to create a Resource Explorer index/view.

### Resource Explorer

Cloudwhistle tries to be efficient as possible. To that end, it uses the AWS Resource Explorer API to discover all the resources in your AWS account.

If they don't yet exist, Cloudwhistle will:

- create a Resource Explorer index for each region
- create a Resource Explorer view called "Cloudwhistle"
- make the us-east-1 index an aggregator index

Again, if any of these already exist, they will not be altered.

This is the only non-read-only API Cloudwhistle requires access to.

### Support & Limitations

Currently, Cloudwhistle supports:

- EC2 instances (only on-demand)
- Lambda functions
- ECS/Fargate tasks

The known limitations are:

- "AWS Free Tier" or "AWS Compute Savings Plans" are not taken into account (yet)
- resource usage is updated once every 5 minutes
- resource specs (such as a Lambda function's memory) are updated once per hour
- pricing data is updated once per day
- `cn-north-1` and `cn-northwest-1` regions are not supported (since unit prices are not in USD)

And of course, an outage affecting any of the monitored regions and/or services will negatively impcat Cloudwhistle's monitoring abilities.

### Cost

Except for one API endpoint, Cloudwhistle only uses AWS APIs that are free of charge. The only exception is the Metrics API.

The Metrics API is used for some of the usage-based resources, namely AWS Lambda, to monitor the number of invocations and their duration. This is required to calculate the cost of the Lambda function.

Cloudwhistle uses the `GetMetricStatistics` API which is eligable for the 1 million _free_ Cloudwatch Metric API requests per month that every account receives.

Assuming no other services are using the Metrics API in your AWS account, the **Free Tier supports monitoring up to 57 AWS Lambda functions for free** (2 metrics, each 8640 requests per month).

Beyond that, it's $0.01 per 1K requests, meaning any extra AWS Lambda function costs $0.17 per month to monitor.

### Uninstall

In order to uninstall Cloudwhistle from your AWS account, you need to _delete_ the CloudFormation stack that was created during the installation process. It is called "CloudwhistleBridge" and is installed in us-east-1. You can find it [here in your AWS console](https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks).

Of course, after you delete the Cloudformation stack, you will no longer be able to see your AWS account's spending in Cloudwhistle.

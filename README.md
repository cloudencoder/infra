## Intro

CloudEncoder is an open source project that allows you to deploy a scalable video encoding solution on AWS.
The infrastructure is implemented with CloudFormation.

Encoding jobs are submitted via the CloudEncoder API and added to a queue. A number of worker nodes (EC2 instances) retrieve jobs from the queue and encode the videos according to the job specifcation. 

## Setup

Before you deploy the CloudEncoder infrastructure (`cloudencoder.yaml`) you must deploy the S3Policy macro (`macro.yaml`). It takes the names of the input and output bucket parameters and generates an S3 policy dynamically that grants access to the specified buckets to the EC2 instances.

**Note: You only have to deploy the macro once**

1. Log in to the AWS Management Console
2. Select CloudFormation from the list of AWS services
3. Click on "Create Stack"
4. Select "Upload a template file". Select `macro.yaml` then click "Next"
5. Choose a name for the stack. Anything will do. Click "Next"
6. Naviate throught the remaining steps, accepting the defaults, then click "Create Stack"

The next step is to deploy the CloudEncoder infrastructure. Repeat the steps above but this time using `cloudencoder.yaml` as the template file. This time you will be prompted to enter a number of parameters (see [Parameters](#parameters)).

It takes approximately 5 minutes to deploy the CloudEnocder infrastructure.

## Parameters

All parameters are required except the email address:

| Name | Description | Default |
|---|---|---|
| CidrBlock | First two octets of the CIDR block for the VPC. | 10.100 |
| InputBucket | Comma-separated list of bucket names from where worker nodes download videos, e.g. myvideos | - |
| OutputBucket | Comma-separated list of bucket names where encoded videos are uploaded, e.g. myencodedvideos | - |
| InstanceType | The instance type of the EC2 instances | c5.large |
| NumberOfInstances | The total number of EC2 instances to create | 1 | 
| VolumeSize | The size of the (root) EBS volume on each EC2 instance. If your videos are large, you will want to increase this value accordingly | 8 GB |
| MarketType | Whether to use spot instances or on-demand for the EC2 instances | on-demand |
| OnDemandInstances | The number of on-demand instances to create. Should be less than or equal to NumberOfInstances | 1 |
| EmailAddress| Sends a notification to the provided email address when the encoding job finishes. You'll receive an initial email asking to confirm your subscription, which you will need to do before receiving any further notifications. *Note: In the future notifications will be configured on a per job basis.*| - |
| Username | Part of the credentials to access the CloudEncoder API | admin |
| Password | Part of the credentials to access the CloudEncoder API | admin |
| WorkerUrl | The URL where to download the encoding application that runs on the EC2 instances | - |

### Example

Let's say you want to have five workers (EC2 instances) running. To keep costs down, four of them are to be spot instances and the remaining one should be an on-demand instance. (It's a good idea to have at least one on-demand instance available so you are guaranteed to have some capacity available to encode your videos.) To achieve this, set the following parameters to the values shown:

* NumberOfInstances = 5
* MarketType  = spot
* OnDemandInstances = 1

## Outputs

| Name | Description | 
|--|--|
| EndpointUrl | The URL of the CloudEncoder API |

## API

To submit an encoding job, you need to POST a JSON file representing the job (see [API](https://github.com/cloudencoder/infra/wiki/API)) to the CloudEncoder API endpoint URL. You'll need to specify the credentials you entered when creating the CloudEncoder infrastructure.

Here's an example using curl:

```bash
$ curl -u admin:admin -d @job.json https://m60l4dr8f8.execute-api.eu-west-2.amazonaws.com/v1/job
```
When the job has finished, you'll receive an email confirmation.


Simon Buckle, simon@webteq.uk

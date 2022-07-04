# aws-cfn-vpc-peer-acceptor

AWS CloudFormation Stack for VPC Peering Acceptor.

## Usage

### stack.json

This repo contains a standard AWS CloudFormation template `stack.json`
which can be deployed with AWS web console, AWS CLI or any other AWS
CloudFormation compitable tool.

This repo can be used along with below repoes:

* [aws-cfn-vpc-peer-requester](https://github.com/alexzhangs/aws-cfn-vpc-peer-requester)
* [aws-cfn-vpc](https://github.com/alexzhangs/aws-cfn-vpc)

To create cross account VPC peer connections, one acceptor peers with
multi requesters. These repoes may make the process easier.

However you will need to make a new template to put all these together,
put `aws-cfn-vpc`, `aws-cfn-vpc-peer-acceptor` and
`aws-cfn-vpc-peer-requester` as the nested stack of your new stack.

About how to do this, you may refer to a real world example
[aws-cfn-vpn](https://github.com/alexzhangs/aws-cfn-vpn), which put
all these together, and is able to create one(acceptor) to many(requester) cross
account VPC peer connections.

This template will create an AWS CloudFormation stack, including
following resources:

* 1 IAM role to automate the acception of VPC peer connection
  request from another VPC.
* 1 SQS queue as a broker to trigger the new route entry creation
  of  the VPC route table.
* 1 IAM role to give the Lambda function necessary permissions for the
SQS, route table and the logs.
* 1 Lambda function to pull SQS messages and edit VPC route table.

For the input parameters and the detail of the template, please check the template
file.

## Troubleshooting

1. API: ec2:AcceptVpcPeeringConnection Roles may not be assumed by root accounts

   You must create IAM user or role to deploy the
   stack, you can not use AWS root user or its access key to do the
   deployment. Because there is IAM assume role inside the template,
   which assumes an action `ec2:AcceptVpcPeeringConnection` and AWS
   restricts it's can't be assumed by root user. Otherwise an error would
   be found in the events of stack while deployment.

   ```
   "ResourceStatus": "CREATE_FAILED",
   "ResourceType": "AWS::EC2::VPCPeeringConnection",
   "ResourceStatusReason": "API: ec2:AcceptVpcPeeringConnection Roles may not be assumed by root accounts"
   ```

1. The VPC peer connection is active but connecting to the IP:port of the resource in the peering VPC is timeout.

   Check following AWS resources on the console:

   1. CloudWatch -> Logs, check the log of Lambda execution.
   2. SQS -> Queue, check the message of queue.
   3. VPC -> Route Tables -> Routes.
   4. Lambda - Functions.
   5. VPC -> Security Groups -> Inbound Rules.
   6. EC2 -> Get System Log (/var/log/cloud-init-output.log).
   7. IAM -> Role.

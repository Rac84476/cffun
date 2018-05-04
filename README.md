## Instructions

### Create CloudFormation Stack

Creates AWS infra. The infra runs the Fugue Demo App.

```
aws --region us-west-2 cloudformation create-stack \
  --template-body file://DemoApp.json \
  --capabilities CAPABILITY_IAM \
  --stack-name DemoApp
```

### Add DeletionPolicy

Add a `DeletionPolicy` to every resource in the CloudFormation JSON file. Creates a new file called
`DemoAppRetain.json`.

```
cat DemoApp.json | jq '.Resources[].DeletionPolicy = "Retain"' > DemoAppRetain.json
```

### Update Stack with DeletionPolicy Attribute Retain

```
aws --region us-west-2 cloudformation update-stack \
  --capabilities CAPABILITY_IAM \
  --template-body file://DemoAppRetain.json \
  --stack-name DemoApp
```

### Copy IDs into Transcriber filter file

Some resources, which cannot be identified with a tag, need to be specifically included in a Transcriber
filter file. In this example, we need to explicitly spcefify:

- Launch Configuration Name
- DynamoDB Table Name
- Instance Profile Name
- Role Name

To get the instance profile name:
```
ips=$(aws iam list-instance-profiles)
echo $ips | jq -r '.InstanceProfiles[] | select(.InstanceProfileName | startswith("DemoApp")).InstanceProfileName'
```

### Transcribe

Transcribe the AWS infra that was created by the `DemoAppRetain.json` CloudFormation stack.

```
fugue-transcriber --region us-west-2 --filter-file filter.yaml DemoApp.lw
```

### Import

Import the resources that we transcribed into Ludwig.

```
fugue run DemoApp.lw -a DemoApp --import
```

### Delete CloudFormation Stack

```
aws --region us-west-2 cloudformation delete-stack --stack-name DemoApp
```

## Notes

### Filtering by Tag

(FUGUE-6607) Tags starting with `aws:` are reserved by AWS. We prepend "fugue-transcriber" to tags that start
with "aws:".

We do this because you can't create tags with those names.

You also don't want to name your stack with anything that starts with `Fugue`.

## List Stacks

```
$ aws --region us-west-2 cloudformation list-stacks --stack-status-filter CREATE_COMPLETE
```

## Describe Stack Resources
```
$ aws --region us-west-2 cloudformation describe-stack-resources \
  --stack-name DemoApp | jq -r .StackResources[]
```

## Get Stack Resources Without Tags
```
$ aws --region us-west-2 cloudformation describe-stack-resources --stack-name DemoApp |
  jq -r '.StackResources[] |
  select(.ResourceType == "AWS::AutoScaling::LaunchConfiguration"
  or .ResourceType == "AWS::DynamoDB::Table"
  or .ResourceType == "AWS::IAM::InstanceProfile"
  or .ResourceType == "AWS::IAM::Role")'
```

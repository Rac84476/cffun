## Notes

### Filtering by Tag

(FUGUE-6607) Tags starting with `aws:` are reserved by AWS. We prepend "fugue-transcriber" to tags that start with "aws:".  

We do this because you can't create tags with those names.

You also don't want to name your stack with anything that starts with `Fugue`.

## List Stacks

```
$ aws --region us-west-2 cloudformation list-stacks \
  --stack-status-filter CREATE_COMPLETE
```

## Delete Stacks

```
$ aws --region us-west-2 cloudformation delete-stack \
  --stack-name DemoApp
```

## Create Stack

```
aws --region us-west-2 cloudformation create-stack \
  --template-body file://DemoApp.yaml \
  --capabilities CAPABILITY_IAM \
  --stack-name DemoApp
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

## Update Stack with DeletionPolicy Attribute Retain
```
aws --region us-west-2 cloudformation update-stack 
  --capabilities CAPABILITY_IAM 
  --template-body file://DemoAppRetain.yaml
  --stack-name DemoApp
```

## Transcribe the Stack
```
$ fugue-transcriber --region us-west-2 --filter-file filter.yaml DemoApp.lw
```

## Import the Stack
```
$ fugue run DemoApp.lw -a DemoApp --import
```

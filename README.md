## Create CloudFormation Stack

```
aws --region us-west-2 cloudformation create-stack \
  --template-body file://DemoApp.json \
  --capabilities CAPABILITY_IAM \
  --stack-name DemoApp
```

Verify the app works by isiting URL:

```
aws --region us-west-2 elb describe-load-balancers | jq -r '.LoadBalancerDescriptions[]
    | select(.LoadBalancerName | startswith("DemoApp")) | .DNSName'
```

## Copy Resource IDs into Transcriber filter file

```
aws --region us-west-2 cloudformation describe-stack-resources --stack-name DemoApp |
  jq -r '.StackResources[] |
  select(.ResourceType == "AWS::AutoScaling::LaunchConfiguration"
  or .ResourceType == "AWS::IAM::InstanceProfile"
  or .ResourceType == "AWS::IAM::Role")'
```

## Transcribe

```
fugue-transcriber --region us-west-2 --filter-file filter.yaml DemoApp.lw
```

## Manually edit transcribed Ludwig

- Remove the four external EC2 instances attached to the ELB ([FUGUE-6999][0]).
- Ensure `DemoApp.lw` compiles.
     ```
     lwc DemoApp.lw
     ```

## Import

Import the resources that we transcribed into Ludwig.

```
fugue run DemoApp.lw -a DemoApp --import
```

*DO NOT* proceed until the process status is `SUCCESS`.

It is very important not to delete the CF stack until Fugue has successfully imported the
infrastructure. Otherwise, you may end up in a situation where the CF stack is deleted and Fugue was not able
to import the resources, in which case nothing would be managing the resources.

```
fugue status
```

## Delete the CloudFormation Stack

### Add DeletionPolicy attribute "retain" to CF template

Add a `DeletionPolicy` to every resource in the CloudFormation JSON file. Creates a new file called
`DemoAppRetain.json`.

```
cat DemoApp.json | jq '.Resources[].DeletionPolicy = "Retain"' > DemoAppRetain.json
```

### Update CF stack with DeletionPolicy attribute "retain"

```
aws --region us-west-2 cloudformation update-stack \
  --capabilities CAPABILITY_IAM \
  --template-body file://DemoAppRetain.json \
  --stack-name DemoApp
```

Ensure update is complete. The following should return "UPDATE_COMPLETE".

```
aws --region us-west-2 cloudformation describe-stacks \
  --stack-name DemoApp | jq '.Stacks[] | select(.StackName == "DemoApp") | .StackStatus'
```

### Delete the CF stack

```
aws --region us-west-2 cloudformation delete-stack --stack-name DemoApp
```

Look in the AWS console to confirm that the stack is deleted.

## Verify DemoApp still works

Visit URL:

```
aws --region us-west-2 elb describe-load-balancers | jq -r '.LoadBalancerDescriptions[]
    | select(.LoadBalancerName | startswith("DemoApp")) | .DNSName'
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


[0]: https://luminal.atlassian.net/browse/FUGUE-6999

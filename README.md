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
  --template-body file://DemoApp.cf \
  --capabilities CAPABILITY_IAM \
  --stack-name DemoApp
```

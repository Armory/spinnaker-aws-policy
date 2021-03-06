# spinnaker-aws-policy

There are three pieces of Spinnaker's architecture that need aws policies: Clouddriver, Rosco and Front50. This repo generates a policy for Clouddriver.

## Clouddriver

To generate an AWS Policy for the current head of Spinnaker Clouddriver:

    ./bin/build
    ./bin/run

The policy will be printed to stdout. It will contain all of the necessary actions for Spinnaker's Clouddriver to operate. If you wish, you can isolate any of the actions to certain resources by modifying the policy after it is generated.

### Multiple AWS Accounts

Using multiple AWS accounts requires creating multiple roles with different polices. One role will be in the account Spinnaker resides and the other will be in account Spinnaker is deploying to.

If you have multiple AWS accounts, you will need to create two roles with different policies. One role is for the AWS account that Spinnaker resides in while the other roles are in the accounts that Spinnaker deploys to.

**Managing Account**

The 'Managing Account' is the account that Spinnaker itself runs in. Since most of the work that Spinnaker does is in the deploy target's account the following is the minimum set of permissions you need for this role:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "",
            "Effect": "Allow",
            "Action": [
                "ec2:DescribeRegions",
                "ec2:DescribeAvailabilityZones"
            ],
            "Resource": [
                "*"
            ]
        }
    ]
}
```

Additionally, you will need a policy for each role you assume in another account. That looks something like this:

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Action": "sts:AssumeRole",
            "Resource": [
                "arn:aws:iam::<aws-account-number>:role/<assumed-role-name>"
            ],
            "Effect": "Allow"
        }
    ]
}
```

**Managed Account**

A 'Managed Account' is an account that Spinnaker deploys to. Spinnaker will assume a role in the managed account. The policy required is generated by the output of this repo. Additionally you will need to create a trust relationship between the role in the managed account and the managing account.

You can find more information [here](https://github.com/spinnaker/clouddriver/tree/master/clouddriver-aws)

### Running Locally

If you have a checkout of Clouddriver locally and want to generate the policy for it you can run the python script directly on your development system:

    Run `src/generate.py <CLOUDDRIVER_AWS_DIRECTORY>`

where `<CLOUDDRIVER_AWS_DIRECTORY>` is the location on disk of the clouddriver github repo checked out to which ever version you want to generate the policy for.

The policy will be printed to stdout. It will contain all of the necessary actions for Spinnaker's Clouddriver to operate. If you wish, you can isolate any of the actions to certain resources by modifying the policy after it is generated.


### How does it work?

The generator searches the clouddriver codebase for calls to the AWS API. It maps any API calls to their required permissions.

_Note:_ The method for generating the policy is imperfect, but considered 'good enough'.

## Rosco

The policy that Spinnaker's Rosco requires is a bit different. Since Rosco uses Packer, you can consult HashiCorp's recommendation [here](https://www.packer.io/docs/builders/amazon.html).

## Front50

Front50 needs access to all S3 actions for the bucket and prefix you are using to persist data.

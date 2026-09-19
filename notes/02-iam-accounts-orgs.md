# IAM, Accounts, and AWS Orgs

## IAM Identity Policies

- in line policies, managed policies
- json, containing statement array:
    - Sid
    - action
    - resource
    - effect (allow / deny)
- deny (explicit) > allow (explicit) > deny (default)
- resources can have policies too

## IAM Users and ARN

- max 5k users per account. user can join max 10 groups.
- arn - amazon resource name
    - arn:aws:service:region:account:resource/id
    - region/account can be omitted if not needed. e.g. s3, because global service

## IAM Groups

- max 300 groups per account
- group doesn't have login credentials etc. you can't login with a group.

## IAM Roles

- STS (secure token service)
- temporary
- trust policy, permissions policy
- when to use
    - services running on your behalf
    - emergency ("break glass for key")
    - external ids (e.g. microsoft active directory in a company)
    - an app that uses aws stuff like dynamodb - e.g. ride sharing app with millions of users logging in with google/fb/etc
    - cross account access
- service linked role & pass roles

## AWS Orgs

## SCP (Service Control Policies)

- it's a feature of **AWS orgs**, used to restrict accounts
    - e.g. restricting the region, or the size of ec2 instance an account can use
- allow list vs deny list
    - deny list has less admin overhead
- **v imp to note**: SCPs don't actually grant permissions. rather, they specify what permissions ***can be granted***
- json doc
- attached to org as a whole, or to OU, or even to individual account (incl root user of that account)
- inherit down the org tree
- mgmt account is never affected by SCP, attached anywhere (incl root container)

## CloudWatch Logs

***todo: diagram***

- it's a public service, region resilient

## CloudTrail

- logs any api calls and account events (basically anything that happens to an account)
    - by default, management events only
    - data events to be configured manually (?)
- 90 days event history is default, and provided for free for each account per region
    - anything beyond this, you need to create a trail — and this data gets stored in s3 (only s3 storage is charged here)
- it is regional service, logging events in a region
    - IAM, STS, Cloudfront are global, and a manual trail needs to be configured, and the data gets stored in us-east-1 region
- not realtime. there's 15 mins delay.
- can be integrated with cloudwatch logs

## AWS Control Tower

***todo: diagram***

- kinda an evolution of AWS orgs — AWS orgs with superpowers
- landing zone, guardrails, account factory

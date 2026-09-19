# Advanced EC2

## Bootstrapping EC2 Using User Data

- automate build/install/config steps, via a script
- passed in UserData segment of the cfn for ec2
- userdata is (also?) accessed via meta-data IP
    - http://169.254.169.254/latest/user-data
- userdata is the data user passes, and is executed only once - on launch
    - executed by instance OS
    - it's opaque (user responsible for ensuring it's correct)
    - it's not secure (don't send passwords)
    - 16kb limit
    - can be modified when instance stopped (but executed only at launch)
- check /var/log — cloud related logs to check bootstrap issues

## CFN INIT (and CFN SIGNAL)

- cfn-init helper script (installed on ec2 os)
- procedural (user data) vs desired state (cfn init)
- a config mgmt system
- can be executed again for updates (unlike user data)
- creation policy and signals
    - waits for the signal from the resource that the userdata etc configs worked

## EC2 Instance Roles and Profile

- profile — to give permissions to apps within ec2 instance
    - temp credentials delivered via instance meta-data
- the creds are always renewed before they expire (ec2 and sts liaise)

***todo: diagram***

- [credential precedence](https://docs.aws.amazon.com/cli/latest/userguide/cli-configure-quickstart.html#cli-configure-quickstart-precedence)

## SSM Parameter Store

- String, StringList, SecureString
- public zone service
- encrypted params too, with kms
- hierarchies and versioning

## System Logging and Application Logging

- CloudWatch Agent (because regular cloudwatch doesn't capture inside instance)
    - needs to be installed in instance, and given config and permissions

***todo: diagram***

## EC2 Placement Groups

- 3 types
    - cluster — pack instances close together for low latency, high throughput (single AZ)

    ***todo: diagram***

    - spread — keep instances separated across distinct hardware (max 7 per AZ)

    ***todo: diagram***

    - partition — groups of instances on separate partitions (racks), useful for distributed systems like HDFS/Kafka

    ***todo: diagram***

## EC2 Dedicated Host

## Enhanced Networking and EBS Optimized Instances

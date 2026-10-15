# Infrastructure as Code — CloudFormation

## CFN Physical and Logical Resources

***todo: diagram — logical vs physical resources***

- template — YAML or JSON
- template has logical resources (the "WHAT") → physical resources
- if template is updated/deleted, the stack is updated/deleted too

## CFN Template and Pseudo Parameters

- template params accept input when stack is created/updated
    - can be referenced from logical resources
    - can be configured with defaults, allowed values, type, no echo etc
- pseudo params are provided/injected by AWS
    - e.g. `AWS::Region`, `AWS::StackId`, `AWS::AccountId` etc
- avoid template params wherever possible

## CFN Intrinsic Functions

- access to data at runtime
- `Ref` & `Fn::GetAtt`
- `Fn::Join` & `Fn::Split`
- `Fn::GetAZs` & `Fn::Select`
- conditions (`Fn::If`, `And`, `Equals`, `Not`, `Or`)
- `Fn::Base64` & `Fn::Sub`
- `Fn::Cidr`
- etc

## CFN Mappings

- it's a top-level object (like Resources, Parameters etc)
- mappings can contain mappings
- mappings use `!FindInMap` intrinsic function, needs at least 1 key

    ```yaml
    !FindInMap [MapName, TopLevelKey, SecondLevelKey]
    ```

- improve template portability
- mapping e.g.

    ```yaml
    Mappings:
        RegionMap:
            us-east-1:
                HVM64: "ami-..."
                HVMG2: "ami-..."
            us-west-1:
                HVM64: "ami-..."
                HVMG2: "ami-..."

    # findmap function for this mapping:
    !FindInMap ["RegionMap", !Ref "AWS::Region", "HVM64"]
    ```

## CFN Outputs

- templates can have an optional outputs section

## CFN Conditions

- optional "conditions" section of the template
- processed before resources are created
- `Fn::If`, `And`, `Equals`, `Not`, `Or`
- e.g. template

    ```yaml
    Parameters:
        EnvType:
            Type: String
            Default: "dev"
            AllowedValues:
                - "dev"
                - "prod"

    Resources:
        Instance:
            Type: AWS::EC2::Instance
            Condition: IsProd
            Properties:
                ImageId: "..."

    Conditions:
        IsProd: !Equals
            - !Ref EnvType
            - "prod"
    ```

- can also be nested

## CFN DependsOn

- an attribute to define dependencies within resources
- determines order of creation of resources
- needed when implicit dependency won't work or isn't possible
    - e.g. creation of an internet gateway attachment has an implicit dependency on both the VPC and the IGW (because of `!Ref`)
    - but, creating an elastic IP within a specific subnet of a VPC won't work without explicitly stating the dependency on the IGW attachment (fails otherwise, because there's no `!Ref` to the attachment)
    - like here:

        ```yaml
        Resources:
            VPC:
                ...
            IGW:
                ...
            IGWAttachment:
                ...
                VpcId: !Ref VPC
                InternetGatewayId: !Ref IGW
            WPEIP:
                Type: AWS::EC2::EIP
                DependsOn: IGWAttachment
                Properties:
                    ...
        ```

## CFN CreationPolicy, cfn-signal, and WaitConditions

- can configure CFN to wait for a certain number of success signals
    - for a timeout duration
- creation fails if either a failure signal is received or the timeout is reached
- used to signal logical resources such as EC2 or ASG which are using a creation policy or wait conditions
    - (AWS suggests using creation policy for provisioning EC2 and ASG)
- WaitCondition
    - it's a logical resource itself
    - depends on WaitHandle (another logical resource) — generates a pre-signed URL for resource signals

## CFN Nested Stacks

- because there are limitations to having a single stack:
    - 500 resources limit
    - can't reuse/reference resources in other stacks
- two ways to solve this:
    - nested stacks
    - cross-stack references
- nested stacks:

    ***todo: diagram — nested stack hierarchy***

    - allow a hierarchy of related templates to be combined to form a single product
    - root stack — stack that's created first, manually, by an entity
    - parent stack — parent of any stack that it immediately creates
    - should only be used when everything is lifecycle-linked (else use cross-stack references)

## CFN Cross-Stack References

***todo: diagram — cross-stack export/import***

- outputs can be exported, making them visible to other stacks
- `Fn::ImportValue` can be used instead of `Ref`

    ```yaml
    Resources:
        VPC:
            ...
    ...
    Outputs:
        SHAREDVPCID:
            Value: !Ref VPC
            Export:
                Name: SHAREDVPCID
    ```

- require unique export names, per region per account
- ideal for re-using the stack/resources (not the template), different lifecycles

## CFN StackSets

- allow stack/infra to be created and managed across multiple regions and accounts from a single location
- StackSets are containers in an **admin account**, containing stack instances which reference stacks (in **target accounts**)
- CFN assumes a role to interact with the target account
- concurrent accounts — the number of individual accounts being deployed to at the same time
- failure tolerance — the number of individual deployments that can fail before the stack set itself is viewed as failed
- retain stacks — prevents certain stacks from being deleted
- use cases/scenarios
    - enable AWS Config across a large range of accounts
    - create IAM roles for cross-account access
    - AWS Config rules — MFA, EIPs, EBS encryption

## CFN Deletion Policy

- by default, the physical resource is deleted when the logical resource is deleted
- but, can optionally retain the resource or take a snapshot (if applicable)
- deletion policy doesn't apply in cases where the resource is replaced

## CFN Stack Roles

***todo: diagram — stack role assumption flow***

- by default CFN uses the permissions of the identity that's creating the stack
- but, roles can be configured for CFN to assume (for role separation etc):
- admin creates the role → gives it to a non-admin user → this user can't assume the role, can only pass it to CFN → CFN assumes the role and creates the stack

## CFN Init

- provides config info to resources (desired state config management)
- e.g.

    ```yaml
    ...
    MyInstance:
        Type: AWS::EC2::Instance
        Metadata:
            AWS::CloudFormation::Init:
                configSets: ...
                <config_key>:
                    packages: ...
                    groups: ...
                    users: ...
                    sources: ...
                    files: ...
                    commands: ...
                    services: ...
                <config_key>: ...
                ...
        UserData:
            Fn::Base64: !Sub |
                #!/bin/bash -xe
                yum -y update
                /opt/aws/bin/cfn-init -v --stack ${AWS::StackId} --resource MyInstance --region ${AWS::Region} --configsets <config_set>
                /opt/aws/bin/cfn-signal -e $? --stack ${AWS::StackId} --resource MyInstance --region ${AWS::Region}
    ```

- there can be multiple config keys — these are containers of config directives, containing the same sections as above
- the configSets section contains which config keys to use and in what order

## CFN cfn-hup

***todo: diagram — cfn-hup change detection loop***

- a helper daemon, needs to be installed
- detects changes in resource metadata, and runs config actions if changes are there (unlike cfn-init, which runs only once, during bootstrapping)

## CFN ChangeSets

- allows review of changes before applying
- multiple changesets can be created
- execute a changeset when satisfied with the changes

## CFN Custom Resources

***todo: diagram — custom resource / Lambda event flow***

- CFN doesn't always support everything in AWS (there's either a lag, or no support ever)
- hence, custom resources for everything that's not supported natively by CFN
- e.g.
    - populate S3 with objects when created, or empty the bucket before delete
    - request config from an external system when setting up an EC2 instance
- done using Lambda etc — passing event data to Lambda to do custom stuff

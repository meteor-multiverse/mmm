# meteor-multiverse-manager

Meteor Multiverse Manager (a.k.a. `mmm`) is a custom CLI for working with [Meteor Multiverse][] environments on Amazon [AWS OpsWorks][]


## Prerequisites

 - Node.js >= `0.10.x`
 - NPM >= `2.x`
 - An Amazon AWS account with:
     - access to the following services:
         - IAM: Identity & Access Management
         - EC2: Elastic Compute Cloud
         - VPC: Virtual Private Cloud _(technically part of EC2)_
         - OpsWorks
         - Route53
     - at least 1 HostedZone (domain) available in Route53 for sub-domaining
     - an active [IAM Access Key](http://docs.aws.amazon.com/IAM/latest/UserGuide/id_credentials_access-keys.html) for accessing the AWS REST API via the AWS CLI
     - policy permissions for your "role" as [detailed below](#complete-aws-iam-policies)
 - Running the tool on:
     - Mac OSX
     - Linux/UNIX environments with Bash
     - _Maaaaybe_ on Windows environments with a Bash-compatible shell like Git Bash (msysgit)


## Installation

```shell
sudo npm install -g meteor-multiverse-manager
```

`sudo` is used here to ensure that a global symlink can be created for running the resulting `mmm` command _anywhere_.


## Usage

### Command Syntax

```shell
mmm <command> [options] [args]
```

To learn about all of the available commands, run:

```shell
mmm --help
```

To learn about each command's available options and arguments, run:

```shell
mmm <command> --help
```


### Basic Example

This example will guide you through initializing a new stack config file:

```shell
mmm init -c my-stack-config
```


### Available Commands

Again, to learn about all of the available commands, run:

```shell
mmm --help
```

However, for the sake of learning more about what Meteor Multiverse Manager is capable of _without_ having to install it first, here is the **current** list of available commands and a brief description of each:

 - `init` &mdash; Initialize a brand new OpsWorks stack configuration through an interactive series of prompts
 - `create` &mdash; Create a new OpsWorks stack
 - `get` &mdash; Get a thorough JSON representation of an existing OpsWorks stack
 - `delete` &mdash; Delete an existing OpsWorks stack
 - `dns` &mdash; Forcibly sync up the Route53 DNS records for an existing OpsWorks stack
 - `bad` &mdash; Build a local Meteor app, push it, and deploy it to an existing OpsWorks stack
 - `build` &mdash; Build a local Meteor app configured as if for an existing OpsWorks stack
 - `push` &mdash; Push a locally built Meteor app bundle to the release repo for an existing OpsWorks stack
 - `deploy` &mdash; Deploy the latest pushed Meteor app bundle from the release repo to an existing OpsWorks stack
 - `update-env` &mdash; Update the App environment variables of an existing OpsWorks stack
 - `cook` &mdash; Update the Chef cookbooks for an existing OpsWorks stack
 - `exec` &mdash; Execute a command against an existing OpsWorks stack
 - `heal` &mdash; Forcibly auto-heal any instances that failed to start within an existing OpsWorks stack
 - `up` &mdash; Ensure all of the Meteor app servers within an existing OpsWorks stack are up
 - `scale` &mdash; Synchronize the schedule of all time-based scaling instances and the metrics used for all load-based scaling instances within an existing OpsWorks stack
 - `lock` &mdash; Lock all SSHD entry points of an existing OpsWorks stack
 - `unlock` &mdash; Unlock all SSHD entry points of an existing OpsWorks
 - `mongo` &mdash; Connect to the Mongo database of an existing OpsWorks stack with the Mongo Shell
 - `sim` &mdash; Simulate running as an existing OpsWorks stack by reconfiguring local environment variables to utilize the same resources (e.g. Mongo databases, etc.)
 - `ssh` &mdash; SSH connect to an instance in an existing OpsWorks stack via a Bastion Host

<!---
 - `scp` &mdash; SSH connect to an instance in an existing OpsWorks stack via a Bastion Host and copy files up/down
-->


## Complete AWS IAM Policies

### Stack Owners

The IAM Policy Permissions necessary to create and destroy Stacks.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "IamAccessForStackOwners",
      "Action": [
        "iam:ListRoles",
        "iam:GetRole",
        "iam:CreateRole",
        "iam:ListRolePolicies",
        "iam:ListAttachedRolePolicies",
        "iam:GetRolePolicy",
        "iam:GetPolicy",
        "iam:PutRolePolicy",
        "iam:DetachRolePolicy",
        "iam:DeleteRolePolicy",
        "iam:*InstanceProfile"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "VpcAccessForStackOwners",
      "Action": [
        "ec2:Describe*",
        "ec2:CreateTags",
        "ec2:*Vpc",
        "ec2:*InternetGateway",
        "ec2:*RouteTable",
        "ec2:CreateRoute",
        "ec2:DeleteRoute",
        "ec2:*NetworkAcl",
        "ec2:*NetworkAclEntry",
        "ec2:*Subnet",
        "ec2:ReplaceNetworkAclAssociation",
        "ec2:AllocateAddress",
        "ec2:ReleaseAddress",
        "ec2:*NatGateway",
        "ec2:*SecurityGroup",
        "ec2:*SecurityGroupEgress",
        "ec2:*SecurityGroupIngress",
        "ec2:*VpcPeeringConnection"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Ec2AccessForStackOwners",
      "Action": [
        "ec2:DescribeInstance*"
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:ModifyInstanceAttribute"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "OpsworksAccessForStackOwners",
      "Action": [
        "opsworks:*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Route53AccessForStackOwners",
      "Action": [
        "route53:ListHostedZonesByName",
        "route53:CreateHostedZone",
        "route53:ListResourceRecordSets",
        "route53:ChangeResourceRecordSets",
        "route53:GetChange"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```



### Stack Operators

The IAM Policy Permissions necessary to operate/maintain existing Stacks.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "OpsworksAccessForStackOperators",
      "Action": [
        "opsworks:Describe*",
        "opsworks:CreateDeployment",
        "opsworks:UpdateApp",
        "opsworks:UpdateStack",
        "opsworks:StartStack",
        "opsworks:StopStack",
        "opsworks:StartInstance",
        "opsworks:StopInstance",
        "opsworks:SetLoadBasedAutoScaling",
        "opsworks:SetTimeBasedAutoScaling"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Ec2AccessForStackOperators",
      "Action": [
        "ec2:DescribeInstance*"
        "ec2:StartInstances",
        "ec2:StopInstances",
        "ec2:TerminateInstances",
        "ec2:ModifyInstanceAttribute"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```


### Stack Deployers

The IAM Policy Permissions necessary to deploy to existing Stacks.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "OpsworksAccessForStackDeployers",
      "Action": [
        "opsworks:Describe*",
        "opsworks:CreateDeployment",
        "opsworks:UpdateApp",
        "opsworks:UpdateStack"
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Ec2AccessForStackDeployers",
      "Action": [
        "ec2:DescribeInstance*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```


### Stack Readers

The IAM Policy Permissions necessary to view existing Stacks.

```json
{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "OpsworksAccessForStackReaders"
      "Action": [
        "opsworks:Describe*",
      ],
      "Effect": "Allow",
      "Resource": "*"
    },
    {
      "Sid": "Ec2AccessForStackReaders",
      "Action": [
        "ec2:DescribeInstance*"
      ],
      "Effect": "Allow",
      "Resource": "*"
    }
  ]
}
```



<!--- RESOURCE LINKS -->

[Meteor Multiverse]: http://meteor-multiverse.github.io/
[AWS OpsWorks]: http://aws.amazon.com/opsworks/

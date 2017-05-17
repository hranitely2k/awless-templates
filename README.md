[![Build Status](https://api.travis-ci.org/wallix/awless-templates.svg?branch=master)](https://travis-ci.org/wallix/awless-templates)

[Twitter](http://twitter.com/awlessCLI) | [Wiki](https://github.com/wallix/awless/wiki) | [Changelog](https://github.com/wallix/awless/blob/master/CHANGELOG.md#readme)

# awless templates

Repository to collect official, verified and runnable templates for the [awless CLI](https://github.com/wallix/awless)

Here are some non exhaustive [Examples](https://github.com/wallix/awless/wiki/Examples) of what you can do with templates. You can also read more about [awless templates](https://github.com/wallix/awless/wiki/Templates)

## Continuous Integration

On each change all templates are verified & compiled against the latest version of `awless`.

You can run the verification locally with:

    go get github.com/wallix/awless  # if needed
    go test verifyall_test.go -v

# Examples


* [Awless readonly group](#awless-readonly-group)
* [Awless readwrite group](#awless-readwrite-group)
* [Group of instances scaling with CPU consumption](#group-of-instances-scaling-with-cpu-consumption)
* [Create an instance accessible with ssh with a new keypair](#create-an-instance-accessible-with-ssh-with-a-new-keypair)
* [Create an instance with preinstalled awless with completion](#create-an-instance-with-preinstalled-awless-with-completion)
* [Create an instance with preconfigured awless and awless-scheduler](#create-an-instance-with-preconfigured-awless-and-awless-scheduler)
* [Create a classic Kafka infra](#create-a-classic-kafka-infra)
* [Create VPC with a Linux host bastion](#create-vpc-with-a-linux-host-bastion)
* [Attach usual readonly AWS policies (set of permissions) on group](#attach-usual-readonly-aws-policies-(set-of-permissions)-on-group)
* [Create a public network enabling routing from the Internet](#create-a-public-network-enabling-routing-from-the-internet)
* [Create a AWS role with usual readonly policies that applies on a resource](#create-a-aws-role-with-usual-readonly-policies-that-applies-on-a-resource)
* [Create a AWS role with usual readonly policies that applies on a user](#create-a-aws-role-with-usual-readonly-policies-that-applies-on-a-user)
* [Upload Image from local file](#upload-image-from-local-file)
* [Create a user with its SDK/Shell access key](#create-a-user-with-its-sdk/shell-access-key)
* [Create a VPC with its internet routing gateway](#create-a-vpc-with-its-internet-routing-gateway)
* [Two instances Bitami wordpress behind a loadbalancer ](#two-instances-bitami-wordpress-behind-a-loadbalancer-)


### Awless readonly group






 Here we define a group that allow users in that group
 to use the `awless` CLI in a readonly mode (i.e. sync, listing).

 Create group name variable:

```sh
groupName = AwlessReadOnlyPermissionsGroup

```
 Create the group:

```sh
create group name=$groupName

```
 Attach corresponding readonly AWS policies (set of permissions) on group related to the `awless` services:

```sh
attach policy arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSNSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSQSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRoute53ReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AWSLambdaReadOnlyAccess group=$groupName
```


Run it locally with: `awless run repo:awless_readonly_group -v`




### Awless readwrite group






 Here we define a group that allow users in that group to use the `awless` CLI in write mode.

 Create group name variable:

```sh
groupName = AwlessReadWritePermissionsGroup

```
 Create the group:

```sh
create group name=$groupName

```
 Attach corresponding AWS policies (set of permissions) on group related to the `awless` services:

```sh
attach policy arn=arn:aws:iam::aws:policy/AmazonEC2FullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonS3FullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSNSFullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSQSFullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonVPCFullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AutoScalingFullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRDSFullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRoute53FullAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AWSLambdaFullAccess group=$groupName

```
 Note that we keep the IAM access readonly

```sh
attach policy arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess group=$groupName
```


Run it locally with: `awless run repo:awless_readwrite_group -v`




### Group of instances scaling with CPU consumption


*Create an autoscaling group of instances and watch their CPU to dynamically allocate/delete instances when needed.*



**tags**: 
infra, autoscaling



 Create the instances launch configuration

```sh
launchconfig = create launchconfiguration image={instance.image} keypair={instance.keypair} name=scalingLaunchConf type={instance.type}

```
 Create the scalinggroup

```sh
create scalinggroup desired-capacity=2 launchconfiguration=$launchconfig max-size={instance.max-number} min-size={instance.min-number} name=instancesScalingGroup subnets={instance.subnets}

```
 Create a scaling policy to add instances (scale-in) and a scaling policy to remove instances (scale-out)

```sh
adjustmentType = ChangeInCapacity
scalein = create scalingpolicy adjustment-scaling=1 adjustment-type=$adjustmentType name=policy-scaling-in scalinggroup=instancesScalingGroup
scaleout = create scalingpolicy adjustment-scaling=-1 adjustment-type=$adjustmentType name=policy-step-scaling-2 scalinggroup=instancesScalingGroup

```
 metrics statistic functions

```sh
statFunction = Average
alarmThreshold = 75
monitoredMetric = CPUUtilization

```
 Add a monitoring alarm to enable scalein when CPU load is above 75% during 2 * 5 min

```sh
create alarm namespace=AWS/EC2 dimensions=AutoScalingGroupName:instancesScalingGroup evaluation-periods=2 metric=$monitoredMetric name=scaleinAlarm operator=GreaterThanOrEqualToThreshold period=300 statistic-function=$statFunction threshold=$alarmThreshold
attach alarm name=scaleinAlarm action-arn=$scalein

```
 Add a monitoring alarm to enable scaleout when CPU load is below 75% during 2 * 5 min

```sh
create alarm namespace=AWS/EC2 dimensions=AutoScalingGroupName:instancesScalingGroup evaluation-periods=2 metric=$monitoredMetric name=scaleoutAlarm operator=LessThanOrEqualToThreshold period=300 statistic-function=$statFunction threshold=$alarmThreshold
attach alarm name=scaleoutAlarm action-arn=$scaleout
```


Run it locally with: `awless run repo:dynamic_autoscaling_watching_CPU -v`




### Create an instance accessible with ssh with a new keypair




**tags**: 
infra, ssh



 Create a new security group for this instance

```sh
securitygroup = create securitygroup vpc={instance.vpc} description={securitygroup.description} name=ssh-from-internet

```
 Authorize access on port 22 to instances in this security group

```sh
update securitygroup id=$securitygroup inbound=authorize protocol=tcp cidr=0.0.0.0/0 portrange=22

```
 Create a new keypair

```sh
keypair = create keypair name={keypair.name}

```
 Create an instance in this security group accessible with the new keypair

```sh
create instance subnet={instance.subnet} image={instance.image} type={instance.type} keypair=$keypair name={instance.name} count=1 securitygroup=$securitygroup
```


Run it locally with: `awless run repo:instance_ssh -v`




### Create an instance with preinstalled awless with completion




**tags**: 
infra, awless



 role name variable

```sh
roleName = {awless.role-name}

```
 Create a AWS role that applies on a resource

```sh
create role name=$roleName principal-service="ec2.amazonaws.com" sleep-after=10

```
 Attach typical necessary awless readonly permissions to the role

```sh
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSNSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSQSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRoute53ReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AWSLambdaReadOnlyAccess

```
 Launch new instance running remote user data script installing awless

```sh
create instance name=awless-commander type=t2.nano keypair={ssh.keypair} userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/install_awless.yml role=$roleName
```


Run it locally with: `awless run repo:instance_with_awless -v`




### Create an instance with preconfigured awless and awless-scheduler




**tags**: 
infra, awless, awless-scheduler



 Awless scheduler role variable

```sh
roleName = {awless-scheduler.role-name}

```
 First we define a role that an EC2 instance can assume to use awless/awless-scheduler (write mode)

```sh
create role name=$roleName principal-service="ec2.amazonaws.com" sleep-after=10

```
 Attach typical necessary awless readonly permissions to the role

```sh
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonEC2FullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonS3FullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSNSFullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSQSFullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonVPCFullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AutoScalingFullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRDSFullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRoute53FullAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AWSLambdaFullAccess

```
 We keep IAM on read only mode

```sh
attach policy role=$roleName arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess

```
 Launch new instance running remote user data script installing awless

```sh
create instance name=AwlessWithScheduler type=t2.nano keypair={ssh.keypair} userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/install_awless_suite.yml role=$roleName
```


Run it locally with: `awless run repo:instance_with_awless_scheduler -v`




### Create a classic Kafka infra


*Create a classic Kafka infra: brokers, 1 zookeeper instance*





 Create securitygroup for SSH: opening port 22 for all IPs

```sh
sshsecgroup = create securitygroup vpc={main.vpc} description=SSHSecurityGroup name=SSHSecurityGroup
update securitygroup id=$sshsecgroup inbound=authorize protocol=tcp cidr={remote-access.cidr} portrange=22

```
 Create securitygroup for Kafka instances (brokers & zookeeper)

```sh
kafkasecgroup = create securitygroup vpc={main.vpc} description=KafkaSecurityGroup name=KafkaSecurityGroup
update securitygroup id=$kafkasecgroup inbound=authorize protocol=tcp cidr={subnet.cidr} portrange=0-65535

```
 Create a role with policy for ec2 resources so that an instance can list other instances using a local `awless`

```sh
create role name=AwlessEC2ReadonlyRole principal-service="ec2.amazonaws.com" sleep-after=20
attach policy role=AwlessEC2ReadonlyRole arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess

```
 Create Kafka broker instances with role created above

```sh
broker_1 = create instance name=broker_1 image={redhat-ami} type={broker-instance-type} keypair={keypair.name} subnet={main.subnet} role=AwlessEC2ReadonlyRole securitygroup=$sshsecgroup userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/redhat/kafka.sh
broker_2 = create instance name=broker_2 image={redhat-ami} type={broker-instance-type} keypair={keypair.name} subnet={main.subnet} role=AwlessEC2ReadonlyRole securitygroup=$sshsecgroup userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/redhat/kafka.sh
broker_3 = create instance name=broker_3 image={redhat-ami} type={broker-instance-type} keypair={keypair.name} subnet={main.subnet} role=AwlessEC2ReadonlyRole securitygroup=$sshsecgroup userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/redhat/kafka.sh

```
 Create Zookeeper instance

```sh
zookeeper = create instance name=zookeeper image={redhat-ami} type={zookeeper-instance-type} keypair={keypair.name} subnet={main.subnet} securitygroup=$sshsecgroup userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/redhat/zookeeper.sh

```
 Update instances with corresponding securitygroups

```sh
attach securitygroup id=$kafkasecgroup instance=$broker_1
attach securitygroup id=$kafkasecgroup instance=$broker_2
attach securitygroup id=$kafkasecgroup instance=$broker_3
attach securitygroup id=$kafkasecgroup instance=$zookeeper
```


Run it locally with: `awless run repo:kafka_infra -v`


Full CLI example:
```sh
awless run repo:kafka_infra redhat-ami=$(awless search images redhat --id-only) remote-access.cidr=$(awless whoami --ip-only)/32 broker-instance-type=t2.medium zookeeper-instance-type=t2.medium
```



### Create VPC with a Linux host bastion


*This template build this typical Linux bastion [architecture](http://docs.aws.amazon.com/quickstart/latest/linux-bastion/architecture.html) except it only deploys one host bastion on one public subnet*



**tags**: 
infra



 Create a new VPC and make it public with an internet gateway

```sh
vpc = create vpc cidr=10.0.0.0/16 name=BastionVpc
gateway = create internetgateway
attach internetgateway id=$gateway vpc=$vpc

```
 Create 2 private subnets each on a different availability zone
 That is where you will deploy resources only accessible through the bastion

```sh
create subnet cidr=10.0.0.0/19 name=PrivSubnet1 vpc=$vpc availabilityzone={zone1}
create subnet cidr=10.0.32.0/19 name=PrivSubnet2 vpc=$vpc availabilityzone={zone2}

```
 Create the the public subnet hosting the bastion

```sh
pubSubnet = create subnet cidr=10.0.128.0/20 name=PubSubnet1 vpc=$vpc availabilityzone={zone1}
update subnet id=$pubSubnet public=true

```
 Create a route table (with routing only allowed within VPC by default)

```sh
rtable = create routetable vpc=$vpc

```
 Make the public subnet use the route table

```sh
attach routetable id=$rtable subnet=$pubSubnet
create route cidr=0.0.0.0/0 gateway=$gateway table=$rtable

```
 Create the firewall with the remote access CIDR applied on each bastion host

```sh
bastionSecGroup = create securitygroup vpc=$vpc description=BastionSecGroup name=bastion-secgroup
update securitygroup id=$bastionSecGroup inbound=authorize protocol=tcp cidr={remoteaccess-cidr} portrange=22
update securitygroup id=$bastionSecGroup inbound=authorize protocol=icmp cidr={remoteaccess-cidr} portrange=any

```
 Allow only a set of permitted actions for the 2 host bastions

```sh
create role name=BastionHostRole principal-service=ec2.amazonaws.com sleep-after=30
bastionEc2Policy = create policy name=BastionEc2Permissions action=ec2:DescribeAddresses,ec2:AssociateAddress resource="*" effect=Allow
attach policy role=BastionHostRole arn=$bastionEc2Policy

```
 Create one elastic IPs for that will be dynamically aasigned to the host bastion by the bootstrap script

```sh
create elasticip domain=vpc

```
 Create the autoscaling group

```sh
launchConfig = create launchconfiguration image={instance.image} keypair={keypair.name} securitygroups=$bastionSecGroup name=BastionHostsLaunchConfig type=t2.micro role=BastionHostRole userdata=https://raw.githubusercontent.com/wallix/awless-templates/master/userdata/prepare_bastion.yml
create scalinggroup desired-capacity=1 launchconfiguration=$launchConfig max-size=1 min-size=1 name=autoscaling-instances-group subnets=$pubSubnet
```


Run it locally with: `awless run repo:linux_bastion -v`




### Attach usual readonly AWS policies (set of permissions) on group


*When you want your users to have a set of permissions, instead of attaching permissions directly on users it is a good practice and simpler to define a group having those permissions and then adding/removing as needed users from those groups.*



**tags**: 
access, policy, role




```sh
groupName = {group-name}
attach policy arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSNSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonSQSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess group=$groupName
attach policy arn=arn:aws:iam::aws:policy/AmazonRoute53ReadOnlyAccess group=$groupName
```


Run it locally with: `awless run repo:policies_on_role -v`




### Create a public network enabling routing from the Internet




**tags**: 
infra



 Create the subnet

```sh
subnet = create subnet cidr={subnet.cidr} vpc={subnet.vpc} name={subnet.name}

```
 Allow instances in this network to have public IP addresses

```sh
update subnet id=$subnet public=true

```
 Create a route table for this network

```sh
rtable = create routetable vpc={subnet.vpc}
attach routetable id=$rtable subnet=$subnet

```
 Enable routing from the Internet to this subnet

```sh
create route cidr=0.0.0.0/0 gateway={vpc.internetgateway} table=$rtable
```


Run it locally with: `awless run repo:public_subnet -v`




### Create a AWS role with usual readonly policies that applies on a resource


*Create a AWS role that applies on a resource (retrieve the account id with `awless whoami`)*



**tags**: 
access, policy, role




```sh
roleName = {role-name}
create role name=$roleName principal-service={aws-service}

```
 Attach policy (set of permissions) to the created role

```sh
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSNSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonSQSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess
attach policy role=$roleName arn=arn:aws:iam::aws:policy/AmazonRoute53ReadOnlyAccess
```


Run it locally with: `awless run repo:role_for_resource -v`




### Create a AWS role with usual readonly policies that applies on a user


*Create a AWS role that applies on a user (retrieve the id with `awless whoami`)*



**tags**: 
access, policy, user




```sh
accountRole = create role name={role-name} principal-account={aws-account-id}

```
 Attach policy (set of permissions) to the created role

```sh
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonEC2ReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonSNSReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonSQSReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonVPCReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AutoScalingReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/IAMReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonRDSReadOnlyAccess
attach policy role={role-name} arn=arn:aws:iam::aws:policy/AmazonRoute53ReadOnlyAccess

```
 Create a policy to allow user with this policy to assume only this role
 You can then attach this policy to a user via `awless attach policy arn=... user=jsmith`

```sh
create policy name={assume-policy-name} effect=Allow action=sts:AssumeRole resource=$accountRole
```


Run it locally with: `awless run repo:role_for_user -v`




### Upload Image from local file


*This template uploads on s3 a local VM file (VHD, OVA, VMDK). Then it creates an AMI from the S3 object.*



**tags**: 
infra, s3



 Upload the image on s3

```sh
bucket = {image.bucket}
imageObject = create s3object bucket=$bucket file={image.filepath}

```
 Create the AMI from the object on S3

```sh
import image description={image.description} bucket=$bucket s3object=$imageObject
```


Run it locally with: `awless run repo:upload_image -v`




### Create a user with its SDK/Shell access key




**tags**: 
access, user




```sh
create user name={user.name}
create accesskey user={user.name}
```


Run it locally with: `awless run repo:user -v`




### Create a VPC with its internet routing gateway




**tags**: 
infra, VPC




```sh
vpc = create vpc cidr={vpc.cidr} name={vpc.name}
gateway = create internetgateway
attach internetgateway id=$gateway vpc=$vpc
```


Run it locally with: `awless run repo:vpc -v`




### Two instances Bitami wordpress behind a loadbalancer 


*Note that the AMI in this template are working only in eu-central-1 region*



**tags**: 
infra



 Loadbalancer
 Create the loadbalancer firewall

```sh
loadbalancerfw = create securitygroup vpc={wordpress.vpc} description=wordpress-loadbalancer-securitygroup name=wordpress-lb-securitygroup
update securitygroup id=$loadbalancerfw inbound=authorize protocol=tcp cidr=0.0.0.0/0 portrange=80

```
 Create the target group for EC2 wordpress servers

```sh
targetgroup = create targetgroup name=wordpress-workers port=80 protocol=HTTP vpc={wordpress.vpc}

```
 Create the application load balancer that will redirect flows to the servers

```sh
lb = create loadbalancer name=wordpress-loadbalancer subnets={wordpress.subnets} securitygroups=$loadbalancerfw
create listener actiontype=forward loadbalancer=$lb port=80 protocol=HTTP target=$targetgroup

```
 Wordpress application servers
 Create the wordpress servers

```sh
inst1 = create instance subnet={instance1.private.subnet} image=ami-3b36fe54 type={instance.type} count=1 name=wordpress-server-1 # AMI WordPress powered by Bitnami in eu-central-1
inst2 = create instance subnet={instance2.private.subnet} image=ami-3b36fe54 type={instance.type} count=1 name=wordpress-server-2

```
 Register the servers in the targetgroup

```sh
check instance id=$inst1 state=running timeout=180
check instance id=$inst2 state=running timeout=180
attach instance id=$inst1 targetgroup=$targetgroup
attach instance id=$inst2 targetgroup=$targetgroup
```


Run it locally with: `awless run repo:wordpress_ha -v`




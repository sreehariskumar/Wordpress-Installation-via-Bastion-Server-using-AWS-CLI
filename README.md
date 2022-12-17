# Wordpress Installation via Bastion Server using AWS CLI


[![Build Status](https://travis-ci.org/joemccann/dillinger.svg?branch=master)](https://travis-ci.org/joemccann/dillinger)

## Usually, we use Terraform to create a VPC and launch an instance in it. But in this project I'm using AWS CLI to do the same task.

### Requirements
- An IAM user with programmatic access and EC2FullAccess permission

### Features

- Configure IAM user to use AWS CLI
- Create a VPC with 2 public and 1 private subnets
- Create an Internet Gateway & NAT Gateway
- Create a key to login to the server
- Create security groups with custom rules
- Launch bastion server and frontend server in the public subnet and a backend server in private subnet

### Let's get started.

1.  Configure AWS CLI using the below command.

``` s
$ aws configure
AWS Access Key ID [None]:XXXXXXXXXXXX 
AWS Secret Access Key [None]: XXXXXXXXXXXXXXXX
Default region name [None]: us-east-2
Default output format [None]: json
```
2. Create a VPC with a 172.16.0.0/16 CIDR block using the following command, and it returns the ID of the new VPC.

``` s
$ aws ec2 create-vpc --cidr-block 172.16.0.0/16 --query Vpc.VpcId --output text
```
```s
vpc-0d71e52a8c79c4305
```

Add a Name tag to the VPC.
``` s
$ aws ec2 create-tags --resources vpc-0d71e52a8c79c4305 --tags Key=Name,Value=VPC-Project
```

3. Create subnets and add name tags to each subnet.
> Create a subnet in your VPC with a 172.16.0.0/18 CIDR block.  
> Here, I am creating it in the us-east-2a availability zone.  
> If no availability zone is specified, a subnet will be created in any of the availability zones in the us-east-2 region

``` s
$ aws ec2 create-subnet --vpc-id vpc-0d71e52a8c79c4305 --cidr-block 172.16.0.0/18 --availability-zone us-east-2a
```
```s
{
    "Subnet": {
        "AvailabilityZone": "us-east-2a",
        "AvailabilityZoneId": "use2-az1",
        "CidrBlock": "172.16.0.0/18",
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-0fc044a0778f08c84",
        "VpcId": "vpc-0d71e52a8c79c4305",
        "OwnerId": "187450818461"
    }
}

```
``` s
$ aws ec2 create-tags --resources subnet-0fc044a0778f08c84 --tags Key=Name,Value=Subnet1
```
``` s
$ aws ec2 create-subnet --vpc-id vpc-0d71e52a8c79c4305 --cidr-block 172.16.64.0/18 --availability-zone us-east-2b
```
```s
{
    "Subnet": {
        "AvailabilityZone": "us-east-2b",
        "AvailabilityZoneId": "use2-az2",
        "CidrBlock": "172.16.64.0/18",
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-097f2ce951a1588d7",
        "VpcId": "vpc-0d71e52a8c79c4305",
        "OwnerId": "187450818461"
    }
}
```

``` s
$ aws ec2 create-tags --resources subnet-0e9f16987c188ccff --tags Key=Name,Value=Subnet2
```

``` s
$ aws ec2 create-subnet --vpc-id vpc-0d71e52a8c79c4305 --cidr-block 172.16.128.0/18 --availability-zone us-east-2c
```
```s
{
    "Subnet": {
        "AvailabilityZone": "us-east-2c",
        "AvailabilityZoneId": "use2-az3",
        "CidrBlock": "172.16.128.0/18",
        "MapPublicIpOnLaunch": false,
        "State": "available",
        "SubnetId": "subnet-034d5c4ae26958fcb",
        "VpcId": "vpc-0d71e52a8c79c4305"
    }
}
```

``` s
$ aws ec2 create-tags --resources subnet-034d5c4ae26958fcb --tags Key=Name,Value=Subnet3
```

4. View the complete information of the VPC and subnets using the below command. 
```s
$ aws ec2 describe-subnets  --filters "Name=vpc-id,Values=vpc-0d71e52a8c79c4305"
```
```s
{
    "Subnets": [
        {
            "AvailabilityZone": "us-east-2a",
            "AvailabilityZoneId": "use2-az1",
            "CidrBlock": "172.16.0.0/18",
            "MapPublicIpOnLaunch": true,
            "State": "available",
            "SubnetId": "subnet-0fc044a0778f08c84",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461",
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "Subnet1"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-2:187450818461:subnet/subnet-0fc044a0778f08c84"
        },
        {
            "AvailabilityZone": "us-east-2b",
            "AvailabilityZoneId": "use2-az2",
            "CidrBlock": "172.16.64.0/18",
            "MapPublicIpOnLaunch": true,
            "State": "available",
            "SubnetId": "subnet-097f2ce951a1588d7",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461",
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "Subnet2"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-2:187450818461:subnet/subnet-097f2ce951a1588d7"
        },
        {
            "AvailabilityZone": "us-east-2c",
            "AvailabilityZoneId": "use2-az3",
            "CidrBlock": "172.16.128.0/18",
            "MapPublicIpOnLaunch": false,
            "State": "available",
            "SubnetId": "subnet-034d5c4ae26958fcb",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461",
            "Ipv6CidrBlockAssociationSet": [],
            "Tags": [
                {
                    "Key": "Name",
                    "Value": "Subnet3"
                }
            ],
            "SubnetArn": "arn:aws:ec2:us-east-2:187450818461:subnet/subnet-034d5c4ae26958fcb"
        }
    ]
}
```

5. Create an Internet Gateway using the following command, which returns the new Internet Gateway ID.
```s
$ aws ec2 create-internet-gateway --query InternetGateway.InternetGatewayId --output text
```
```s
igw-0776eab2191d55fcd
```

Attach the internet gateway to our VPC to allow the instances(and NAT gateway) to communicate with the outside world.

```s
$ aws ec2 attach-internet-gateway --vpc-id vpc-0d71e52a8c79c4305 --internet-gateway-id igw-0776eab2191d55fcd
```

6. Create a NAT Gateway after allocating an elastic IP using the allocate-address command.

```s
$ aws ec2 allocate-address
```
```s
{
    "PublicIp": "18.218.15.68",
    "AllocationId": "eipalloc-02e2f01ec0811e9c5",
    "PublicIpv4Pool": "amazon",
    "NetworkBorderGroup": "us-east-2",
    "Domain": "vpc"
}
```

7. Create a NAT gateway for the private subnet and assign the elastic IP.

```s
$ aws ec2 create-nat-gateway --subnet-id subnet-034d5c4ae26958fcb --allocation-id eipalloc-02e2f01ec0811e9c5
```
```s
{
    "ClientToken": "7a552c09-ac35-4c17-82d3-dea2574caa88",
    "NatGateway": {
        "CreateTime": "2022-12-16T06:28:53.000Z",
        "NatGatewayAddresses": [
            {
                "AllocationId": "eipalloc-02e2f01ec0811e9c5"
            }
        ],
        "NatGatewayId": "nat-0e858265b8c49214c",
        "State": "pending",
        "SubnetId": "subnet-097f2ce951a1588d7",
        "VpcId": "vpc-0d71e52a8c79c4305"
    }
}
```

8. A route table will be created by default.  We can find the default route table information of the VPC using the below command

```s
$ aws ec2 describe-route-tables --filters="Name=vpc-id,Values=vpc-0d71e52a8c79c4305"
```
```s
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "Main": true,
                    "RouteTableAssociationId": "rtbassoc-00b93b2be2f4c2f6b",
                    "RouteTableId": "rtb-01f50968f2b15d30e",
                    "AssociationState": {
                        "State": "associated"
                    }
                }
            ],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-01f50968f2b15d30e",
            "Routes": [
                {
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461"
        }
    ]
}
```

9. Create a custom route table(for private subnet) using the below command.

```s
$ aws ec2 create-route-table --vpc-id vpc-0d71e52a8c79c4305 --query RouteTable.RouteTableId --output text
```
```s
rtb-07b8bad039aee580f
```

10.  Associate the private subnet with the route table.

```s
 $ aws ec2 associate-route-table  --subnet-id subnet-034d5c4ae26958fcb --route-table-id rtb-07b8bad039aee580f
```
```s
{
    "AssociationId": "rtbassoc-076a30856ccbe1b09",
    "AssociationState": {
        "State": "associated"
    }
}
```

11. Add a route table entry to redirect all the traffic from VPC to pass through the InternetGateway.

```s
$ aws ec2 create-route --route-table-id rtb-01f50968f2b15d30e --destination-cidr-block 0.0.0.0/0 --gateway-id igw-0776eab2191d55fcd
```
```s
{
    "Return": true
}
```

12. Add route table entry in the custom route table to enable instances in the private subnet to communicate with the NAT gateway.

```s
$ aws ec2 create-route --route-table-id rtb-07b8bad039aee580f --destination-cidr-block 0.0.0.0/0 --nat-gateway-id nat-0e858265b8c49214c
```
```s
{
    "Return": true
}
```

13. To confirm that the route has been created and is active, we can describe the route table using the following describe-route-tables command.

```s
$ aws ec2 describe-route-tables --filters "Name=vpc-id,Values=vpc-0d71e52a8c79c4305"
```
```s
{
    "RouteTables": [
        {
            "Associations": [
                {
                    "Main": true,
                    "RouteTableAssociationId": "rtbassoc-00b93b2be2f4c2f6b",
                    "RouteTableId": "rtb-01f50968f2b15d30e",
                    "AssociationState": {
                        "State": "associated"
                    }
                }
            ],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-01f50968f2b15d30e",
            "Routes": [
                {
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "GatewayId": "igw-0776eab2191d55fcd",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461"
        },
        {
            "Associations": [
                {
                    "Main": false,
                    "RouteTableAssociationId": "rtbassoc-076a30856ccbe1b09",
                    "RouteTableId": "rtb-07b8bad039aee580f",
                    "SubnetId": "subnet-034d5c4ae26958fcb",
                    "AssociationState": {
                        "State": "associated"
                    }
                }
            ],
            "PropagatingVgws": [],
            "RouteTableId": "rtb-07b8bad039aee580f",
            "Routes": [
                {
                    "DestinationCidrBlock": "172.16.0.0/16",
                    "GatewayId": "local",
                    "Origin": "CreateRouteTable",
                    "State": "active"
                },
                {
                    "DestinationCidrBlock": "0.0.0.0/0",
                    "NatGatewayId": "nat-0e858265b8c49214c",
                    "Origin": "CreateRoute",
                    "State": "active"
                }
            ],
            "Tags": [],
            "VpcId": "vpc-0d71e52a8c79c4305",
            "OwnerId": "187450818461"
        }
    ]
}
```

14. Modify the VPC attribute to enable public DNS hostname. These commands will run without any output if executed correctly.

``` s
$ aws ec2 modify-vpc-attribute --vpc-id vpc-0d71e52a8c79c4305 --enable-dns-support "{\"Value\":true}"
```
```s
$ aws ec2 modify-vpc-attribute --vpc-id vpc-0d71e52a8c79c4305 --enable-dns-hostnames "{\"Value\":true}"
```



15. Create a security group for bastion server to accept SSH connection

```s
aws ec2 create-security-group --group-name bastion --description "allow 22 from my-ip" --vpc-id vpc-0d71e52a8c79c4305
```
```s
{
    "GroupId": "sg-0ff79d57eb21e754a"
}
```

16. Create a security group for frontend server to accept SSH connection from bstion server and HTTP connection from anywhere.

```s
$ aws ec2 create-security-group --group-name frontend --description "allow 22 from bastion and 80 from all" --vpc-id vpc-0d71e52a8c79c4305
```
```s
{
    "GroupId": "sg-05d1b2f90e1b0c6cb"
}
```

17. Create a security group for backend database server to allow SSH connection from bastion server only.

```s
$ aws ec2 create-security-group --group-name backend --description "allow 22 from bastion and 3306 from frontend" --vpc-id vpc-0d71e52a8c79c4305
```
```s
{
    "GroupId": "sg-039e85b4be2c09627"
}
```

18. Add rules to the security groups.
> Create a security group for Bastion server. Find our public IP by running the following command.
```s
$ curl https://checkip.amazonaws.com
```
```s
49.37.232.13
```

```s
$ aws ec2 authorize-security-group-ingress --group-id sg-0ff79d57eb21e754a --protocol tcp --port 22 --cidr 49.37.232.13/32
```

> We will now create a security group for Frontend server.
```s
$ aws ec2 authorize-security-group-ingress --group-id sg-05d1b2f90e1b0c6cb --protocol tcp --port 80 --cidr 0.0.0.0/0
```
```s
$ aws ec2 authorize-security-group-ingress --group-id sg-05d1b2f90e1b0c6cb --protocol tcp --port 22 --source-group sg-0ff79d57eb21e754a
```
> Finally for the Backend server.

```s
$ aws ec2 authorize-security-group-ingress --group-id sg-039e85b4be2c09627 --protocol tcp --port 3306 --source-group sg-05d1b2f90e1b0c6cb
```
```s
$ aws ec2 authorize-security-group-ingress --group-id sg-039e85b4be2c09627 --protocol tcp --port 22 --source-group sg-0ff79d57eb21e754a
```

19. Create a key pair and save it to a file.  Also, change the permission of the key file to 400.

```s
$ aws ec2 create-key-pair --key-name vpc_key --query 'KeyMaterial' --output text > vpc_key.pem
```

20. Launch the bastion server with the required security group, keypair, subnet, and AMI (Here, I've used Amazon Linux 2 AMI)

```s
$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name vpc_key --security-group-ids sg-0ff79d57eb21e754a --subnet-id subnet-0fc044a0778f08c84
```
```s
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-0e1b896867eee0b15",
            "InstanceType": "t2.micro",
            "KeyName": "vpc_key",
            "LaunchTime": "2022-12-16T06:57:56.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2a",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-25-2.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.25.2",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-0fc044a0778f08c84",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "54c8b59b-2c84-4805-a7be-b0cb96882889",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2022-12-16T06:57:56.000Z",
                        "AttachmentId": "eni-attach-0455985130a26499a",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "bastion",
                            "GroupId": "sg-0ff79d57eb21e754a"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "02:34:d2:a5:52:22",
                    "NetworkInterfaceId": "eni-05e9bbec81344e2f8",
                    "OwnerId": "187450818461",
                    "PrivateDnsName": "ip-172-16-25-2.us-east-2.compute.internal",
                    "PrivateIpAddress": "172.16.25.2",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-16-25-2.us-east-2.compute.internal",
                            "PrivateIpAddress": "172.16.25.2"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-0fc044a0778f08c84",
                    "VpcId": "vpc-0d71e52a8c79c4305",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "bastion",
                    "GroupId": "sg-0ff79d57eb21e754a"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "187450818461",
    "ReservationId": "r-0b74a0ea9c7824f6d"
}
```

21. Launch the frontend server with the required security group, keypair, subnet, and AMI (Here, I've used Amazon Linux 2 AMI)

```s
$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name vpc_key --security-group-ids sg-05d1b2f90e1b0c6cb --subnet-id subnet-097f2ce951a1588d7
```
```s
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-0126b8c1a2e7026e7",
            "InstanceType": "t2.micro",
            "KeyName": "vpc_key",
            "LaunchTime": "2022-12-16T07:00:04.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2b",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-108-190.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.108.190",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-097f2ce951a1588d7",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "a4e6e773-b8db-4bfc-89ae-2dcd50e8fc71",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2022-12-16T07:00:04.000Z",
                        "AttachmentId": "eni-attach-0d6d70755e2b78c44",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "frontend",
                            "GroupId": "sg-05d1b2f90e1b0c6cb"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "06:1b:57:b4:a6:4a",
                    "NetworkInterfaceId": "eni-01588d2257485799e",
                    "OwnerId": "187450818461",
                    "PrivateDnsName": "ip-172-16-108-190.us-east-2.compute.internal",
                    "PrivateIpAddress": "172.16.108.190",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-16-108-190.us-east-2.compute.internal",
                            "PrivateIpAddress": "172.16.108.190"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-097f2ce951a1588d7",
                    "VpcId": "vpc-0d71e52a8c79c4305",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "frontend",
                    "GroupId": "sg-05d1b2f90e1b0c6cb"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "187450818461",
    "ReservationId": "r-0bcb672813647fc72"
}
```

22. Launch the backend server with the required security group, keypair, subnet, and AMI (Here, I’ve used Amazon Linux 2 AMI)

```s
$ aws ec2 run-instances --image-id ami-0beaa649c482330f7 --count 1 --instance-type t2.micro --key-name vpc_key --security-group-ids sg-039e85b4be2c09627 --subnet-id subnet-034d5c4ae26958fcb
```
```s
{
    "Groups": [],
    "Instances": [
        {
            "AmiLaunchIndex": 0,
            "ImageId": "ami-0beaa649c482330f7",
            "InstanceId": "i-05ac5e2c63c4f3dca",
            "InstanceType": "t2.micro",
            "KeyName": "vpc_key",
            "LaunchTime": "2022-12-16T07:01:33.000Z",
            "Monitoring": {
                "State": "disabled"
            },
            "Placement": {
                "AvailabilityZone": "us-east-2c",
                "GroupName": "",
                "Tenancy": "default"
            },
            "PrivateDnsName": "ip-172-16-183-3.us-east-2.compute.internal",
            "PrivateIpAddress": "172.16.183.3",
            "ProductCodes": [],
            "PublicDnsName": "",
            "State": {
                "Code": 0,
                "Name": "pending"
            },
            "StateTransitionReason": "",
            "SubnetId": "subnet-034d5c4ae26958fcb",
            "VpcId": "vpc-0d71e52a8c79c4305",
            "Architecture": "x86_64",
            "BlockDeviceMappings": [],
            "ClientToken": "eacfc952-caaa-4ac8-b3a7-ba775741b8e5",
            "EbsOptimized": false,
            "EnaSupport": true,
            "Hypervisor": "xen",
            "NetworkInterfaces": [
                {
                    "Attachment": {
                        "AttachTime": "2022-12-16T07:01:33.000Z",
                        "AttachmentId": "eni-attach-0e6afc60bc75aa2c2",
                        "DeleteOnTermination": true,
                        "DeviceIndex": 0,
                        "Status": "attaching"
                    },
                    "Description": "",
                    "Groups": [
                        {
                            "GroupName": "backend",
                            "GroupId": "sg-039e85b4be2c09627"
                        }
                    ],
                    "Ipv6Addresses": [],
                    "MacAddress": "0a:4d:4f:3a:53:80",
                    "NetworkInterfaceId": "eni-0d809254626858218",
                    "OwnerId": "187450818461",
                    "PrivateDnsName": "ip-172-16-183-3.us-east-2.compute.internal",
                    "PrivateIpAddress": "172.16.183.3",
                    "PrivateIpAddresses": [
                        {
                            "Primary": true,
                            "PrivateDnsName": "ip-172-16-183-3.us-east-2.compute.internal",
                            "PrivateIpAddress": "172.16.183.3"
                        }
                    ],
                    "SourceDestCheck": true,
                    "Status": "in-use",
                    "SubnetId": "subnet-034d5c4ae26958fcb",
                    "VpcId": "vpc-0d71e52a8c79c4305",
                    "InterfaceType": "interface"
                }
            ],
            "RootDeviceName": "/dev/xvda",
            "RootDeviceType": "ebs",
            "SecurityGroups": [
                {
                    "GroupName": "backend",
                    "GroupId": "sg-039e85b4be2c09627"
                }
            ],
            "SourceDestCheck": true,
            "StateReason": {
                "Code": "pending",
                "Message": "pending"
            },
            "VirtualizationType": "hvm",
            "CpuOptions": {
                "CoreCount": 1,
                "ThreadsPerCore": 1
            },
            "CapacityReservationSpecification": {
                "CapacityReservationPreference": "open"
            },
            "MetadataOptions": {
                "State": "pending",
                "HttpTokens": "optional",
                "HttpPutResponseHopLimit": 1,
                "HttpEndpoint": "enabled"
            }
        }
    ],
    "OwnerId": "187450818461",
    "ReservationId": "r-0be0bf8e9f2c7aebd"
}
```


23. Create Name tags for the instances. For that we need to get the instance Id's of the servers.
```s
$ aws ec2 describe-instances
```
```s
 "InstanceId": "i-0e1b896867eee0b15",
 "InstanceId": "i-0126b8c1a2e7026e7",
 "InstanceId": "i-05ac5e2c63c4f3dca",
 ```
 
 ```s
$ aws ec2 create-tags --resources i-0e1b896867eee0b15 --tags Key=Name,Value=bastion
```
```s
$ aws ec2 create-tags --resources i-0126b8c1a2e7026e7 --tags Key=Name,Value=frontend
```
```s
$ aws ec2 create-tags --resources i-05ac5e2c63c4f3dca --tags Key=Name,Value=backend
```

24. Login to the servers.
```s
user@MyLinux:~$ ssh -i vpc_key.pem ec2-user@ec2-18-222-6-227.us-east-2.compute.amazonaws.com
The authenticity of host 'ec2-18-222-6-227.us-east-2.compute.amazonaws.com (18.222.6.227)' can't be established.
ECDSA key fingerprint is SHA256:vjJl82XBg42pZNiq2a01BBpfkJgspyI3QvdUYfKk8RY.
Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
Warning: Permanently added 'ec2-18-222-6-227.us-east-2.compute.amazonaws.com,18.222.6.227' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-16-25-2 ~]$ 
[ec2-user@ip-172-16-25-2 ~]$ 
[ec2-user@ip-172-16-25-2 ~]$ 
```
Change the hostname of the server to bastion for easy identification.
```s
sudo hostnamectl set-hostname bastion.us-east-2.compute.internal
```
```s
[ec2-user@ip-172-16-25-2 ~]$ sudo hostnamectl set-hostname bastion.us-east-2.compute.internal
[ec2-user@ip-172-16-25-2 ~]$ 
[ec2-user@ip-172-16-25-2 ~]$ logout
user@MyLinux:~$ ssh -i vpc_key.pem ec2-user@ec2-18-222-6-227.us-east-2.compute.amazonaws.com
[ec2-user@bastion ~]$ 
[ec2-user@bastion ~]$
```

We need to copy the key to the bastion server to enable SSH login to frontend and backend servers.

Try login to frontend server from bastion server.

```s
[ec2-user@bastion ~]$ ssh -i vpc_key.pem ec2-user@ec2-3-139-74-168.us-east-2.compute.amazonaws.com
The authenticity of host 'ec2-3-139-74-168.us-east-2.compute.amazonaws.com (172.16.108.190)' can't be established.
ECDSA key fingerprint is SHA256:SPVGususkOPxb5gPd1kHqjfCeSR5Z4FNBLhWCGEVGyo.
ECDSA key fingerprint is MD5:86:8b:e3:47:d8:7e:8c:d2:33:1e:d3:01:50:73:aa:75.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ec2-3-139-74-168.us-east-2.compute.amazonaws.com,172.16.108.190' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@ip-172-16-108-190 ~]$ 
[ec2-user@ip-172-16-108-190 ~]$ 
```

Change the hostname of frontend server for easy identification.
```s
sudo hostnamectl set-hostname frontend.us-east-2.compute.internal
```

```s
[ec2-user@ip-172-16-108-190 ~]$ sudo hostnamectl set-hostname frontend.us-east-2.compute.internal
[ec2-user@ip-172-16-108-190 ~]$ 
[ec2-user@ip-172-16-108-190 ~]$ logout
Connection to ec2-3-139-74-168.us-east-2.compute.amazonaws.com closed.
[ec2-user@bastion ~]$ 
[ec2-user@bastion ~]$ ssh -i vpc_key.pem ec2-user@ec2-3-139-74-168.us-east-2.compute.amazonaws.com
Last login: Fri Dec 16 07:13:59 2022 from ip-172-16-25-2.us-east-2.compute.internal

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

https://aws.amazon.com/amazon-linux-2/
19 package(s) needed for security, out of 38 available
Run "sudo yum update" to apply all updates.
[ec2-user@frontend ~]$ 
[ec2-user@frontend ~]$ 
```

Logout from the frontend server and try login to backend server from bastion server. 
>You need to remember one thing that the backend server will not have a public ip or public DNS name as it is launched in a private subnet.
>You can either use private ip or the private DNS name to SSH into the server.

```s
[ec2-user@bastion ~]$ ssh -i vpc_key.pem ec2-user@ip-172-16-183-3.us-east-2.compute.internal
The authenticity of host 'ip-172-16-183-3.us-east-2.compute.internal (172.16.183.3)' can't be established.
ECDSA key fingerprint is SHA256:94CI7tYo4OKLX2TBfag3M14EMrjsDBF7OzQR+8PZtWk.
ECDSA key fingerprint is MD5:bb:1e:45:a9:69:3b:1c:8c:31:ef:b0:3b:ee:60:6b:5c.
Are you sure you want to continue connecting (yes/no)? yes
Warning: Permanently added 'ip-172-16-183-3.us-east-2.compute.internal,172.16.183.3' (ECDSA) to the list of known hosts.

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

[ec2-user@ip-172-16-183-3 ~]$ 
[ec2-user@ip-172-16-183-3 ~]$ 
```

Change the hostname of backend server as well.
```s
sudo hostnamectl set-hostname backend.us-east-2.compute.internal
```

```s
[ec2-user@ip-172-16-183-3 ~]$ sudo hostnamectl set-hostname backend.us-east-2.compute.internal
[ec2-user@ip-172-16-183-3 ~]$ 
[ec2-user@ip-172-16-183-3 ~]$ logout
Connection to ip-172-16-183-3.us-east-2.compute.internal closed.
[ec2-user@bastion ~]$ 
[ec2-user@bastion ~]$ ssh -i vpc_key.pem ec2-user@ip-172-16-183-3.us-east-2.compute.internal
Last login: Fri Dec 16 07:17:20 2022 from ip-172-16-25-2.us-east-2.compute.internal

       __|  __|_  )
       _|  (     /   Amazon Linux 2 AMI
      ___|\___|___|

[ec2-user@backend ~]$ 
[ec2-user@backend ~]$ 
```


_Hi! I've completed my project. You can now login to the servers and install packages. Feel free to try it and let me know your thoughts on this method for creating a VPC and launching instances using AWS CLI._


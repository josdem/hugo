+++
date = "2017-02-13T15:11:39-06:00"
title = "Launching an EC2 using CloudFormation"
categories = ["techtalk","code"]
tags = ["josdem","techtalks","programming","technology", "AWS"]

+++

AWS CloudFormation is a service that helps you to set up your Amazon Web Services resources, so that you can spend less time managing those resources and more time focusing on your applications that run in AWS. This time I will show you how to create and launch an EC2 instance using CloudFormation and Amazon command line interface.

Fist we need to setp CLI (AWS Command Line Interface), which is a tool that allows you to manage AWS services from the command line and automate them through scripts. To setup please follow this information [CLI](https://www.google.com/aclk?sa=L&ai=DChcSEwj-zMfHho7SAhWNVg0KHYBNBSYYABAA&sig=AOD64_0NVvryse0LAZdxCEjTmcWHSwRgYw&q=&ved=0ahUKEwi2tsHHho7SAhXHbiYKHZEeBpYQ0QwIGA&adurl=)

Once we get the CLI installed you should be able to type this command:

```bash
$ aws help
```


**Working with Stacks**

A stack is a collection of AWS resources that you can manage as a single unit. In other words, you can create, update, or delete a collection of resources by creating, updating, or deleting stacks.

To create a stack you run the `aws cloudformation create-stack command`. You must provide the stack name, the location of a valid template, and any input parameters.

This is an real create stack command line:

```bash
aws cloudformation create-stack --stack-name micro-redhat --template-body file:////home/ec2-user/AWS/micro-redhat.json --parameters ParameterKey=KeyName,ParameterValue=b612
```

Where:

* `file:////home/ec2-user/AWS/micro-redhat.json` is the template file in a JSON format.
* `ParameterKey=KeyName,ParameterValue=b612` b612 is the key pair name


**Working with AWS CloudFormation Templates**

You can author AWS CloudFormation templates in JSON or YAML formats. [Here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/sample-templates-services-us-west-1.html#d0e203908) you can see Amazon EC2 sample templates.

Here is how my Micro RedHat instance template looks like:

```json
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "AWS CloudFormation Template: Create an Amazon EC2 T2 micro instance running the Amazon Linux AMI. The AMI is chosen based on the region in which the stack is run.",
  "Parameters": {
    "KeyName": {
      "Description": "b612",
      "Type": "AWS::EC2::KeyPair::KeyName",
      "ConstraintDescription": "The little prince asteroid key"
    },
    "InstanceType": {
      "Description": "WebServer EC2 instance type",
      "Type": "String",
      "Default": "t2.micro",
      "AllowedValues": [
        "t1.micro",
        "t2.nano",
        "t2.micro",
        "t2.small",
        "t2.medium",
        "t2.large",
        "m1.small",
        "m1.medium",
        "m1.large",
        "m1.xlarge",
        "m2.xlarge",
        "m2.2xlarge",
        "m2.4xlarge",
        "m3.medium",
        "m3.large",
        "m3.xlarge",
        "m3.2xlarge",
        "m4.large",
        "m4.xlarge",
        "m4.2xlarge",
        "m4.4xlarge",
        "m4.10xlarge",
        "c1.medium",
        "c1.xlarge",
        "c3.large",
        "c3.xlarge",
        "c3.2xlarge",
        "c3.4xlarge",
        "c3.8xlarge",
        "c4.large",
        "c4.xlarge",
        "c4.2xlarge",
        "c4.4xlarge",
        "c4.8xlarge",
        "g2.2xlarge",
        "g2.8xlarge",
        "r3.large",
        "r3.xlarge",
        "r3.2xlarge",
        "r3.4xlarge",
        "r3.8xlarge",
        "i2.xlarge",
        "i2.2xlarge",
        "i2.4xlarge",
        "i2.8xlarge",
        "d2.xlarge",
        "d2.2xlarge",
        "d2.4xlarge",
        "d2.8xlarge",
        "hi1.4xlarge",
        "hs1.8xlarge",
        "cr1.8xlarge",
        "cc2.8xlarge",
        "cg1.4xlarge"
      ],
      "ConstraintDescription": "T2 micro 1GB RAM instance type."
    },
    "ProtocolLocation": {
      "Description": "The IP address range that can be used to IpProtocol to the EC2 instances",
      "Type": "String",
      "MinLength": "9",
      "MaxLength": "18",
      "Default": "0.0.0.0/0",
      "AllowedPattern": "(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})\\.(\\d{1,3})/(\\d{1,2})",
      "ConstraintDescription": "must be a valid IP CIDR range of the form x.x.x.x/x."
    }
  },
  "Mappings": {
    "AWSInstanceType2Arch": {
      "t1.micro": {
        "Arch": "PV64"
      },
      "t2.nano": {
        "Arch": "HVM64"
      },
      "t2.micro": {
        "Arch": "HVM64"
      },
      "t2.small": {
        "Arch": "HVM64"
      },
      "t2.medium": {
        "Arch": "HVM64"
      },
      "t2.large": {
        "Arch": "HVM64"
      },
      "m1.small": {
        "Arch": "PV64"
      },
      "m1.medium": {
        "Arch": "PV64"
      },
      "m1.large": {
        "Arch": "PV64"
      },
      "m1.xlarge": {
        "Arch": "PV64"
      },
      "m2.xlarge": {
        "Arch": "PV64"
      },
      "m2.2xlarge": {
        "Arch": "PV64"
      },
      "m2.4xlarge": {
        "Arch": "PV64"
      },
      "m3.medium": {
        "Arch": "HVM64"
      },
      "m3.large": {
        "Arch": "HVM64"
      },
      "m3.xlarge": {
        "Arch": "HVM64"
      },
      "m3.2xlarge": {
        "Arch": "HVM64"
      },
      "m4.large": {
        "Arch": "HVM64"
      },
      "m4.xlarge": {
        "Arch": "HVM64"
      },
      "m4.2xlarge": {
        "Arch": "HVM64"
      },
      "m4.4xlarge": {
        "Arch": "HVM64"
      },
      "m4.10xlarge": {
        "Arch": "HVM64"
      },
      "c1.medium": {
        "Arch": "PV64"
      },
      "c1.xlarge": {
        "Arch": "PV64"
      },
      "c3.large": {
        "Arch": "HVM64"
      },
      "c3.xlarge": {
        "Arch": "HVM64"
      },
      "c3.2xlarge": {
        "Arch": "HVM64"
      },
      "c3.4xlarge": {
        "Arch": "HVM64"
      },
      "c3.8xlarge": {
        "Arch": "HVM64"
      },
      "c4.large": {
        "Arch": "HVM64"
      },
      "c4.xlarge": {
        "Arch": "HVM64"
      },
      "c4.2xlarge": {
        "Arch": "HVM64"
      },
      "c4.4xlarge": {
        "Arch": "HVM64"
      },
      "c4.8xlarge": {
        "Arch": "HVM64"
      },
      "g2.2xlarge": {
        "Arch": "HVMG2"
      },
      "g2.8xlarge": {
        "Arch": "HVMG2"
      },
      "r3.large": {
        "Arch": "HVM64"
      },
      "r3.xlarge": {
        "Arch": "HVM64"
      },
      "r3.2xlarge": {
        "Arch": "HVM64"
      },
      "r3.4xlarge": {
        "Arch": "HVM64"
      },
      "r3.8xlarge": {
        "Arch": "HVM64"
      },
      "i2.xlarge": {
        "Arch": "HVM64"
      },
      "i2.2xlarge": {
        "Arch": "HVM64"
      },
      "i2.4xlarge": {
        "Arch": "HVM64"
      },
      "i2.8xlarge": {
        "Arch": "HVM64"
      },
      "d2.xlarge": {
        "Arch": "HVM64"
      },
      "d2.2xlarge": {
        "Arch": "HVM64"
      },
      "d2.4xlarge": {
        "Arch": "HVM64"
      },
      "d2.8xlarge": {
        "Arch": "HVM64"
      },
      "hi1.4xlarge": {
        "Arch": "HVM64"
      },
      "hs1.8xlarge": {
        "Arch": "HVM64"
      },
      "cr1.8xlarge": {
        "Arch": "HVM64"
      },
      "cc2.8xlarge": {
        "Arch": "HVM64"
      }
    },
    "AWSInstanceType2NATArch": {
      "t1.micro": {
        "Arch": "NATPV64"
      },
      "t2.nano": {
        "Arch": "NATHVM64"
      },
      "t2.micro": {
        "Arch": "NATHVM64"
      },
      "t2.small": {
        "Arch": "NATHVM64"
      },
      "t2.medium": {
        "Arch": "NATHVM64"
      },
      "t2.large": {
        "Arch": "NATHVM64"
      },
      "m1.small": {
        "Arch": "NATPV64"
      },
      "m1.medium": {
        "Arch": "NATPV64"
      },
      "m1.large": {
        "Arch": "NATPV64"
      },
      "m1.xlarge": {
        "Arch": "NATPV64"
      },
      "m2.xlarge": {
        "Arch": "NATPV64"
      },
      "m2.2xlarge": {
        "Arch": "NATPV64"
      },
      "m2.4xlarge": {
        "Arch": "NATPV64"
      },
      "m3.medium": {
        "Arch": "NATHVM64"
      },
      "m3.large": {
        "Arch": "NATHVM64"
      },
      "m3.xlarge": {
        "Arch": "NATHVM64"
      },
      "m3.2xlarge": {
        "Arch": "NATHVM64"
      },
      "m4.large": {
        "Arch": "NATHVM64"
      },
      "m4.xlarge": {
        "Arch": "NATHVM64"
      },
      "m4.2xlarge": {
        "Arch": "NATHVM64"
      },
      "m4.4xlarge": {
        "Arch": "NATHVM64"
      },
      "m4.10xlarge": {
        "Arch": "NATHVM64"
      },
      "c1.medium": {
        "Arch": "NATPV64"
      },
      "c1.xlarge": {
        "Arch": "NATPV64"
      },
      "c3.large": {
        "Arch": "NATHVM64"
      },
      "c3.xlarge": {
        "Arch": "NATHVM64"
      },
      "c3.2xlarge": {
        "Arch": "NATHVM64"
      },
      "c3.4xlarge": {
        "Arch": "NATHVM64"
      },
      "c3.8xlarge": {
        "Arch": "NATHVM64"
      },
      "c4.large": {
        "Arch": "NATHVM64"
      },
      "c4.xlarge": {
        "Arch": "NATHVM64"
      },
      "c4.2xlarge": {
        "Arch": "NATHVM64"
      },
      "c4.4xlarge": {
        "Arch": "NATHVM64"
      },
      "c4.8xlarge": {
        "Arch": "NATHVM64"
      },
      "g2.2xlarge": {
        "Arch": "NATHVMG2"
      },
      "g2.8xlarge": {
        "Arch": "NATHVMG2"
      },
      "r3.large": {
        "Arch": "NATHVM64"
      },
      "r3.xlarge": {
        "Arch": "NATHVM64"
      },
      "r3.2xlarge": {
        "Arch": "NATHVM64"
      },
      "r3.4xlarge": {
        "Arch": "NATHVM64"
      },
      "r3.8xlarge": {
        "Arch": "NATHVM64"
      },
      "i2.xlarge": {
        "Arch": "NATHVM64"
      },
      "i2.2xlarge": {
        "Arch": "NATHVM64"
      },
      "i2.4xlarge": {
        "Arch": "NATHVM64"
      },
      "i2.8xlarge": {
        "Arch": "NATHVM64"
      },
      "d2.xlarge": {
        "Arch": "NATHVM64"
      },
      "d2.2xlarge": {
        "Arch": "NATHVM64"
      },
      "d2.4xlarge": {
        "Arch": "NATHVM64"
      },
      "d2.8xlarge": {
        "Arch": "NATHVM64"
      },
      "hi1.4xlarge": {
        "Arch": "NATHVM64"
      },
      "hs1.8xlarge": {
        "Arch": "NATHVM64"
      },
      "cr1.8xlarge": {
        "Arch": "NATHVM64"
      },
      "cc2.8xlarge": {
        "Arch": "NATHVM64"
      }
    },
    "AWSRegionArch2AMI": {
      "us-east-1": {
        "PV64": "ami-2a69aa47",
        "HVM64": "ami-6869aa05",
        "HVMG2": "ami-648d9973"
      },
      "us-west-2": {
        "PV64": "ami-7f77b31f",
        "HVM64": "ami-7172b611",
        "HVMG2": "ami-09cd7a69"
      },
      "us-west-1": {
        "PV64": "ami-a2490dc2",
        "HVM64": "ami-31490d51",
        "HVMG2": "ami-1e5f0e7e"
      },
      "eu-west-1": {
        "PV64": "ami-4cdd453f",
        "HVM64": "ami-f9dd458a",
        "HVMG2": "ami-b4694ac7"
      },
      "eu-west-2": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-886369ec",
        "HVMG2": "NOT_SUPPORTED"
      },
      "eu-central-1": {
        "PV64": "ami-6527cf0a",
        "HVM64": "ami-ea26ce85",
        "HVMG2": "ami-de5191b1"
      },
      "ap-northeast-1": {
        "PV64": "ami-3e42b65f",
        "HVM64": "ami-374db956",
        "HVMG2": "ami-df9ff4b8"
      },
      "ap-northeast-2": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-2b408b45",
        "HVMG2": "NOT_SUPPORTED"
      },
      "ap-southeast-1": {
        "PV64": "ami-df9e4cbc",
        "HVM64": "ami-a59b49c6",
        "HVMG2": "ami-8d8d23ee"
      },
      "ap-southeast-2": {
        "PV64": "ami-63351d00",
        "HVM64": "ami-dc361ebf",
        "HVMG2": "ami-cbaf94a8"
      },
      "ap-south-1": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-ffbdd790",
        "HVMG2": "ami-decdbab1"
      },
      "us-east-2": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-f6035893",
        "HVMG2": "NOT_SUPPORTED"
      },
      "ca-central-1": {
        "PV64": "NOT_SUPPORTED",
        "HVM64": "ami-730ebd17",
        "HVMG2": "NOT_SUPPORTED"
      },
      "sa-east-1": {
        "PV64": "ami-1ad34676",
        "HVM64": "ami-6dd04501",
        "HVMG2": "NOT_SUPPORTED"
      },
      "cn-north-1": {
        "PV64": "ami-77559f1a",
        "HVM64": "ami-8e6aa0e3",
        "HVMG2": "NOT_SUPPORTED"
      }
    }
  },
  "Resources": {
    "EC2Instance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "InstanceType": {
          "Ref": "InstanceType"
        },
        "SecurityGroups": [
          {
            "Ref": "InstanceSecurityGroup"
          }
        ],
        "KeyName": {
          "Ref": "KeyName"
        },
        "ImageId": {
          "Fn::FindInMap": [
            "AWSRegionArch2AMI",
            {
              "Ref": "AWS::Region"
            },
            {
              "Fn::FindInMap": [
                "AWSInstanceType2Arch",
                {
                  "Ref": "InstanceType"
                },
                "Arch"
              ]
            }
          ]
        },
        "UserData":{ "Fn::Base64" : { "Fn::Join" : ["", [
          "#!/bin/bash \n",
          "yum install -y git"
        ]]}}
      }
    },
    "InstanceSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription": "Enable access Inbound to the EC2Instance",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": "22",
            "ToPort": "22",
            "CidrIp": {
              "Ref": "ProtocolLocation"
            }
          },
          {
            "IpProtocol": "tcp",
            "FromPort": "80",
            "ToPort": "80",
            "CidrIp": {
              "Ref": "ProtocolLocation"
            }
          }
        ]
      },
      "Metadata": {
        "AWS::CloudFormation::Designer": {
          "id": "7d788cc2-ca7f-41c3-ac5e-e0abb39d8e8c"
        }
      }
    }
  },
  "Outputs": {
    "InstanceId": {
      "Description": "InstanceId of the newly created EC2 instance",
      "Value": {
        "Ref": "EC2Instance"
      }
    },
    "AZ": {
      "Description": "Availability Zone of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": [
          "EC2Instance",
          "AvailabilityZone"
        ]
      }
    },
    "PublicDNS": {
      "Description": "Public DNSName of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": [
          "EC2Instance",
          "PublicDnsName"
        ]
      }
    },
    "PublicIP": {
      "Description": "Public IP address of the newly created EC2 instance",
      "Value": {
        "Fn::GetAtt": [
          "EC2Instance",
          "PublicIp"
        ]
      }
    }
  },
  "Metadata": {
    "AWS::CloudFormation::Designer": {
      "7d788cc2-ca7f-41c3-ac5e-e0abb39d8e8c": {
        "size": {
          "width": 60,
          "height": 60
        },
        "position": {
          "x": 60,
          "y": 90
        },
        "z": 1,
        "embeds": []
      },
      "c089c0be-f05d-4d5a-a9d0-323935f6a33b": {
        "size": {
          "width": 60,
          "height": 60
        },
        "position": {
          "x": 180,
          "y": 90
        },
        "z": 1,
        "embeds": [],
        "ismemberof": [
          "7d788cc2-ca7f-41c3-ac5e-e0abb39d8e8c"
        ]
      }
    }
  }
}
```

**hints**

* `InstanceSecurityGroup` Creates an Amazon EC2 security group. Here we are defining 22 and 80 inbound rules, for more information about Security Groups go [Here](http://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-ec2-security-group.html)
* `UserData` property runs a shell command in order to install last git version

That's it, to launch your EC2 instance execute this command from terminal:

```bash
aws cloudformation create-stack --stack-name micro-redhat --template-body file:////home/ec2-user/AWS/micro-redhat.json --parameters ParameterKey=KeyName,ParameterValue=b612
```

**Where**

* `--parameters` A list of parameter structure that specify input key and value for the stack. Go [here](http://docs.aws.amazon.com/AWSCloudFormation/latest/APIReference/API_Parameter.html) for more information.
* In this case `b612` is the key pair name, the key I need in order to connect to my EC2 instance.


[Return to the main article](/techtalk/sysadmin)

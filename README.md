# cloudformating 
{
  "AWSTemplateFormatVersion": "2010-09-09",
  "Description": "",
  "Metadata": {},
  "Parameters": {
    "MyAppVpcCidr": {
      "Description": "Choose Cidr for Vpc",
      "Type": "String",
      "Default": "10.0.0.0/24"
    },
    "Subnet1Cidr": {
      "Description": "Choose Cidr for Subnet1",
      "Type": "String",
      "Default": "10.0.0.0/25"
    },
    "Subnet2Cidr": {
      "Description": "Choose Cidr for Subnet1",
      "Type": "String",
      "Default": "10.0.0.128/25"
    },
    "Subnet1AZs": {
      "Description": "Choose AZ for Subnet1",
      "Type": "AWS::EC2::AvailabilityZone::Name"
    },
    "Subnet2AZs": {
      "Description": "Choose AZ for Subnet1",
      "Type": "AWS::EC2::AvailabilityZone::Name"
    },
    "WebserverKeyName": {
      "Description": "Choose Key Name",
      "Type": "AWS::EC2::KeyPair::KeyName"
    },
    "WebserverInstanceType": {
      "Description": "Choose Instance Type",
      "Type": "String",
      "Default": "t2.micro",
      "AllowedValues": [
        "t2.micro",
        "t2.nano",
        "t2.small"
      ]
    },
    "WebserverAMIID": {
      "Description": "Add AMI for webserver",
      "Type": "String"
    }
  },
  "Mappings": {},
  "Conditions": {},
  "Resources": {
    "MyAppVpc": {
      "Type": "AWS::EC2::VPC",
      "Properties": {
        "CidrBlock": {
          "Ref": "MyAppVpcCidr"
        },
        "InstanceTenancy": "default",
        "Tags": [
          {
            "Key": "Name",
            "Value": "MyAppVpc"
          }
        ]
      }
    },
    "Subnet1": {
      "Type": "AWS::EC2::Subnet",
      "Properties": {
        "MapPublicIpOnLaunch": true,
        "AvailabilityZone": {
          "Ref": "Subnet1AZs"
        },
        "VpcId": {
          "Ref": "MyAppVpc"
        },
        "CidrBlock": {
          "Ref": "Subnet1Cidr"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "Subnet1"
          }
        ]
      }
    },
    "Subnet2": {
      "Type": "AWS::EC2::Subnet",
      "Properties": {
        "MapPublicIpOnLaunch": true,
        "AvailabilityZone": {
          "Ref": "Subnet2AZs"
        },
        "VpcId": {
          "Ref": "MyAppVpc"
        },
        "CidrBlock": {
          "Ref": "Subnet2Cidr"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "Subnet2"
          }
        ]
      }
    },
    "MyAppIGW": {
      "Type": "AWS::EC2::InternetGateway",
      "Properties": {
        "Tags": [
          {
            "Key": "Name",
            "Value": "MyAppIGW"
          }
        ]
      }
    },
    "AttachGateway": {
      "Type": "AWS::EC2::VPCGatewayAttachment",
      "Properties": {
        "VpcId": {
          "Ref": "MyAppVpc"
        },
        "InternetGatewayId": {
          "Ref": "MyAppIGW"
        }
      }
    },
    "PublicSubnetRT": {
      "Type": "AWS::EC2::RouteTable",
      "Properties": {
        "VpcId": {
          "Ref": "MyAppVpc"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "PublicSubnetRT"
          }
        ]
      }
    },
    "RouteIGW": {
      "Type": "AWS::EC2::Route",
      "Properties": {
        "RouteTableId": {
          "Ref": "PublicSubnetRT"
        },
        "DestinationCidrBlock": "0.0.0.0/0",
        "GatewayId": {
          "Ref": "MyAppIGW"
        }
      }
    },
    "Subnet1RouteTableAssoc": {
      "Type": "AWS::EC2::SubnetRouteTableAssociation",
      "Properties": {
        "SubnetId": {
          "Ref": "Subnet1"
        },
        "RouteTableId": {
          "Ref": "PublicSubnetRT"
        }
      }
    },
    "Subnet2RouteTableAssoc": {
      "Type": "AWS::EC2::SubnetRouteTableAssociation",
      "Properties": {
        "SubnetId": {
          "Ref": "Subnet2"
        },
        "RouteTableId": {
          "Ref": "PublicSubnetRT"
        }
      }
    },
    "WebServersSecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupName": "WebServersSecurityGroup",
        "GroupDescription": "SG created for web servers",
        "SecurityGroupIngress": [
          {
            "IpProtocol": "tcp",
            "FromPort": "80",
            "ToPort": "80",
            "CidrIp": "0.0.0.0/0"
          },
          {
            "IpProtocol": "tcp",
            "FromPort": "22",
            "ToPort": "22",
            "CidrIp": "0.0.0.0/0"
          }
        ],
        "VpcId": {
          "Ref": "MyAppVpc"
        },
        "Tags": [
          {
            "Key": "Name",
            "Value": "WebServersSecurityGroup"
          }
        ]
      }
    },
    "Webserver1": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "KeyName": {
          "Ref": "WebserverKeyName"
        },
        "SubnetId": {
          "Ref": "Subnet1"
        },
        "ImageId": {
          "Ref": "WebserverAMIID"
        },
        "InstanceType": {
          "Ref": "WebserverInstanceType"
        },
        "Monitoring": "false",
        "SecurityGroupIds": [
          {
            "Ref": "WebServersSecurityGroup"
          }
        ],
        "Tags": [
          {
            "Key": "Name",
            "Value": "Webserver1"
          }
        ],
        "UserData": {
          "Fn::Base64": {
            "Fn::Join": [
              "",
              [
                "#!/bin/bash -ex",
                "-"
              ]
            ]
          }
        }
      }
    }
  },
  "Outputs": {}
}

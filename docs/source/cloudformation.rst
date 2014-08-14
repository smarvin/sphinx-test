==============
CloudFormation
==============

This topic describes the Eucalyptus implementation of the AWS CloudFormation web service. This includes how CloudFormation works and some details and examples of how to add CloudFormation to your Eucalyptus deployment.

Overview
________
Virtualization technology, together with cloud computing, allows for application repeatability and redundancy. You can spin up as many virtual machines as you need, application configuration needs to happen only when virtual images are created. CloudFormation takes this concept to the next level because it allows you to configure an entire set of resources (instances, security groups, user roles and policies, and more) in a single JSON template file, and executed with a single command. This means that you get not just machine repeatability, but environment repeatability. CloudFormation allows you to clone environments in different cloud setups, as well as giving applications the ability to be set up and torn down in a repeatable manner.

How CloudFormtion Works
_______________________
CloudFormation manages a set of resources, called a stack, in batch operations (create, update, or delete). Stacks are described in JSON templates, and can be simple, as the following example:

..  codeblock:: javascript
{
  "Resources" : {
    "MyInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId" : "emi-db0b2276"
      }
    }
  }
}

This stack creates a single instance, based on the image with ID emi-db0b2276. However, this stack is not portable because different clouds might have different image IDs.

CloudFormation allows stack customization through user parameters that are passed in at stack creation time. The following is an example of the template above with a user parameter called MyImageId. Changes are in bold.

.. codeblock:: javascript
{
  "Parameters": {
    "MyImageId": {
      "Description":"Image id",
      "Type":"String"
    }
  },
  "Resources" : {
    "MyInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId" : { "Ref" : "MyImageId" }
      }
    }
  }
}

This stack creates a single instance, but the image ID will be required to be passed in using the command line. For example, the following example uses the euform-create-stack command in Euca2ools:

.. code-block::
euform-create-stack --template-file template.json -p MyImageId=emi-db0b2276 MyStack

This command passes the parameter MyImageId with value emi-db0b2276 into the stack creation process using the -p flag.

You can also use templates to create multiple resources and associate them with each other. For example, the following template creates an instance with its own security group and ingress rule.

.. codeblock:: javascript
{
  "Parameters": {
    "MyImageId": {
      "Description":"Image id",
      "Type":"String"
    }
  },
  "Resources" : {
    "MySecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription" : "Security Group with Ingress Rule for MyInstance",
        "SecurityGroupIngress" : [
          {
            "IpProtocol" : "tcp",
            "FromPort" : "22",
            "ToPort" : "22",
            "CidrIp" : "0.0.0.0/0"
          }
        ]
      }
    },
    "MyInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId" : { "Ref":"MyImageId" },
        "SecurityGroups" : [ 
          { "Ref" : "MySecurityGroup" } 
        ]
      }
    }
  }
}

Templates can be more complicated than the ones shown above, but CloudFormation allows many resources to be deployed in one operation. Resources from most Eucalyptus services are supported.

Requirements
____________

To run CloudFormation on Eucalyptus, you need the following:
  * A running Eucalyptus cloud, version 4.0 or later, with at least one Cloud Controller, Node Controller, and Cluster Controller up, running and registered
  * At least one active running service of each of the following: CloudWatch, AutoScaling, Load Balancing, Compute, and IAM
  * A registered active CloudFormation service
  
Supported Resources
___________________
The following resources are supported by CloudFormation in Eucalyptus.

+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| Resource                                 | Description                                                                                                                                                                                                                                                                                                                                                                            |
+==========================================+========================================================================================================================================================================================================================================================================================================================================================================================+
| AWS::AutoScaling::AutoScalingGroup       | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: HealthCheckType, Tags, and VpcZoneIdentifier.                                                                                                                                                                                                                              |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::AutoScaling::LaunchConfiguration    | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except AssociatePublicIpAddress.                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::AutoScaling::ScalingPolicy          | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::CloudFormation::WaitCondition       | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::CloudFormation::WaitConditionHandle | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::CloudWatch::Alarm                   | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::EIP                            | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except Domain.                                                                                                                                                                                                                                                                     |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::EIPAssociation                 | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: AllocationId, NetworkInterfaceId, and PrivateIpAddress.                                                                                                                                                                                                                    |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::Instance                       | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: NetworkInterfaces, SecurityGroupIds, SourceDestCheck, Tags, and Tenancy. All other properties are forwarded to the Compute service internally but VPC is not implemented so VPC oriented properties are likely ignored there.                                              |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::SecurityGroup                  | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: SecurityGroupEgress, Tags, and VpcId.                                                                                                                                                                                                                                      |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::SecurityGroupIngress           | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except SourceSecurityGroupId.                                                                                                                                                                                                                                                      |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::Volume                         | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: HealthCheckType and Tags.                                                                                                                                                                                                                                                  |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::EC2::VolumeAttachment               | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::ElasticLoadBalancing::LoadBalancer  | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except: AccessLoggingPolicy, ConnectionDrainingPolicy, CrossZone, Policies.InstancePorts, and Policies.LoadBalanerPorts. All other properties are passed through to the LoadBalancing service internally but some features are not implemented so properties may be ignored there. |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::AccessKey                      | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported except Serial.                                                                                                                                                                                                                                                                     |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::Group                          | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::InstanceProfile                | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::Policy                         | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::Role                           | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::User                           | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::IAM::UserToGroupAddition            | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+
| AWS::S3::Bucket                          | All properties in the Template Reference section of the AWS CloudFormation User Guide are supported.                                                                                                                                                                                                                                                                                   |
+------------------------------------------+----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------+

Use Case
========

This topic describes a use case for creating a stack, checking the stack progress, and deleting the stack.

For this use case, we will use the following template:

  ..codeblock:: JSON
{
  "Parameters": {
    "MyImageId": {
      "Description":"Image id",
      "Type":"String"
    },
    "MyKeyPair": {
      "Description":"Key Pair",
      "Type":"String"
    }
  },
  "Resources" : {
    "MySecurityGroup": {
      "Type": "AWS::EC2::SecurityGroup",
      "Properties": {
        "GroupDescription" : "Security Group with Ingress Rule for MyInstance",
        "SecurityGroupIngress" : [
          {
            "IpProtocol" : "tcp",
            "FromPort" : "22",
            "ToPort" : "22",
            "CidrIp" : "0.0.0.0/0"
          }
        ]
      }
    },
    "MyInstance": {
      "Type": "AWS::EC2::Instance",
      "Properties": {
        "ImageId" : { "Ref":"MyImageId" },
        "SecurityGroups" : [ 
          { "Ref" : "MySecurityGroup" } 
        ],
        "KeyName" : { "Ref" : "MyKeyPair" }
      }
    }
  }
}

This template creates an instance with a security group that allows global SSH access (port 22), but uses a keypair to log in. It requires two parameters, MyImageId, which is the image ID of the instance to create, and MyKeyPair, which is the name of the keypair to use to log in with. You can use both values with the euca-run-instances command to create an instance manually (for example, euca-run-instances -k mykey emi-db0b2276) so the arguments needed here are standard instance arguments.

The steps to run this template through the system are explained in the following steps.

.. important:: 
These steps require that you have an available image (run euca-describe-images to verify) and that the CloudFormation service is running (run euca-describe-services to verify).




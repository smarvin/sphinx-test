CloudFormation
==============

This topic describes the Eucalyptus implementation of the AWS CloudFormation web service. This includes how CloudFormation works and some details and examples of how to add CloudFormation to your Eucalyptus deployment.

Overview
________
Virtualization technology, together with cloud computing, allows for application repeatability and redundancy. You can spin up as many virtual machines as you need, application configuration needs to happen only when virtual images are created. CloudFormation takes this concept to the next level because it allows you to configure an entire set of resources (instances, security groups, user roles and policies, and more) in a single JSON template file, and executed with a single command. This means that you get not just machine repeatability, but environment repeatability. CloudFormation allows you to clone environments in different cloud setups, as well as giving applications the ability to be set up and torn down in a repeatable manner.

How CloudFormtion Works
_______________________
CloudFormation manages a set of resources, called a stack, in batch operations (create, update, or delete). Stacks are described in JSON templates, and can be simple, as the following example:

..  codeblock:: xml
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

.. codeblock:: xml
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

.. codeblock::
euform-create-stack --template-file template.json -p MyImageId=emi-db0b2276 MyStack

This command passes the parameter MyImageId with value emi-db0b2276 into the stack creation process using the -p flag.

You can also use templates to create multiple resources and associate them with each other. For example, the following template creates an instance with its own security group and ingress rule.

..codeblock::
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
}d

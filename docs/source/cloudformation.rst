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

# Least-privilege access brief

## The policy shape

The IAM policy should allow the support engineer to use `ec2:RebootInstances`, but only on EC2 instances that have the tag `env=staging`. 
The permission should be scoped by the tag instead of relying only on the resource ARN. 
The engineer should not be allowed to reboot production or other EC2 instances, and should not have other EC2 actions that are not needed for the job. 
This follows the principle of least privilege because the engineer only gets the permission needed to perform the required task.

## Why not full EC2 access

Giving the engineer `AmazonEC2FullAccess` would give them much broader permissions than they need. 
They would be able to perform many EC2 actions instead of only rebooting the staging instances. 
This violates the principle of least privilege and increases the potential impact if the engineer's credentials 
or access were compromised because an attacker could use the broader permissions to make changes to more EC2 resources.

## Network boundary

I would use routing to make sure the engineer's tooling has a network path to the staging environment without also providing a path to production. 
This adds another layer of control because IAM controls what the engineer can do, while the network boundary controls which environments the tooling can reach.

# AWS Security Audit

## Finding 1 — Root account

### What's wrong

The root account is being used daily for accessing the AWS account and MFA is not enabled.

### Why it matters

This is the highest access level in an AWS account and can perform important account-level actions like changing payment information, deleting the account, and so on. This should rarely be used because it has broad access and can control almost everything in the account. If the account is breached, an attacker could use this access to make major changes or run up resources quickly. The fact that MFA is not enabled also makes the risk greater because an attacker who gets the root credentials would not have to pass an additional authentication step.

### What to do instead

The root account should not be used for daily work. Create users, groups, or roles and use IAM to give them access to what they need to perform their required job and nothing more. Also enable MFA for the root account to add another layer of security.

## Finding 2 — Public GitHub access key

### What's wrong

An AWS access key is stored in a deployment script in a public GitHub repository and has been exposed since 2024.

### Why it matters

The access key is already exposed and someone with malicious intent can use it to access AWS resources or run up costs on resources that will be billed to us.

### What to do instead

First, we need to deactivate or revoke the exposed key and investigate whether it has been used. If we still need programmatic access, we should replace it with a safer authentication method and make sure credentials are not included in code that will be committed to GitHub. When possible, we should use an IAM role instead of a long-lived access key.

## Finding 3 — EC2 application access key

### What's wrong

The EC2 application can read from S3 using an access key stored in its configuration file.

### Why it matters

This is creating permanent access for something that only needs temporary access. If the access key is exposed or compromised, someone could use it outside the EC2 instance and potentially access AWS resources that the application has permission to use.

### What to do instead

The better approach is to create an IAM role with only the permissions the application needs and attach the role to the EC2 instance. This allows the application to use temporary credentials instead of storing a permanent access key in its configuration.

## Finding 4 — Unencrypted S3 customer records

### What's wrong

The S3 bucket holding the customer records has no encryption configured.

### Why it matters

If the customer records are exposed, the data would not have the additional protection provided by encryption at rest. Encryption helps protect the stored data from being easily understood without the required key.

Encryption does not replace access controls. IAM and S3 permissions determine who can access the data, while encryption provides another layer of protection for the data when it is stored.

### What to do instead

Enable encryption for the S3 bucket. We can use SSE-S3 where AWS manages the encryption keys, or SSE-KMS where we have more control over who can use the encryption key.

## Finding 5 — CloudTrail status unknown

### What's wrong

Nobody knows whether CloudTrail is switched on.

### Why it matters

That means if something goes wrong, it will be difficult to see who or what did the action that led to the problem, when they did it, and whether the action was allowed or denied.

### What to do instead

Enable and appropriately configure CloudTrail so account activity can be recorded and investigated when necessary.

## Priority

1. Root account — Because this has the highest level of access in the account, and using it daily without MFA increases the potential impact if the credentials are compromised.

2. Public GitHub access key — Because the credential is already exposed and someone else could potentially use it to access resources or run up costs that will be billed to us.

3. CloudTrail status unknown — Because we need to know what is happening in the account and have a way to investigate suspicious or unexpected actions.

4. EC2 application access key — Because we don't need to give permanent access to something that only needs temporary access.

5. Unencrypted S3 customer records — Because encryption provides an additional layer of protection for customer data while it is stored.

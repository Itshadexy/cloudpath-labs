# Week 1 Security Review

## What was wrong

The original policy allowed every action on every resource because both `Action` and `Resource` were set to `*`. 
This gave the application much more access than it needed to read files from one S3 bucket. 
And that violates the principle of least privilege, which means giving an identity only the permissions it needs to perform its required job.

## What I changed

I changed the policy to allow only `s3:GetObject` and restricted the resource to objects inside `my-training-bucket`. 
The application can therefore read the required objects without receiving unnecessary permissions.

## Why it matters

A policy with unnecessarily broad permissions can increase the impact of compromised credentials or a compromised application. 
Limiting permissions to what is actually required reduces unnecessary access and helps protect AWS resources.

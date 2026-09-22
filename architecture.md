# Architecture Decision Record

## Decision 1 — Photo storage

### Context

Users upload photos that are viewed many times a day when they are new. 
Photos older than a year are almost never viewed, but the company still needs to keep them. 
The company is also very cost-sensitive, so keeping all photos in the most expensive storage class would not be ideal.

### Decision

I will store the photos in Amazon S3 using S3 Standard when they are new. 
I will use an S3 lifecycle rule to move them to S3 Standard-IA after 90 days and then to S3 Glacier after 365 days.

### Rationale

S3 Standard is appropriate when the photos are being accessed frequently and need immediate retrieval. 
After 90 days, I expect the access frequency to have reduced, so Standard-IA can reduce storage costs while still providing immediate retrieval when a photo is needed. 
After one year, the photos are almost never viewed, so Glacier is more appropriate for long-term, lower-cost storage.

I rejected keeping all photos in S3 Standard because the older photos are rarely accessed and would continue using a more expensive storage class than necessary.

### Consequences

This reduces storage costs as photos become less frequently accessed, but older photos may have retrieval costs or longer retrieval times depending on their storage class. 
The lifecycle rule also means the transitions happen automatically instead of requiring someone to move the photos manually.

## Decision 2 — Shared web-server templates

### Context

The web application runs on two EC2 instances behind a load balancer, and both instances need to read the same set of site templates.

### Decision

I will use Amazon EFS to provide a shared filesystem that both EC2 instances can access.

### Rationale

EFS is appropriate because multiple EC2 instances need to access the same files at the same time. 
It provides a shared filesystem that both web servers can mount and use.

EBS is the obvious alternative because it provides block storage that behaves like a disk for an EC2 instance. 
However, EBS does not fit this requirement because the two web servers need shared access to the same filesystem rather than separate disks.

S3 would also not be the best fit because the web servers need shared filesystem access to the templates rather than simply retrieving objects from a bucket.

### Consequences

Both web servers can access the same templates, so the application does not need separate copies on each server. 
The trade-off is that EFS is being used for shared storage, which is more than is needed when only one machine needs the files.

## Decision 3 — EC2 pricing

### Context

The application has a steady and predictable workload on weekdays, but traffic roughly triples during a monthly promotion. 
The promotion date is known in advance, and the application is customer-facing, so the additional capacity needs to be reliable.

### Decision

I will use Reserved Instances for the predictable baseline workload. 
For the monthly promotion, I would use reliable non-Spot capacity rather than depending on Spot Instances because the promotional workload is customer-facing and cannot tolerate an unexpected interruption.

### Rationale

The weekday workload is steady and predictable, making a long-term commitment through Reserved Instances appropriate.

I would not use Spot Instances for the promotional capacity because AWS can interrupt Spot Instances when it needs the capacity. 
A customer-facing promotion should not depend on capacity that can be taken away unexpectedly.

The promotion is also known in advance, so the additional capacity can be planned for ahead of time rather than treated as an unpredictable workload.

### Consequences

Using Reserved Instances for the baseline can reduce the cost of predictable compute, but it involves a commitment. 
Using reliable non-Spot capacity for the promotion avoids relying on interruptible instances, but the temporary increase in capacity will cost more than using Spot Instances.

## Lifecycle

* Day 0: Photos are stored in S3 Standard.
* Day 90: Photos transition to S3 Standard-IA.
* Day 365: Photos transition to S3 Glacier.

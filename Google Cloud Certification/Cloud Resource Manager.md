* Quotas
* IAM
* Billing

Target:
- How is our resources organized?
- What we are paying for?
- Who has access to what?
- What is happening to our Cloud resources?


# Google Cloud Resource Hierarchy

-  Hierarchy of Ownership is via parent/child relationship. *Child will inherit all the permissions of the parent.*
- Similar to the traditional file system:
	- Each child will have only ONE parent
	- Permissions are inherited from Top to Down (automatically inherited)
- More permissive Parent Policy always overrules more Restrictive Child Policy

➡️ Primary Content that governs it is - ***IAM***

***Hierarchy model***
- Organizations (root Node) - each account has one organization
- Folders (Optional)
- Projects (25 Project limit for Single Account)
- Inside the project, create Inside GCP resources:
	- Compute Engines, Network, Storage, etc,...

> One organization have only one Billing Account.


# Cloud IAM (Access and Identity Management)
- IAM is applicable from the very top of your Google cloud G Suite to the bottom of your services (even a folder in the cloud storage) ***for the monitoring purpose***
- Monitor all the cloud resources using the ***stack driver***.
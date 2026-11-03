#Deployment Journal (Minimal-Cost Build)

Datadog, CloudTrail, the secret, the IAM policy and GuardDuty do not need a
cluster, so they are done first; the cluster is created last, used, and deleted
in one sitting.

**to begin session, run the commands below**

>cd ~/projects/aws_eks-sec_project
>export AWS_PROFILE=LabAD
>export AWS_REGION=us-east-1
>aws sts get-caller-identity
>git status -sb

Check that the identity is still your account name, and that your working tree is clean or contains only what you expect. Variables like SECRET_ARN and POLICY_ARN don’t survive a new terminal, so fetch them again when you need them.



##IDENTITY

Identity -  ```arn:aws:iam::************:user/Emma-Admin```
"Used an admin user because EKS creates IAM roles and CloudFormation stacks;the pod will get much smaller role."

Policy Name - SecretReadPolicy
To change a policy you create a new version (create-policy-version),
and set it as default. 

That gives you an audit trail of who changed what.

## Stage 1 - Account-level controls (no cluster, near-zero cost)

### 1.1 Dummy secret + least-privilege policy
```bash
scripts/create-secret-and-policy.sh
source scripts/env.sh        # loads SECRET_ARN and POLICY_ARN from inventory.local.env
```
The policy allows `GetSecretValue`/`DescribeSecret` on that one secret ARN only.

##CloudTrail Name - security-lab-trail

##S3 Bucket Name - aws-cloudtrail-logs-*************-GreGG017E


### 1.2 CloudTrail (console)
Create trail `security-lab-trail`:
- New dedicated S3 bucket (leave Block Public Access on).
- **SSE-KMS: off** (S3-managed encryption is enough here; a CMK adds cost).
- Log file validation: **on**.
- CloudWatch Logs: **off**.
- Events: **Management, Read + Write** only. No data events, network events or Insights.

Console-created trails are **multi-Region**.

### 1.3 CloudTrail Log Anaysis

| Field Name | Redacted Value | What It Tells an Analyst |
| :--- | :--- | :--- |
| **eventName** | `CreateOAuth2Token` | Identifies the specific action which is creating an authentication token for access. |
| **eventTime** | `2026-11-03T22:11:57Z` | Establishes the precise timestamp in UTC for timeline correlation. |
| **userIdentity.arn** | `arn:aws:iam::[REDACTED]:user/Tanma-Admin` | Proves exactly which user identity or role initiated the API call. |
| **sourceIPAddress** | `105.127.xx.xx` | Identifies the network origin or IP location where the request originated. |
| **userAgent** | `aws-cli/2.38.4 md/awscrt#0.38.0 ua/2.1 os/linux...` | Identifies the specific tool or software interface used to make the call. |

#EXISTING SETUPs

Existing Setups in account - Nothing

#PROOF
Tanma:~/projects/aws_eks-sec_project# eksctl get cluster --region us-east-1
No clusters found


#BUDGET STATUS
Credit Status - 147usd 


##Budget Amount - $10.00
DATADOG - Cloud SIEM included (DataDog expires in 6days o 2026-10-05 @23:59)
##AWS BALANCE - $147



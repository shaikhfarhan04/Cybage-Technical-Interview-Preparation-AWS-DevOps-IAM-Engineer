# **Cybage Technical Interview Preparation – AWS DevOps / IAM Engineer**

Based on the JD and the project scenario you described, I would prepare for this interview as an **AWS Multi-Account Security & DevOps Modernization Project**.

The most important thing is to demonstrate that you can design, automate, secure, and operate an AWS environment consisting of 40+ AWS accounts.

This is not just an IAM administration role. It combines:

1. AWS Organizations and multi-account architecture.
2. Advanced IAM policy evaluation and access management.
3. Centralized security governance.
4. Terraform-based infrastructure automation.
5. Organization-wide monitoring and auditing.
6. Security remediation and compliance automation.
7. Python, Terraform, and AWS CLI scripting.

I'll explain this as if you are already working on the project, so you can use the same explanations in your interview.

# **1. Understand the actual project scenario**

Imagine Cybage has an existing AWS infrastructure with 40+ AWS accounts.

Currently, different teams manage their infrastructure manually through the AWS Management Console.

This creates several problems:

- IAM policies are different across accounts.
- Some accounts have excessive IAM permissions.
- Root account access is not properly governed.
- CloudTrail logging is not centralized.
- AWS Config rules are inconsistent.
- Security Hub and GuardDuty are not enabled everywhere.
- No standard Terraform automation exists.
- There is no centralized governance using SCPs.
- Manual account onboarding and offboarding take time.
- Security compliance is difficult to track.

**Your project objective:**

Modernize the existing infrastructure by implementing centralized governance, security automation, IAM standardization, and Infrastructure as Code across all 40+ AWS accounts.

## **How I would explain my role**

**Role: AWS DevOps – IAM & Security Automation Engineer**

Responsible for modernizing and securing a multi-account AWS environment consisting of 40+ AWS accounts using AWS Organizations, IAM, Terraform, Python, and AWS Security services.

**Key responsibilities:**

- Designed and implemented centralized IAM governance across multiple AWS accounts.
- Worked on AWS Organizations, Organizational Units (OUs), Service Control Policies (SCPs), and cross-account IAM role configurations.
- Automated IAM users, roles, policies, permission boundaries, and trust relationships using Terraform.
- Implemented centralized CloudTrail logging and AWS Config compliance monitoring.
- Worked on organization-wide deployment and governance of GuardDuty and Security Hub.
- Implemented least-privilege access controls and reviewed IAM permissions using IAM Access Analyzer.
- Designed cross-account access using IAM roles and AWS STS AssumeRole.
- Worked on SAML-based federation and centralized workforce access management.
- Automated AWS security configuration and compliance checks using Python and Boto3.
- Implemented centralized encryption and key management using AWS KMS.
- Worked on AWS WAF managed rule groups to protect internet-facing applications.
- Developed reusable Terraform modules for consistent security configurations across multiple AWS accounts.
- Troubleshot IAM access-denied issues, SCP restrictions, role assumption failures, and AWS security configuration issues.

Use these responsibilities as a preparation template. In the interview, distinguish clearly between what you have actually implemented and what you understand architecturally.

# **2. Proposed architecture for 40+ AWS accounts**

This is the architecture I would propose during the interview.

[Improving Overall Security Posture with Wiz Secured AWS landing zone | AWS Partner Network (APN) Blog](https://images.openai.com/static-rsc-4/Bv1bH7Sn3UfF8iegzcYvPgyjuQciNQbW3RP5XC3N-tvk5gJmd5YhcN6jHwNY8bE5nP7fZzEXArjVKtdxx-UKfDWx3juhpb71RdQj1kzyQo5pB1kPaNp3h9F9HuwMmeWSNQX6mND3KFLZ_Ybc_kjwD8BmKiSiU56i9Q6PUpSQc5A?purpose=inline)

[aws.amazon.com](https://aws.amazon.com/blogs/apn/improving-overall-security-posture-with-wiz-secured-aws-landing-zone/?utm_source=chatgpt.com)

## **Step 1: Organize all 40+ accounts**

First, I would assess the existing accounts and organize them using AWS Organizations.

Example OU structure:

**AWS Organizations – Management Account**

Organization-level governance and billing

**Security OU**

Security Tooling

Log Archive

Audit

**Infrastructure OU**

Network

Shared Services

DevOps

**Non-Production OU**

Development

Testing

UAT

**Production OU**

Production App 1

Production App 2

Production App 3

Illustrative OU structure. Actual placement depends on account ownership, workloads, and governance requirements.

### **What is the purpose of each account?**

| Account           | Responsibility                                             |
| ----------------- | ---------------------------------------------------------- |
| Management        | AWS Organizations administration                           |
| Security Tooling  | GuardDuty, Security Hub, delegated security administration |
| Log Archive       | Centralized CloudTrail and security logs                   |
| Audit             | Compliance and read-only security auditing                 |
| Network           | Shared networking and connectivity                         |
| Workload accounts | Applications and business workloads                        |

**Important interview point:** Do not deploy application workloads in the AWS Organizations management account. Keep it restricted to organization-level administration.

### **What is AWS Control Tower?**

Service: AWS Control Tower

AWS Control Tower helps establish and govern a multi-account landing zone.

It provides:

- Account Factory for account provisioning.
- Landing Zone setup.
- Preventive controls through SCPs.
- Detective controls through AWS Config.
- Centralized logging.
- Integration with AWS Organizations.

For 40+ accounts, I would evaluate Control Tower as the foundation, rather than building every governance mechanism independently.

# **3. IAM policy evaluation – Most important interview topic**

This is likely to be one of the most important technical questions.

You should understand how AWS evaluates multiple policies together.

## **Different types of IAM policies**

| Policy               | Purpose                                                        |
| -------------------- | -------------------------------------------------------------- |
| Identity-based       | Defines permissions attached to users, groups, roles           |
| Resource-based       | Defines who can access a resource                              |
| Permissions boundary | Maximum permissions an identity-based policy can grant         |
| SCP                  | Maximum permissions available to principals in member accounts |
| Session policy       | Further restricts permissions for a particular role session    |

### **Example scenario**

Suppose:

- IAM role allows `s3:*`.
- Permission boundary allows only `s3:GetObject`.
- SCP allows `s3:GetObject` and `s3:ListBucket`.
- Session policy allows only `s3:GetObject`.

What happens?



Allowed

`s3:GetObject` (assuming the resource policy and other conditions also allow it).





Denied

`s3:PutObject`.



Why?

Because the effective permissions are constrained by the applicable policy types. A permission granted by one policy cannot override an applicable explicit deny or an applicable maximum-permissions restriction.

### **Remember this rule**

**Explicit Deny always wins.**

A simplified way to explain it:

```
Effective access =
    Applicable Allow permissions
    AND applicable permission limits
    AND applicable organization restrictions
    AND applicable session restrictions
    AND resource authorization
    MINUS explicit denies
```

This is a conceptual model, not a complete mathematical representation of every AWS authorization scenario. Resource policies, role sessions, service-specific behavior, and cross-account authorization have additional nuances.

## **3.1 SCP – Service Control Policy**

Service: AWS Organizations

SCP is one of the most important controls in a 40+ account environment.

Suppose you have 40 accounts and want to ensure nobody can disable CloudTrail.

Instead of creating an IAM policy in every account, create an SCP and attach it to the appropriate OU.

Example SCP:

**Objective:** Prevent member accounts from disabling CloudTrail or deleting trails.

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "DenyCloudTrailModification",
      "Effect": "Deny",
      "Action": [
        "cloudtrail:StopLogging",
        "cloudtrail:DeleteTrail",
        "cloudtrail:UpdateTrail"
      ],
      "Resource": "*"
    }
  ]
}

```

How it works:

1. Create an SCP in the Organizations management account.
2. Attach it to the Production OU.
3. All member accounts under that OU inherit the restriction.
4. Even an IAM administrator in a member account cannot perform the explicitly denied actions.

**Important interview questions:**

- SCP does not grant permissions.
- SCP defines the maximum available permissions.
- SCP affects member accounts, including their root users.
- SCP does not restrict principals in the management account.
- Service-linked roles are exempt from SCP restrictions.

Never apply a restrictive SCP to all 40 accounts without testing it first.

## **3.2 Cross-account IAM role design**

This is another very important practical scenario.

Suppose you have:

- Account A: Central DevOps account.
- Account B: Development account.
- Account C: Production account.
- Account D: Security account.

Your DevOps team needs to access EC2 and S3 resources in all these accounts without maintaining IAM users separately in every account.

**Solution: Cross-account IAM roles + AWS STS AssumeRole.**

[Deep Dive - Cloud IAM at Scale | Robert Leggett | Robert Leggett](https://images.openai.com/static-rsc-4/IF2U4TGoJBuH0auDCEltr1XkymP8O9FvoVT-lkxNMuqO2qcf8HdmlF2nGdM84PV30q21fSQzdJ8btttNk-hp8j102WgqZthzeVIrnWr3V-gSTFf6sXXNG2K6XlASdBDC48lKIOkJ3AK9cQJfWK1SzHrDTfTx7jJ04KIyyHC4uZE?purpose=inline)

[robertleggett.com.au](https://robertleggett.com.au/blog/deep-dive-cloud-iam-at-scale/?utm_source=chatgpt.com)

### **Example: Trust policy in the target account**

Suppose the DevOps role is in account `111111111111` and the target production account is `222222222222`.

Create this role in the production account:

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Sid": "AllowDevOpsAccount",
      "Effect": "Allow",
      "Principal": {
        "AWS": "arn:aws:iam::111111111111:role/CentralDevOpsRole"
      },
      "Action": "sts:AssumeRole"
    }
  ]
}

```

### **Permissions policy attached to the target role**

```

{
  "Version": "2012-10-17",
  "Statement": [
    {
      "Effect": "Allow",
      "Action": [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes",
        "s3:ListAllMyBuckets"
      ],
      "Resource": "*"
    }
  ]
}

```

### **How does it work?**

1. DevOps engineer authenticates into the Central DevOps account.
2. CentralDevOpsRole has permission to call `sts:AssumeRole` on the target role.
3. The target role's trust policy trusts CentralDevOpsRole.
4. AWS STS generates temporary security credentials.
5. DevOps engineer accesses permitted resources in the production account.

Remember: Both sides matter.

- Source account: Identity policy must allow `sts:AssumeRole`.
- Target account: Trust policy must trust the source principal.
- The resulting role session must have the required permissions.
- Applicable SCPs and other authorization controls must also permit the request.

For production, I would additionally implement MFA or federated identity controls, session duration limits, external ID where appropriate for third-party access, and CloudTrail auditing.

Reference:&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.aws.amazon.com\&sz=32)

AWS Identity and Access Management+1



# **4. SAML Federation – How enterprise users access AWS**

The interviewer may ask:

**How do you provide AWS access to 200 developers without creating 200 IAM users?**

My answer:

I would implement centralized federation using an external Identity Provider (IdP), such as Microsoft Entra ID (Azure AD), Okta, or another enterprise SAML provider.

[Workforce identity management - AWS Prescriptive Guidance](https://images.openai.com/static-rsc-4/47KcqZ-IfQiL--iGNCmk2VN2g1QE2Dvw3eOOiRRo51PYQmQokhztkidy7NarWlZiVCat2kNT6LuWmk5btmgCQNcMo__NpdcGiyfMlAX8tEboppDYlD5EJtsPV71LdN0gGxEjXYEiKtdIqBZJJGT-NTd7iEtqH6dNHpkGGgHogu8?purpose=inline)

[docs.aws.amazon.com](https://docs.aws.amazon.com/prescriptive-guidance/latest/security-reference-architecture-identity-management/workforce-identity-management.html?utm_source=chatgpt.com)

### **Authentication flow**

1. Employee logs in to the corporate identity provider.
2. IdP authenticates the employee.
3. IdP sends a SAML assertion to AWS.
4. AWS validates the assertion.
5. The user receives federated access through the configured AWS role or IAM Identity Center permission set.
6. The user accesses assigned AWS accounts.

### **What tools would I use?**

- AWS IAM Identity Center.
- Microsoft Entra ID / Okta.
- SAML 2.0.
- Permission Sets.
- SCIM provisioning for users and groups.
- AWS Organizations.

For a new multi-account workforce-access implementation, I would generally prefer IAM Identity Center with an external identity provider rather than creating individual IAM users.

**Interview question: What is the difference between SAML and AssumeRole?**

| SAML Federation                 | AssumeRole                                          |
| ------------------------------- | --------------------------------------------------- |
| Enterprise identity federation  | Temporary access to an IAM role                     |
| Uses SAML assertion             | Uses AWS STS API                                    |
| Commonly used for workforce SSO | Commonly used for cross-account and workload access |
| Authenticates through an IdP    | Requires a trusted principal and authorization      |

# **5. AWS security services – Understand their practical usage**

You should be able to explain what each service does and how it fits into the 40+ account architecture.

[GuardDuty SVG and PNG Icons | AWS Icons](https://images.openai.com/static-rsc-4/HJlZ5dhZl77V8g1knMpq9GiSscojP1qSg3sVbyqPsU7yZs2rQvSYGhFz-HH_UXLyJ_LRUnEkWh5_eIpO9m2JuxvbqFw_y6n42TH4ApAgHOA7TeKJ2-1hHO4pAUJFcZx_fKXfRZtCj-4JG_jP4PZPvuKnsbBf5FHyoEjJkaaWFKQ?purpose=inline)

**1. Amazon GuardDuty**

Threat detection service.

- Detects suspicious API activity.
- Identifies potentially compromised EC2 instances.
- Detects malicious IP communication.
- Monitors CloudTrail, VPC Flow Logs and DNS-related activity.

**Project usage:** Enable organization-wide GuardDuty and configure a delegated administrator.

[AWS Security Hub | AWS Security Blog](https://images.openai.com/static-rsc-4/d7vT_vFnP5s2e_5n-A6mqKDtVhq5GXsc0AZP-3m4jQwce05iJsE6jRDUq5SaKbrydLcuj6-H4zqaqslwvFF85mAz-7QZ84Ef_kCHbbbT1mOoMBDXNzpv7nGqTvIXJ_RuKeEatODfkBz_epHpGnJQmyOEEIi0BVIXafygN7WET9U?purpose=inline)

**2. AWS Security Hub**

Centralized security posture management.

- Consolidates security findings.
- Checks security standards such as AWS Foundational Security Best Practices.
- Provides visibility across accounts.
- Integrates with GuardDuty and other security services.

**Project usage:** Central security account receives findings from all member accounts.

[AWS Config Logo PNG Vector (SVG) Free Download](https://images.openai.com/static-rsc-4/9IVonJsqkMxQjLnrxALAnAA90DAPeudhVojYSvBKk06aUC-rcQny-sWby06XVoTAN2XPsT6iJ-GGEpiSnngnPDqrcsYdmg4dIA6G8XK0aTrArza_rl4OXHTTEg9mpqdRFH21p-jzRZY6c9RDUobaAp4U6b5M1SJas5YvLu46SGg?purpose=inline)

**3. AWS Config**

Configuration compliance and resource tracking.

- Tracks AWS resource configuration.
- Evaluates compliance against Config rules.
- Detects non-compliant resources.
- Maintains configuration history.

**Project usage:** Organization-wide Config deployment and centralized aggregation.

[How to Use AWS CloudTrail: Creating Trails | INE Internetwork Expert](https://images.openai.com/static-rsc-4/8esdxi1-8NHo7EKTuFsSsHGe7ZPQPI-ZX7wHUaJBlHcFxC2Et221BjgTZ9az7LeHPCdRP_jDwSdEil92tqRIu6Kqb1CoKkB5pwfctEK-ga4Y8C3ClWTARdetFxGs0080qrHv-ZIu9U5XJvt31-atE2mog53S6djZsXjngHUWwZ0?purpose=inline)

**4. AWS CloudTrail**

API activity auditing.

- Tracks who performed an API action.
- Records when and from where the action happened.
- Helps investigate IAM and security incidents.
- Supports organization trails.

**Project usage:** Centralized multi-account logging into a protected S3 Log Archive account.

[AWS Security Services Overview](https://images.openai.com/static-rsc-4/4WmevMyp9OQBlcSuVpk5YPWPBOiAefbAXcH0PMMj6w6TZoRQrsXSFvY8-zRoxrQqYVjT3wYnFm6SBIICw0FkwDvkGiRWzI7rUZGbX3dB9X2Q5xANKVkjwypldBiMA1_TI9QVKZ1z5SDdk5Cb403nKBaU1DbHJaY2U6h_38QvAuk?purpose=inline)

**5. AWS KMS**

Encryption key management.

- Encrypts S3, EBS, RDS and other supported resources.
- Controls key access through key policies and IAM policies.
- Supports key rotation and auditing.
- Supports cross-account key usage with appropriate policies.

**Project usage:** Standardized encryption requirements across accounts.

[WAF SVG and PNG Icons | AWS Icons](https://images.openai.com/static-rsc-4/kjbslaIjjqUFgruSb8hp56zmySomWeU3qzbzq1WkcIXjCqX51EXq7yJAgh4MR_2w22nhhWZ29h0fPuOHBxAhSiSPpyqwp5wiDA8XybzKfkzIUzOis2Uk3Ljys0O5xSbFG0iqPIhaMlTAaGak0t1Wh1DN4TfJz4V-GCedygwGEdk?purpose=inline)

**6. AWS WAF**

Web application firewall.

- Protects web applications against common attacks.
- Supports AWS Managed Rules.
- Can help protect against SQL injection and cross-site scripting.
- Can be integrated with CloudFront, Application Load Balancers and API Gateway.

**Project usage:** Standardized WAF managed rule groups for internet-facing applications.

### **Important: How do you centrally manage these services?**

| Service      | Central management approach                                          |
| ------------ | -------------------------------------------------------------------- |
| GuardDuty    | Delegated administrator                                              |
| Security Hub | Delegated administrator                                              |
| AWS Config   | Organization aggregator and delegated administration                 |
| CloudTrail   | Organization trail and delegated administration                      |
| KMS          | Account/region-specific keys and key policies                        |
| WAF          | Centralized standards/modules; Web ACL association at workload level |

For delegated administration, use a dedicated Security Tooling account rather than the management account wherever supported.

# **6. Terraform – How to automate 40+ AWS accounts**

High-priority practical topic

If they ask you how you will modernize manually deployed infrastructure into Terraform, explain the following approach.

### **My implementation approach**

**Phase 1 – Discovery**

- Identify all 40+ accounts.
- Collect account IDs and OU structure.
- Inventory IAM users, groups, roles and policies.
- Identify existing SCPs.
- Inventory CloudTrail, Config, GuardDuty and Security Hub configurations.
- Identify security gaps and compliance violations.

**Phase 2 – Establish Terraform foundation**

- Configure Terraform.
- Use S3 remote backend for state storage.
- Enable state locking using an S3 lockfile.
- Enable bucket versioning and encryption.
- Restrict access to the state bucket.
- Use GitHub/GitLab for source control.
- Implement CI/CD pipelines.

Note: For new Terraform S3 backends, use native S3 lockfiles (`use_lockfile = true`); DynamoDB-based locking is the older approach.

**Phase 3 – Create reusable modules**

I would create these modules:

```
terraform-aws-security/
│
├── modules/
│   ├── organizations/
│   ├── iam-roles/
│   ├── iam-policies/
│   ├── permission-boundaries/
│   ├── scp/
│   ├── cloudtrail/
│   ├── aws-config/
│   ├── guardduty/
│   ├── security-hub/
│   ├── kms/
│   └── waf/
│
├── environments/
│   ├── dev/
│   ├── staging/
│   └── production/
│
└── pipelines/
```

**Phase 4 – Rollout**

- Start with one non-production account.
- Test IAM and SCP policies.
- Validate cross-account role access.
- Deploy to a small pilot OU.
- Gradually roll out to all 40+ accounts.
- Monitor CloudTrail and Config compliance.
- Introduce drift detection.

### **Terraform coding sample 1: Create and attach an SCP**

This is an example you can discuss or write during the interview.

```

provider "aws" {
  region = "ap-south-1"
}

variable "target_ou_id" {
  type = string
}

resource "aws_organizations_policy" "deny_cloudtrail_changes" {
  name        = "DenyCloudTrailChanges"
  description = "Prevent disabling CloudTrail"
  type        = "SERVICE_CONTROL_POLICY"

  content = jsonencode({
    Version = "2012-10-17"

    Statement = [
      {
        Sid    = "DenyCloudTrailChanges"
        Effect = "Deny"

        Action = [
          "cloudtrail:StopLogging",
          "cloudtrail:DeleteTrail"
        ]

        Resource = "*"
      }
    ]
  })
}

resource "aws_organizations_policy_attachment" "security_guardrail" {
  policy_id = aws_organizations_policy.deny_cloudtrail_changes.id
  target_id = var.target_ou_id
}

```

What happens here?

1. Terraform creates an SCP.
2. The SCP prevents CloudTrail from being stopped or deleted.
3. Terraform attaches the policy to the specified OU.
4. All applicable member accounts under the OU inherit the restriction.

Before deploying, validate the policy and test it in a pilot OU. Also verify that your logging architecture has the necessary service permissions.

### **Terraform coding sample 2: Create a cross-account IAM role**

```

resource "aws_iam_role" "central_devops_role" {
  name = "CentralDevOpsRole"

  assume_role_policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Principal = {
        AWS = "arn:aws:iam::111111111111:root"
      }

      Action = "sts:AssumeRole"

      Condition = {
        ArnEquals = {
          "aws:PrincipalArn" =
            "arn:aws:iam::111111111111:role/CentralDevOpsRole"
        }
      }
    }]
  })
}

resource "aws_iam_role_policy" "readonly_access" {
  name = "ReadOnlyEC2"
  role = aws_iam_role.central_devops_role.id

  policy = jsonencode({
    Version = "2012-10-17"

    Statement = [{
      Effect = "Allow"

      Action = [
        "ec2:DescribeInstances",
        "ec2:DescribeVolumes"
      ]

      Resource = "*"
    }]
  })
}

```

This illustrates the trust relationship and role permissions. In a real multi-account setup, I would deploy the target role through a reusable module, and the source role would also need an identity policy allowing `sts:AssumeRole`.

# **7. Python + Boto3 – Coding questions for the interview**

For an AWS DevOps-IAM position, prepare Python fundamentals and Boto3 automation.

I would prioritize these coding exercises:

1. List all AWS accounts in an organization.
2. Assume a role across multiple AWS accounts.
3. Identify IAM users without MFA.
4. Identify IAM policies with AdministratorAccess.
5. Find IAM roles with overly permissive trust policies.
6. Check whether GuardDuty is enabled.
7. Identify CloudTrail status across accounts.
8. Identify non-compliant AWS resources.

## **Coding sample 1: Fetch all 40+ AWS accounts**

**Question:** How will you retrieve all AWS accounts from AWS Organizations?

```
import boto3org = boto3.client("organizations")paginator = org.get_paginator("list_accounts")for page in paginator.paginate():    for account in page["Accounts"]:        print(            "Account Name:", account["Name"],            "Account ID:", account["Id"],            "Status:", account["Status"]        )
```

Explanation:

- Boto3 is the AWS SDK for Python.
- Organizations client communicates with AWS Organizations.
- Paginator handles large account lists.
- We retrieve account names, IDs and status.

**Interview follow-up:** What if you have 40 accounts?

Answer: I would use pagination and process accounts in batches, rather than manually calling each account.

## **Coding sample 2: AssumeRole into multiple AWS accounts**

Very important

**Question:** How will you access multiple AWS accounts using a single central DevOps account?

```
import boto3org = boto3.client("organizations")sts = boto3.client("sts")def get_accounts():    accounts = []    paginator = org.get_paginator("list_accounts")    for page in paginator.paginate():        for account in page["Accounts"]:            if account["Status"] == "ACTIVE":                accounts.append(account["Id"])    return accountsdef assume_role(account_id):    role_arn = (        f"arn:aws:iam::{account_id}:role/CentralDevOpsRole"    )    try:        response = sts.assume_role(            RoleArn=role_arn,            RoleSessionName="DevOpsAuditSession"        )        credentials = response["Credentials"]        session = boto3.Session(            aws_access_key_id=credentials["AccessKeyId"],            aws_secret_access_key=credentials["SecretAccessKey"],            aws_session_token=credentials["SessionToken"]
```

**How to explain this in the interview:**

"I developed Python automation using Boto3 to fetch AWS accounts through Organizations, assume a predefined IAM role using STS, and execute read-only infrastructure audits across accounts. This helps eliminate manual account-by-account verification."

In a production implementation, I would add:

- Structured logging.
- Retry and backoff handling.
- API throttling protection.
- Role assumption failure reporting.
- Secure credential handling.
- Concurrent execution with controlled limits.

The code assumes you have an authorized source identity, Organizations read permissions, and the target role and its trust policy already configured.

## **Coding sample 3: Identify IAM users without MFA**

**Question:** How will you identify IAM users who have not enabled MFA?

```
import boto3iam = boto3.client("iam")paginator = iam.get_paginator("list_users")for page in paginator.paginate():    for user in page["Users"]:        username = user["UserName"]        mfa = iam.list_mfa_devices(            UserName=username        )        if len(mfa["MFADevices"]) == 0:            print(f"MFA NOT ENABLED: {username}")        else:            print(f"MFA ENABLED: {username}")
```

**Real-world application:** Run this script across accounts through AssumeRole and generate a centralized security report.

Important: This checks IAM users, not workforce identities managed through IAM Identity Center. Those require separate identity-management auditing.

## **Coding sample 4: Check whether GuardDuty is enabled**

```
import boto3def check_guardduty(session, account_id, region):    client = session.client(        "guardduty",        region_name=region    )    response = client.list_detectors()    if response["DetectorIds"]:        print(f"{account_id}: GuardDuty ENABLED")    else:        print(f"{account_id}: GuardDuty NOT ENABLED")regions = ["ap-south-1", "us-east-1"]for region in regions:    check_guardduty(        boto3.Session(),        "222222222222",        region    )
```

This sample checks the current credentials' account in each region. To audit all 40 accounts, first assume the target account's audit role, then pass that account's session to this function.

# **8. IAM troubleshooting scenarios – Prepare these answers**

These are the kinds of real-world situations I would expect you to be able to troubleshoot.

## **Scenario 1: Developer has AdministratorAccess but still gets AccessDenied.**

**Question:** How will you troubleshoot?

My answer:

I will not immediately change the AdministratorAccess policy. I will check the entire authorization chain.

1. Check the identity-based policy.
2. Check whether a permissions boundary is attached.
3. Check the SCPs inherited from the OU and root.
4. Check session policies.
5. Check resource-based policies.
6. Check trust policies if AssumeRole is involved.
7. Check KMS key policies if the denied operation involves encryption.
8. Review the exact denied API request in CloudTrail.

Tools:

- IAM Policy Simulator.
- IAM Access Analyzer.
- AWS CloudTrail.
- AWS CLI.
- AWS Organizations.

## **Scenario 2: Cross-account AssumeRole is failing.**

My troubleshooting steps:

```
aws sts get-caller-identity
```

First verify the current identity.

Then:

```
aws sts assume-role \
  --role-arn arn:aws:iam::222222222222:role/CentralDevOpsRole \
  --role-session-name DevOpsTest
```

Check:

- Is the source identity allowed to call `sts:AssumeRole`?
- Does the target trust policy trust the source?
- Is the correct account ID and role ARN used?
- Is there an explicit deny in an SCP?
- Is the target role's trust condition satisfied?
- Are external ID or MFA conditions required?
- Are temporary credentials expired?

## **Scenario 3: Someone accidentally removes AdministratorAccess.**

I would:

1. Check CloudTrail for the API call.
2. Identify the principal who changed the policy.
3. Check whether another administrative role exists.
4. Check SCP restrictions.
5. Restore the minimum required permissions through an approved process.
6. Investigate the root cause and introduce change controls.

Never assume you can bypass an SCP by using the root user of a member account.

## **Scenario 4: How will you identify unused IAM permissions?**

I would use IAM Access Analyzer and IAM last-accessed information.

- Identify unused roles and policies.
- Review services and actions accessed.
- Generate least-privilege policy recommendations.
- Test changes in non-production.
- Deploy through Terraform after approval.

## **Scenario 5: How do you prevent developers from creating IAM administrators?**

My answer:

"I would implement a combination of SCPs, IAM permission boundaries, and centrally managed role provisioning. SCPs establish organization-level restrictions, while permission boundaries limit the maximum permissions that developers can delegate to identities they create. I would also monitor changes through CloudTrail and Access Analyzer."

# **9. Tools required for this project**

Here is the toolset I would mention when the interviewer asks: "What tools will you use to modernize this infrastructure?"

| Category       | Tools                                | Purpose                               |
| -------------- | ------------------------------------ | ------------------------------------- |
| Cloud          | AWS                                  | Infrastructure platform               |
| Governance     | AWS Organizations                    | Manage 40+ accounts                   |
| Landing Zone   | AWS Control Tower                    | Standardized account governance       |
| Identity       | IAM Identity Center                  | Workforce SSO                         |
| Federation     | SAML 2.0, Entra ID, Okta             | Enterprise authentication             |
| IAM Security   | Access Analyzer, Policy Simulator    | Access reviews                        |
| IaC            | Terraform                            | Infrastructure automation             |
| Coding         | Python, Boto3                        | Multi-account automation              |
| CLI            | AWS CLI                              | Troubleshooting and operations        |
| CI/CD          | GitHub Actions / Jenkins / GitLab CI | Terraform deployment                  |
| State          | S3 backend                           | Terraform remote state                |
| Security       | GuardDuty, Security Hub              | Threat detection and security posture |
| Audit          | CloudTrail                           | API activity tracking                 |
| Compliance     | AWS Config                           | Resource compliance                   |
| Encryption     | KMS                                  | Encryption and key management         |
| Web Security   | AWS WAF                              | Web application protection            |
| Monitoring     | CloudWatch, SNS                      | Monitoring and alerting               |
| Policy as Code | Checkov, tfsec, OPA/Conftest         | Terraform policy validation           |

For an enterprise rollout, I would also use Terraform plan reviews, protected branches, approval gates, and automated security testing.

# **10. End-to-end implementation workflow**

This is the workflow you should be able to draw on a whiteboard.

[Secure AWS Multi-Account Reference Architecture (High-Level Design)](https://images.openai.com/static-rsc-4/oTVE3g_UGChKQGPtA_2w4o_TU5PWKAzXHuVff2M7h7NRi-WvQRo8wxzqoax1sj_6GrqPEYK_AOailOaQuU4cKgD5Nz0ZcyuRADhmW7-tLym5oTC9M5lMUgAMeYpl-e80u5yQXDmDSSQ1YWDLvfOlFIx70Id2bS2IJboMKfScL6w?purpose=inline)

[linkedin.com](https://www.linkedin.com/pulse/secure-aws-multi-account-reference-architecture-yarravarapu-qwdlc?utm_source=chatgpt.com)

```
                  GitHub / GitLab
                         |
                         v
                   Terraform Code
                         |
                         v
                  CI/CD Pipeline
                         |
             +-----------+-----------+
             |                       |
             v                       v
        Terraform Plan          Security Scan
             |                  Checkov / OPA
             +-----------+-----------+
                         |
                         v
                  Manual Approval
                         |
                         v
               AWS Management Account
                         |
                  AWS Organizations
                         |
              +----------+----------+
              |          |          |
              v          v          v
           Dev OU      Prod OU   Security OU
              |          |          |
              v          v          v
          IAM Roles   IAM Roles  Security Hub
          SCPs        SCPs       GuardDuty
          Config      Config     CloudTrail
              |          |          |
              +----------+----------+
                         |
                         v
                Centralized Monitoring
                         |
                         v
                 Security Dashboard
```

## **How I would divide the project into implementation phases**

| Phase | Work                                 | Deliverable                           |
| ----- | ------------------------------------ | ------------------------------------- |
| 1     | Account discovery and IAM assessment | Account inventory                     |
| 2     | Organizations and OU design          | Governance structure                  |
| 3     | IAM role and SCP standardization     | IAM baseline                          |
| 4     | Terraform modules and CI/CD          | Automated deployment                  |
| 5     | CloudTrail and Config                | Centralized auditing                  |
| 6     | GuardDuty and Security Hub           | Security monitoring                   |
| 7     | KMS and WAF                          | Encryption and application protection |
| 8     | Drift detection and remediation      | Continuous governance                 |

**A very important point:** If the 40+ accounts already contain manually created resources, I would not blindly run Terraform apply over them.

I would first import existing resources into Terraform state where supported, reconcile configuration, identify drift, and then migrate gradually.

Also, I would use Terraform for provisioning and AWS Config/CloudTrail for ongoing detection. They solve different problems.

# **11. Most expected interview questions**

Prepare these questions before tomorrow.

**IAM and AWS Organizations**

Highest priority

Select a question to reveal its key answer.

1\. Explain IAM policy evaluation logic.

2\. What is the difference between SCP and IAM policy?

3\. What is a permission boundary?

4\. What is the difference between trust policy and permission policy?

5\. How do you implement cross-account access?

6\. What is SAML federation?

7\. What is IAM Identity Center?

8\. What is delegated administration?

9\. What is the difference between GuardDuty and Security Hub?

10\. How does AWS Config work?

11\. How do you enable CloudTrail across 40 accounts?

12\. What is IAM Access Analyzer?

13\. How do you automate IAM using Terraform?

14\. How do you troubleshoot AccessDenied?

15\. How do you manage Terraform state?

# **12. Five advanced questions that can differentiate you**

These are especially useful because the JD specifically mentions expert-level IAM and security governance.

**Q1. Can an IAM Administrator bypass an SCP?**

No. An SCP restricts permissions available to member-account principals, including the member account root user. An IAM administrator cannot override an applicable SCP explicit deny. The Organizations management account is not restricted by SCPs in the same way.

**Q2. Does an SCP grant permission?**

No. SCPs define the maximum permissions available. You still need an applicable identity-based or resource-based authorization.

**Q3. Can a resource-based policy override a permission boundary?**

There are important nuances. In the same account, a resource policy granting access directly to a role session can behave differently from one granting access to the IAM role ARN. You should not give a blanket yes or no. Explain the principal type and policy context.

**Q4. How would you protect the management account?**

- Restrict access to a very small number of administrators.
- Use MFA and centralized federation where appropriate.
- Avoid application workloads in the management account.
- Monitor sensitive API operations through CloudTrail.
- Protect the root user credentials.
- Implement break-glass procedures and alerts.

**Q5. How would you deploy SCPs without affecting production?**

- Create a test OU.
- Test policies in a development account.
- Use IAM Policy Simulator and Access Analyzer where applicable.
- Review critical AWS service dependencies.
- Deploy with Terraform through an approval pipeline.
- Roll out progressively.
- Monitor CloudTrail and application operations.
- Maintain a tested rollback procedure.

References for these advanced concepts:&#x20;

[image](https://www.google.com/s2/favicons?domain=https://docs.aws.amazon.com\&sz=32)

AWS Identity and Access Management+3



# **13. Your 2-minute project explanation**

This is probably the most important part of your preparation. If the interviewer asks, "Explain how you would handle our 40+ AWS account modernization project", you can use this answer.

**Project: AWS Multi-Account Infrastructure Modernization and Security Governance**

In this project, we are focusing on modernizing an existing AWS infrastructure consisting of more than 40 AWS accounts.

The existing infrastructure has been provisioned and managed through the AWS Management Console, resulting in inconsistent IAM configurations, manual security operations, limited centralized visibility, and governance challenges.

My responsibility as an AWS DevOps-IAM Engineer would be to standardize IAM security, implement centralized governance, and automate infrastructure configurations across multiple AWS accounts.

First, I would assess the existing AWS Organizations structure, identify all accounts, review IAM users, roles, policies, existing SCPs, and security configurations.

Next, I would establish a standardized OU structure based on security, infrastructure, development, and production requirements.

For IAM governance, I would implement least-privilege access, centralized role management, permission boundaries, cross-account AssumeRole access, and SAML-based federation using IAM Identity Center.

For organization-level security, I would implement SCPs to prevent critical operations such as disabling CloudTrail or modifying mandatory security configurations.

From an automation perspective, I would use Terraform to develop reusable modules for IAM roles, policies, SCPs, CloudTrail, AWS Config, GuardDuty, Security Hub, and KMS.

I would maintain Terraform state in an encrypted and versioned S3 backend and integrate Terraform with GitHub Actions or Jenkins for automated plan, validation, approval, and deployment.

For centralized monitoring, I would configure organization-wide CloudTrail, AWS Config aggregation, GuardDuty, and Security Hub with dedicated delegated administrator accounts.

I would also develop Python automation using Boto3 to audit IAM permissions, identify accounts with missing security configurations, validate cross-account access, and generate compliance reports.

Finally, I would implement continuous monitoring, drift detection, and automated remediation through approved workflows.

The overall objective is to achieve consistent security governance, reduce manual operational effort, improve visibility, and establish a scalable AWS multi-account architecture.

# **14. What to study tonight – Priority-wise revision plan**

Since your interview is tomorrow, don't try to learn every AWS service in depth. Concentrate on the following.

## **Interview preparation checklist**

Mark each topic as you revise it.

**Your progress**

# **0%**

**IAM policy evaluation**

Identity, resource, boundary, SCP, session

**SCP practical examples**

Deny CloudTrail deletion, restrict regions, protect IAM

**Cross-account roles**

Trust policy, sts:AssumeRole, temporary credentials

**SAML and Identity Center**

IdP, SAML assertion, permission sets

**AWS Organizations**

OU design, delegated administration, account onboarding

**Terraform**

SCP, IAM role, modules, remote state, locking

**Python Boto3**

List accounts, AssumeRole, IAM MFA audit

**CloudTrail and Config**

Organization trail, aggregator, compliance

**GuardDuty and Security Hub**

Delegated administrator, findings

**Troubleshooting**

AccessDenied, SCP, trust relationship, KMS

## **Official AWS documentation to revise**

1. [IAM Policy Evaluation Logic](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic.html?utm_source=chatgpt.com)
2. [Cross-Account Policy Evaluation](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic-cross-account.html?utm_source=chatgpt.com)
3. [IAM Permissions Boundaries](https://docs.aws.amazon.com/IAM/latest/UserGuide/access_policies_boundaries.html?icmpid=docs_iam_console\&utm_source=chatgpt.com)
4. [CloudTrail Delegated Administration](https://docs.aws.amazon.com/awscloudtrail/latest/userguide/cloudtrail-delegated-administrator.html?utm_source=chatgpt.com)
5. [Organization CloudTrail](https://docs.aws.amazon.com/en_en/awscloudtrail/latest/userguide/creating-trail-organization.html?utm_source=chatgpt.com)
6. [AWS Authorization Evaluation Details](https://docs.aws.amazon.com/IAM/latest/UserGuide/reference_policies_evaluation-logic_policy-eval-denyallow.html?utm_source=chatgpt.com)

# **15. My recommendation for your interview**

I would focus on three things:

**1. IAM expertise:** Be very confident explaining the difference between IAM policy, SCP, permissions boundary, trust policy, and session policy. The interviewer may give you a real-world AccessDenied scenario and expect you to troubleshoot it.

**2. Terraform automation:** Be prepared to write an SCP, create an IAM role, attach a policy, and explain remote state and modules.

**3. Multi-account architecture:** Explain how you would onboard 40+ accounts, implement centralized security, use delegated administration, and roll out changes without disrupting production.

One important distinction: Don't claim that you personally implemented all these services if you have only designed or studied them. Explain your actual hands-on experience and then describe how you would implement the remaining components.

**Let's do one final preparation exercise: a Cybage-style mock technical interview.** I can act as the interviewer and ask you 20 scenario-based questions, one at a time, covering advanced IAM, SCP, Terraform, Boto3, AWS Organizations, and troubleshooting. I'll evaluate each answer and give you an improved interview-ready response.

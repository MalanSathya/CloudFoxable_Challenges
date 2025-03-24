# It's Another Secret

## Challenge Statement
I needed to assume the `Ertz` role, determine its permissions, and retrieve the flag stored in AWS Systems Manager Parameter Store.

## Solution

### Step 1: Assuming the `Ertz` Role
To begin, I assumed the `Ertz` role using the `cloudfoxable` profile:
```sh
aws --profile ertz sts get-caller-identity
```
**Output:**
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "its-another-secret-policy",
            "PolicyArn": "arn:aws:iam::872678986397:policy/its-another-secret-policy"
        }
    ]
}
```

### Step 2: Checking Attached IAM Policies
I checked if the `Ertz` role had any attached IAM policies:
```sh
aws iam list-attached-role-policies --role-name Ertz --profile cloudfoxable
```
**Output:**
```json
{
    "AttachedPolicies": [
        {
            "PolicyName": "its-another-secret-policy",
            "PolicyArn": "arn:aws:iam::872678986397:policy/its-another-secret-policy"
        }
    ]
}
```

### Step 3: Retrieving Policy Details
I retrieved the details of the `its-another-secret-policy` policy:
```sh
aws iam get-policy --policy-arn arn:aws:iam::872678986397:policy/its-another-secret-policy --profile cloudfoxable
```
**Output:**
```json
{
    "Policy": {
        "PolicyName": "its-another-secret-policy",
        "PolicyId": "ANPA4WJPMALZGGTPTMQGBH",
        "Arn": "arn:aws:iam::872678986397:policy/its-another-secret-policy",
        "Path": "/",
        "DefaultVersionId": "v1",
        "AttachmentCount": 1,
        "PermissionsBoundaryUsageCount": 0,
        "IsAttachable": true,
        "Description": "policy that only allows access to the its-another-secret flag",
        "CreateDate": "2025-02-12T15:37:17+00:00",
        "UpdateDate": "2025-02-12T15:37:17+00:00",
        "Tags": []
    }
}
```

### Step 4: Checking the Policy Version
To inspect the policy permissions, I retrieved the policy document:
```sh
aws iam get-policy-version --policy-arn arn:aws:iam::872678986397:policy/its-another-secret-policy --version-id v1 --profile cloudfoxable
```
**Output:**
```json
{
    "PolicyVersion": {
        "Document": {
            "Statement": [
                {
                    "Action": [
                        "ssm:GetParameter"
                    ],
                    "Effect": "Allow",
                    "Resource": [
                        "arn:aws:ssm:us-west-2:872678986397:parameter/cloudfoxable/flag/its-another-secret"
                    ]
                }
            ],
            "Version": "2012-10-17"
        },
        "VersionId": "v1",
        "IsDefaultVersion": true,
        "CreateDate": "2025-02-12T15:37:17+00:00"
    }
}
```

### Step 5: Retrieving the Flag
Since the policy allowed `ssm:GetParameter` on the specific flag, I retrieved it using:
```sh
aws ssm get-parameter --name /cloudfoxable/flag/its-another-secret --with-decryption --profile ertz
```
**Dummy Output:**
```json
{
    "Parameter": {
        "Name": "/cloudfoxable/flag/its-another-secret",
        "Type": "SecureString",
        "Value": "FLAG{ItsAnotherSecret::ThereWillBeALotOfAssumingRolesInThisCTF}",
        "Version": 1,
        "LastModifiedDate": "2025-02-12T10:37:13.901000-05:00",
        "ARN": "arn:aws:ssm:us-west-2:872678986397:parameter/cloudfoxable/flag/its-another-secret",
        "DataType": "text"
    }
}
```

## Reflection

* **What was my approach?**
  - I assumed the `Ertz` role and investigated its IAM policies to determine access permissions.

* **What was the biggest challenge?**
  - Identifying the correct policy permissions and ensuring the role had access to retrieve the flag.

* **How did I overcome the challenges?**
  - Used `aws iam` commands to inspect policies and understand permissions before executing the flag retrieval.

* **What led to the breakthrough?**
  - Discovering the `its-another-secret-policy` allowed `ssm:GetParameter` access to the flag.

* **Lessons Learned:**
  - Always check attached policies and their versions for permission details.
  - AWS Systems Manager Parameter Store can store secrets with restricted access policies.
  - Role assumptions and AWS profiles simplify access management during security assessments.


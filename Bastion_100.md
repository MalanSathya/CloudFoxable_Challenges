# Bastion 100 Challenge

## Challenge Statement
Deploy a bastion host using Terraform in the CloudFoxable environment, access it using SSM, determine available IAM permissions, and retrieve the flag.

## Solution

### Step 1: Enabling the Bastion Challenge
1. I navigated to the CloudFoxable Terraform configuration:
   ```sh
   cd cloudfoxable/aws
   ```
2. I edited the Terraform configuration file:
   ```sh
   nano terraform.tfvars
   ```
3. I set the bastion flag to `true`:
   ```
   bastion_enabled = true
   ```
4. Then, I applied the changes:
   ```sh
   terraform apply
   ```
   
### Step 2: Workaround for "Entity, Resource, Policy Already Exists" Error
After running `terraform apply`, I encountered an error stating that certain entities, resources, or policies already existed. To fix this:
- I first destroyed the conflicting resources:
  ```sh
  terraform destroy -target aws_iam_role.sawnson -target aws_instance.bastion_host
  ```
- Repeadted this step for all the existing resources, then I re-ran:
  ```sh
  terraform apply
  ```
  This allowed Terraform to apply the configuration successfully.
  
### Step 3: Finding the Bastion Instance ID
Once deployed, I used CloudFox to find the instance ID:
```sh
cloudfox aws -p cloudfoxable instances -v2
```
The output contained a "loot" file, which included the required instance ID.

### Step 4: Installing and Using AWS SSM to Connect to the Bastion
1. I installed the AWS Session Manager Plugin:
   ```sh
   https://s3.amazonaws.com/session-manager-downloads/plugin/latest/windows/SessionManagerPluginSetup.exe
   ```
2. Then, I connected to the instance:
   ```sh
   aws ssm start-session --target i-037c9df8ab2222c97
   ```

### Step 5: Using CloudFox to Determine IAM Permissions
I needed to list the permissions for the assumed role, so I ran:
```sh
cloudfox aws -p cloudfoxable permissions --principal [NameOfInstanceRole]
```

### Step 6: Workaround for "Cannot Assume Role (reyna) to List S3 Buckets"
When I tried assuming the `reyna` role to list S3 buckets, I encountered an issue. Here’s how I fixed it:
- I checked if `reyna` had an assume-role policy attached:
  ```sh
  aws iam get-role --role-name reyna
  ```
- Since it was missing, I manually attached an assume-role policy:
  ```sh
  aws iam attach-role-policy --role-name reyna --policy-arn arn:aws:iam::aws:policy/AmazonS3ReadOnlyAccess
  ```
- Then, I assumed the role correctly:
  ```sh
  aws sts assume-role --role-arn arn:aws:iam::ACCOUNT_ID:role/reyna --role-session-name reyna-session
  ```
  Using the temporary credentials from the output, I was finally able to list the S3 buckets:
  ```sh
  aws s3 ls
  ```

### Step 7: Finding the Flag
1. I listed the available S3 buckets:
   ```sh
   aws s3 ls
   ```
2. I identified the relevant bucket (e.g., `cloudfoxable-bastion-zcv2a`).
3. I listed the contents of the bucket:
   ```sh
   aws s3 ls s3://cloudfoxable-bastion-zcv2a
   ```
4. I found `flag.txt` in the bucket and displayed its contents:
   ```sh
   aws s3 cp s3://cloudfoxable-bastion-zcv2a/flag.txt -
   ```
5. I retrieved the flag: `{FLAG:bastion::ifYouHaveAccessToAnEC2YouHaveAccessToItsIamPermissions}`

### Step 8: Cleanup
I exited the bastion instance shell and cleaned up the Terraform resources:
```sh
terraform destroy
```

## Reflection

* **What was my approach?**
  - I followed Terraform deployment steps, fixed errors, and used CloudFox for enumeration.

* **What was the biggest challenge?**
  - The "Entity, resource, policy already exists" error when applying Terraform.
  - Difficulty assuming the `reyna` role to list S3 buckets.

* **How did I overcome the challenges?**
  - I used `terraform destroy` on specific conflicting resources before re-applying Terraform.
  - I manually verified and updated IAM policies to enable role assumption.

* **What led to the breakthrough?**
  - Debugging the IAM policies and reconfiguring the assume-role settings for `reyna`.
  - Using CloudFox to enumerate permissions before trying to assume the role.

* **Lessons Learned:**
  - Terraform errors can often be resolved by destroying and re-applying resources.
  - Always check IAM policies before assuming roles.
  - CloudFox provides useful insights into permissions and role access in AWS environments.


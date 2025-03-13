# Personal AWS Environment set up Guide

## Overview
Setting up an AWS environment correctly is crucial for security and operational efficiency. This guide walks through the essential steps to create a secure AWS environment along with lessons learned from challenges encountered.

## Step-by-Step Guide to Set Up AWS Environment

### 1. Create an AWS Account
- Sign up for an AWS account at [AWS Console](https://aws.amazon.com/).
- Log in using the root account credentials.
- Navigate to Billing Preferences and enable multi-factor authentication (MFA) for the root user.

### 2. Secure the Root User
- Go to IAM (Identity & Access Management) in the AWS Console.
- Enable MFA for the root user:
  - Select Security Credentials under your root user profile.
  - Click Activate MFA and choose a virtual MFA device (Google Authenticator, Authy, etc.).
  - Scan the QR code and enter two consecutive authentication codes to enable MFA.
- Remove root access keys if any exist to prevent unauthorized access.

### 3. Create an IAM Admin User
- Navigate to IAM > Users and click Add user.
- Enter a username (`eg: malan_cs_lab_user`).
- Select AWS Management Console access and set an initial password.
- Enable Require password reset upon first login for security.
- Click Next: Permissions and choose Attach policies directly.
- Attach the AdministratorAccess policy.
- Click Next: Tags, add any tags (optional), and click Create user.

### 4. Configure MFA for the IAM User
- Go to IAM > Users > malan_cs_lab_user.
- Select Security credentials and click Manage MFA.
- Repeat the same steps taken for setting root user MFA.
 
### 5. Create Access Keys for CLI Access
- Go to IAM > Users > malan_cs_lab_user > Security credentials.
- Under Access keys, click Create access key.
- Choose Command Line Interface (CLI) usage and confirm.
- Copy the Access Key ID and Secret Access Key.

### 6. Install and Configure AWS CLI
- Download and install the AWS CLI from [AWS CLI Installation](https://aws.amazon.com/cli/).
- Configure the AWS CLI by running:
  ```sh
  aws configure
  ```
  Enter the IAM user’s access key, secret key, region, and default output format.
- Verify the setup using:
  ```sh
  aws sts get-caller-identity
  ```
  This should return the IAM user’s details.

## Reflection
* **Why create an IAM admin user instead of using the root user?**
  - The root user has unrestricted access and should only be used for critical account-wide configurations.
  - An IAM admin user allows for better access control, logging, and auditing.

* **Security Measures Implemented:**
  - Enabled MFA for all users to reduce unauthorized access risks.
  - Created a separate IAM user (`malan_cs_lab_user`) instead of using the root user.
  - Restricted console and programmatic access based on user roles.

* **Challenges Faced:**
  - Configuring MFA correctly for IAM users.
  - Confusion in granting appropriate permissions to `malan_cs_lab_user` while keeping the root user secure.
  - Ensuring `malan_cs_lab_user` had both administrative and console access without excessive permissions.

* **How These Challenges Were Overcome:**
  - Carefully reviewing IAM policies and testing access before finalizing permissions.
  - Using AWS documentation to understand best practices for IAM user management.

---

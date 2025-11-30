IAM Identity and Acess Management 
- IAM users
- AWS CLI
- IAM groups
- IAM Roles
- IAM identity policy
- UAM resource based policy
- Session policy
- Permision boundary

Policies and Permissions

✅ 1. IAM Access Analyzer – Policy Generation (Recommended)

Purpose: Automatically generates minimal-required permissions based on actions you actually perform.
- You turn on Access Analyzer > Policy generation
- Perform the actions in the AWS Console or CLI
- AWS analyzes CloudTrail activity and produces a least-privilege policy
➡️ This is currently the best tool for discovering the exact permissions needed.

✅ 2. IAM Policy Simulator

Purpose: Test an existing policy to see if it allows or denies operations.
- Lets you simulate which permissions are required
- Good for validating or debugging policies
https://policysim.aws.amazon.com

✅ 3. AWS Console "IAM Policy Creator" (Visual Editor)

Purpose: Helps you build policies by selecting service → actions → resources.
- The visual editor shows all available actions for each service
- Helps you avoid missing required actions

✅ 4. AWS Documentation: Service Authorization Reference

Every AWS service has a full list of:
- Actions
- Required permissions
- Supported resource types
- Permission boundaries

Search: “AWS <service> permissions reference”
Example: “AWS S3 Actions, Resources, and Condition Keys”

The principle of least privilege (PoLP) refers to an information security concept in which a user is given the minimum levels of access – or permissions – needed to perform his/her job functions.

✅ Explicit DENY always overrides ALLOW

So with your setup:

- EC2 instance role → policy with Deny: eks:PodRestart (or similar)

- Your IAM user → policy with Allow: eks:PodRestart

👉 Final outcome: the action is DENIED.  
Even if one policy says Allow, ANY explicit Deny anywhere in the evaluation chain blocks the action.
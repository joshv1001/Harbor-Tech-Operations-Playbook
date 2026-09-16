# Week 2: IAM and AWS CLI Investigation

## HarborTech Ticket Summary
Riverside Goods reported that Marcus Webb can sign in to AWS but cannot complete one of the inventory tasks required for his job. The ticket is TKT-2026-0002. Marcus needs to list the riverside-inventory S3 bucket, read inventory report objects, and upload approved inventory files. His request to list the bucket returns AccessDenied.
The purpose of this investigation is to identify whether the problem involves authentication, authorization, permissions, or an expected security restriction. The goal is to recommend the correct access direction without giving Marcus unnecessary permissions.

## Client Impact
Marcus is unable to complete the inventory work assigned to him because the current AWS access configuration does not allow the requested bucket-listing action. This can delay inventory work and prevent him from reading or uploading the files he needs.
The problem should not be solved by giving him full access to AWS services that are unrelated to his job. Incorrect permissions could allow unnecessary access to other S3 buckets or other resources. The correct approach is to restore the required work while keeping the access limited to the resources Marcus needs.

## AWS Services Involved
The main AWS services and concepts involved in this investigation are:
AWS Identity and Access Management (IAM)
IAM users, groups, roles, and policies
Amazon S3
AWS CloudShell
AWS Command Line Interface (AWS CLI)
AWS Regions
IAM controls which identities can access AWS resources and what actions they can perform. Amazon S3 stores the inventory data and files Marcus needs to work with. CloudShell and the AWS CLI provide a way to inspect the AWS environment and collect evidence.

## Virtualization Connection
Cloud computing provides software-defined resources such as virtual machines, storage, and networking. These resources can be created and managed through cloud services instead of only using physical hardware.
Identity and access controls are important because they determine who can manage or access those resources. For example, an employee may be allowed to read files from an S3 bucket but not change IAM permissions or manage virtual machines. This separation helps protect cloud environments and supports the principle of least privilege.

## Evidence Reviewed
The evidence reviewed for TKT-2026-0002 includes:
Marcus successfully signing in to the Riverside Goods AWS console.
His job requirement to list the riverside-inventory bucket, read inventory report objects, and upload approved inventory files.
The IAM onboarding record showing that the user exists.
The absence of a listed job-function group membership.
The absence of a directly attached permission policy.
The AccessDenied result from the bucket-listing request.
The client's proposal to attach AmazonS3FullAccess directly to Marcus.
For the Learner Lab investigation, I used the AWS CLI commands:
aws sts get-caller-identity
aws iam get-role --role-name LabRole
aws iam list-attached-role-policies --role-name LabRole
aws iam list-role-policies --role-name LabRole
The actual caller identity, role information, policy results, and any AccessDenied messages should be recorded from the CloudShell session. I should not add made-up command output or sensitive credentials to this Playbook.

## Operational Analysis
The evidence shows that Marcus can authenticate successfully but cannot perform the requested inventory action. Authentication verifies the user's identity and allows the user to sign in. Authorization determines what that identity is allowed to do after signing in.
Marcus's successful login proves that the authentication process worked. The AccessDenied result shows that AWS rejected his request to list the inventory bucket under his current permissions. This supports an authorization or permissions gap rather than a failed login.
The onboarding record shows that the IAM user exists, but no job-function group membership or directly attached policy is listed. This is consistent with the user not having the required S3 access. The evidence does not prove that the account itself is broken. It shows that the requested action is not currently authorized.
The AccessDenied result can also be an expected security restriction. AWS should deny actions that are not allowed by the identity's effective permissions. The correct response is to investigate the missing permissions and compare them with the business requirement.
The client proposal to attach AmazonS3FullAccess directly to Marcus is broader than the documented job requirement. Marcus does not need full S3 access to perform his inventory duties, and the broad policy could allow access to unrelated buckets or S3 actions.

## Recommendation
The recommended direction is to use an approved job-function group or IAM role with a narrowly scoped S3 policy. The policy should support the inventory work Marcus actually needs to perform.
At a conceptual level, the required S3 actions are:
s3:ListBucket for the riverside-inventory bucket.
s3:GetObject for the inventory report objects Marcus needs to read.
s3:PutObject for uploading approved inventory files.
The final policy should use the correct resource scope for the bucket and objects. Any required upload restrictions or conditions should also be reviewed. The AmazonS3FullAccess policy should not be attached directly to Marcus just to make the error disappear.

## Escalation Notes
The supported finding is that Marcus can sign in but is missing authorization for the required S3 task. The onboarding record and AccessDenied result support an access-control problem.
The account owner proposed attaching AmazonS3FullAccess, but this policy is broader than the stated business requirement. The appropriate access direction is to review and implement a least-privilege S3 policy through an approved IAM group or role.
The remaining implementation decision must be made by an authorized HarborTech IAM or cloud operations reviewer. The reviewer needs to determine the correct group or role, policy resource scope, upload restrictions, and approval requirements. The policy should be tested against the required tasks before the production change is made.
As the junior cloud operations intern, I am authorized to inspect, document, test, recommend, and escalate. I am not authorized to grant broad production permissions. The actual IAM access change should be reviewed and implemented by an authorized HarborTech team member.

## Learner Lab CLI Evidence
The Learner Lab investigation uses the pre-created LabRole. I did not create Marcus, create IAM users or groups, or change permissions to reproduce the fictional client account.

## Caller Identity
Command:
aws sts get-caller-identity
This establishes the AWS account and identity associated with the current CloudShell session. The Account and Arn values should be recorded from the actual output. They help identify who is making the request but do not prove that the identity has permission to perform every AWS action.

## LabRole Trust Evidence
Command:
aws iam get-role --role-name LabRole
If permitted, the output provides the role ARN and trust policy information. The trust relationship identifies which principals are allowed to assume the role. If the command returns AccessDenied, that restriction should be recorded rather than bypassed.

## Permission Sources
Commands:
aws iam list-attached-role-policies --role-name LabRole
aws iam list-role-policies --role-name LabRole
The first command checks managed policies attached to LabRole. The second checks inline policies. An empty managed-policy result does not automatically mean that LabRole has no permissions because other permission sources may exist.

## Console and CLI Comparison
If the IAM console displays LabRole details, the console information should be compared with the CLI results. Any differences, missing information, or AccessDenied conditions should be documented. The actual results from CloudShell should be used instead of guessing what the role contains.

## Why LabRole Is Not the Client Solution
LabRole is part of the AWS Academy Learner Lab environment. It is useful for practicing identity and access investigations, but it should not be recommended as Marcus's production access solution.
Marcus needs access to a specific inventory bucket and selected inventory objects. The proper solution is an approved least-privilege group or role for Riverside Goods. The LabRole trust relationship and permission sources do not establish that it is appropriate for Marcus or that it meets his business requirements.

## Lessons Learned
This week taught me that authentication and authorization are separate parts of AWS security. A user can successfully sign in and still be denied when trying to access a resource. IAM permissions determine which actions an identity can perform.
I also learned that least privilege is important because users should only receive the access required for their jobs. Giving full S3 access to Marcus would create unnecessary permissions and could increase the risk of unauthorized access to other resources.
The AWS CLI is useful for collecting evidence about the current identity and permission configuration. CloudShell provides a way to practice these commands without creating personal access keys. I also learned that the Learner Lab environment has its own restrictions, so its IAM configuration should not automatically be treated as the same as the client's production account.

## Professional Vocabulary
-Authentication: The process of verifying a user's identity before allowing access.
-Authorization: The process of determining which actions an authenticated identity is allowed to perform.
-IAM: AWS Identity and Access Management, which controls access to AWS resources.
-Policy: A document that defines which actions are allowed or denied for an AWS identity or resource.
-Least Privilege: Giving a user only the permissions needed to complete their assigned job.
-AccessDenied: An AWS error that means the requested action was not allowed under the current permissions.
-AWS CLI: A command-line tool used to interact with AWS services.
-CloudShell: A browser-based command-line environment provided by AWS.
-Caller Identity: The AWS account and identity associated with the current request.
-Resource Scope: The specific AWS resources that a permission applies to, such as a particular S3 bucket or its objects.

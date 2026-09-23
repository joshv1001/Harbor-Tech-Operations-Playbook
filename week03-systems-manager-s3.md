# Week 3: Systems Manager and S3

## HarborTech Ticket Summary

Ticket TKT-2026-0003 focuses on Bright Path Community Services and its need to reduce repetitive administrative work. Dana currently has to perform the same maintenance process across five EC2 instances. Bright Path also needs a simple public webpage for program hours, contact information, images, and downloadable forms without adding another server.

The goal of this investigation was to determine which AWS services would reduce manual work while still keeping the environment simple and manageable.

## Client Impact

Repeated manual administration can take a lot of time when the same task has to be completed on multiple EC2 instances. Manual work can also lead to inconsistent results if an administrator accidentally skips a step or performs the task differently on one system.

Using centralized management can make repeated tasks more consistent and easier to verify. It can also reduce the need for administrators to directly access individual servers.

For Bright Path's public webpage, using S3 avoids the additional maintenance involved with running another EC2 web server. This keeps the design simpler for a website that only needs static content.

## AWS Services Involved

The main AWS services and concepts reviewed during Week 3 were:

* AWS Systems Manager — Provides centralized tools for managing AWS resources and managed instances.
* Run Command — Allows commands to be sent to managed EC2 instances without manually logging into each server.
* Session Manager — Provides interactive access when an administrator actually needs to work directly with an instance.
* Inventory — Collects information about managed instances, installed software, and system configuration.
* Parameter Store — Provides a central location for storing configuration values.
* Amazon S3 — Provides object storage for files and static content.
* Static Website Hosting — Allows static website files such as HTML and images to be served from an S3 bucket.

## Virtualization Connection

Systems Manager provides a centralized management layer for virtual machines such as EC2 instances. Instead of connecting individually to every virtual machine, an administrator can use Systems Manager to run commands and collect information from managed instances.

This is especially useful when several virtual machines need the same maintenance operation. It reduces repetitive work and provides a central location for reviewing the results.

S3 provides a different approach because it stores objects instead of requiring a traditional web server. For Bright Path's simple webpage, S3 can provide the required static content without creating another virtual machine that HarborTech would have to maintain.

## Evidence Reviewed

During the Week 3 investigation, I reviewed the requirements for Dana's repeated maintenance task and the Bright Path static website.

The investigation showed that:

* Dana's maintenance process affects five EC2 instances.
* The maintenance command needs to be performed consistently across the instances.
* The task is repeated, making it a potential candidate for centralized management.
* Systems Manager requires instances to be properly configured as managed nodes.
* The SSM Agent needs to be available and running.
* The instances require the appropriate IAM permissions and connectivity.
* Run Command is appropriate for repeated non-interactive commands.
* Session Manager is more appropriate when interactive troubleshooting is necessary.
* Parameter Store can centralize environment-specific configuration values.
* Bright Path's website only requires static content.
* An S3 bucket can store the HTML website and other public files.
* CloudShell can be used to run AWS CLI commands for verification and file updates.

### S3 Investigation Evidence

The S3 investigation resulted in the following configuration:

* Bucket: brightpath-site-jv-2026
* Region: us-west-2
* Object Key: index.html
* Index Document: index.html
* Static Website Hosting: Enabled
* Website Endpoint: S3-generated website endpoint for the bucket
* Website Access Test: 403 Forbidden

After enabling static website hosting, I tested the S3 website endpoint. The request returned a 403 Forbidden response. This indicates that the Learner Lab environment did not allow the website content to be accessed publicly with the current configuration.

I documented the 403 response rather than attempting to bypass the Learner Lab's access restrictions. In a normal AWS account, the bucket's public-access configuration and appropriate permissions would need to be reviewed and approved before allowing public access to the website.

I also used CloudShell to verify the AWS identity associated with the Learner Lab session using:

aws sts get-caller-identity

The S3 contents were verified with:

aws s3 ls s3://brightpath-site-jv-2026/

The expected index.html object was listed in the bucket.

The website content was updated using:

aws s3 sync ./brightpath-site s3://brightpath-site-jv-2026/

The CLI operation provided evidence that the local website directory could be synchronized with the S3 bucket, even though the Learner Lab restricted public access to the website endpoint.

## Operational Analysis

The repeated EC2 maintenance task is better suited for AWS Systems Manager Run Command because the same operation needs to be performed across five instances. Manually logging into every instance would take more time and could result in inconsistent changes.

Run Command allows HarborTech to centrally send the same command to the required managed instances. The results can then be reviewed to determine whether each instance completed the operation successfully.

For situations where an administrator needs to interact directly with a system, Session Manager would be more appropriate. There is no need to use interactive access for every routine maintenance task when a non-interactive command can accomplish the work.

Parameter Store is appropriate for configuration values that are repeated throughout an environment. Instead of maintaining the same value in several files, the application or automation can retrieve the value using its parameter name. The application still needs to be changed to retrieve the value because Parameter Store does not automatically rewrite existing files.

The Bright Path website is a good fit for S3 static website hosting because it only requires static HTML, images, and downloadable files. Using an EC2 instance would introduce additional operating system and server maintenance that is not necessary for this requirement.

## Recommendation

For Dana's repeated EC2 maintenance, I recommend AWS Systems Manager Run Command. The task is repeated across five instances, so centralized command execution is more practical than manually accessing each system.

Before using Run Command, HarborTech should verify that the EC2 instances are registered as Systems Manager managed nodes, the SSM Agent is functioning, the required IAM permissions are available, and the instances have the necessary connectivity to Systems Manager.

For shared configuration values, I recommend Parameter Store. It provides one central location for the value and allows applications or automation to retrieve it by parameter name.

For Bright Path's public resource page, I recommend Amazon S3 static website hosting. The website only needs static files, so S3 provides the needed storage and hosting without requiring another EC2 server.

## Escalation Notes

The main items that could require additional support are Systems Manager prerequisites and Learner Lab restrictions.

If an EC2 instance is not showing as a managed node, HarborTech should check the SSM Agent, IAM permissions, instance configuration, and connectivity before attempting to run maintenance commands.

For the S3 website, The S3 website test returned a 403 Forbidden response when I attempted to access the static website endpoint. I treated this as a Learner Lab access restriction and did not attempt to bypass the restriction.

In a normal AWS environment, HarborTech would need to review the S3 bucket's public-access settings and the permissions required for public website access. Any changes should follow the organization's security and access-control requirements.

The EC2 instances used for Systems Manager would also need to meet the required managed-node, SSM Agent, IAM permission, and connectivity requirements before Run Command could be used successfully.

## Lessons Learned

Week 3 helped me understand that automation should be used when it actually improves the operation. A task that is repeated across several systems is a good candidate because automation can save time and make the results more consistent.

I also learned that Systems Manager has different features for different situations. Run Command is useful for repeatable commands, while Session Manager is better when interactive access is necessary. Parameter Store can help centralize configuration values instead of keeping duplicate hardcoded values in multiple places.

The S3 portion of the lab also showed me that not every website needs a traditional server. A simple static website can be hosted using S3, which can reduce the amount of infrastructure that HarborTech needs to maintain.

## Professional Vocabulary

-Systems Manager: An AWS service that provides centralized tools for managing and operating resources such as EC2 instances.

-Managed Node: An instance or system that has been configured so Systems Manager can communicate with and manage it.

-Run Command: A Systems Manager feature used to remotely execute commands on managed instances.

-Session Manager: A feature that provides interactive access to managed instances without requiring traditional remote-access methods.

-Inventory: A Systems Manager feature that collects information about systems, software, and configuration.

-Parameter Store: A service that provides centralized storage for configuration values that applications and automation can retrieve.

-Automation: Using a defined process or technology to perform repeated tasks with less manual effort.

-Static Website Hosting: Hosting website files such as HTML, CSS, images, and downloadable documents without requiring server-side application processing.

-Object Storage: A method of storing individual files as objects, such as files stored inside an Amazon S3 bucket.

-Management Plane: The part of a cloud environment used to configure, control, monitor, and manage resources.

## Week 3 Operational Evidence Summary

The investigation demonstrated that the Bright Path website could be represented as static content in Amazon S3 and that the AWS CLI could be used to verify and update the website files.

The S3 bucket was created in the permitted us-west-2 region, with index.html used as the website's index document. CloudShell was used to verify the AWS identity and perform S3 commands. The S3 listing confirmed that the website object was present.

For Dana's EC2 maintenance requirement, Systems Manager Run Command was identified as the appropriate management capability because the same command needs to be executed repeatedly across five EC2 instances. The instances must meet the Systems Manager managed-node prerequisites before HarborTech should expect the operation to work successfully.

Overall, the Week 3 investigation showed how HarborTech can reduce unnecessary manual administration by selecting the appropriate AWS management or hosting service instead of automatically adding more servers or creating automation where it is not needed.

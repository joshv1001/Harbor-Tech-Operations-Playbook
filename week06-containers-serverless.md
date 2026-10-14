# Week 6: Containers and Serverless Computing

## HarborTech Ticket Summary

**Ticket:** TKT-2026-0006  
**Client:** Bright Path Community Services  
**Team:** HarborTech Operations

Bright Path currently runs a daily reporting task and a small API on one EC2 instance. The application works, but the server stays online even when there is not much work being done. The goal is to find out if there is a better way to run each workload without creating unnecessary costs or maintenance.

I recommend looking at serverless computing for the daily report and API, while keeping EC2 or considering containers for applications that need continuous operation or more operating system control.

## Client Impact

Keeping a server running when it is barely being used can waste resources and increase operating costs. The team also has to maintain the operating system, install updates, monitor the server, and troubleshoot problems.

Choosing a better compute model could reduce maintenance and help the applications use resources more efficiently. However, the new design still needs to be tested to make sure it meets the client's performance, security, and reliability requirements.

## AWS Services Involved

- **Amazon EC2:** Runs virtual machines and provides control over the guest operating system.
- **Containers:** Package an application and its dependencies so they can run in a consistent environment.
- **Amazon ECS with AWS Fargate:** Runs containers without requiring the team to manage the underlying servers.
- **AWS Lambda:** Runs application code when a request or event triggers it.
- **Amazon API Gateway:** Receives API requests and can send them to Lambda functions.
- **Amazon EventBridge Scheduler:** Runs scheduled tasks, such as a report that needs to execute every morning.
- **AWS Step Functions:** Coordinates multiple steps in a workflow when a process needs more than one function or action.
- **AWS IAM execution roles:** Give Lambda functions permission to access the AWS resources they need.
- **Amazon CloudWatch:** Collects logs and metrics and helps identify errors or performance problems.
- **Amazon VPC:** Provides network isolation and connectivity. A VPC-connected Lambda function can access configured private resources when the networking and permissions are set up correctly.

## Virtualization Connection

Virtual machines, containers, and serverless computing all provide ways to run applications without managing physical hardware directly, but they work at different levels.

With EC2, the team gets a virtual machine with its own guest operating system. This gives more control but also means the team must handle operating system updates and maintenance.

Containers package the application and its dependencies while sharing the host operating system's kernel. They can make deployments more consistent, but the container images and application still need maintenance.

Serverless computing takes this a step further by allowing AWS to manage the underlying server infrastructure. With Lambda, the team focuses more on the code, permissions, configuration, and monitoring instead of maintaining a server.

The main difference is how much infrastructure the team needs to manage and how much control it requires.

## Evidence Reviewed

Based on the Week 6 ticket description, the known workload evidence is:

- The reporting task runs once each morning.
- The report completes quickly and does not keep local state between runs.
- The API receives requests intermittently throughout the day.
- Both workloads currently share one EC2 instance.
- The instance spends much of its time idle.
- The current application is working, so a migration should not be made without testing.

Before making a final decision, I would collect additional evidence, including:

- EC2 CPU and memory utilization, running hours, and current costs.
- The report's actual execution time, dependencies, and output destination.
- API request volume, response times, authentication requirements, and error rates.
- Lambda test results, execution logs, errors, and timeout behavior.
- IAM permissions and whether the application needs access to private VPC resources.
- Whether the application requires a persistent process or direct operating system access.
- Whether the workload has multiple steps that would benefit from Step Functions.

I have not verified these items in the AWS environment yet, so they should be treated as follow-up checks rather than completed tests.

## Operational Analysis

The daily report looks like a good fit for Lambda because it runs on a schedule, finishes quickly, and does not need to keep local state between executions. EventBridge Scheduler could trigger the function each morning.

The API could also work with Lambda and API Gateway. API Gateway would receive requests, and Lambda would execute the application logic and return the response. This would reduce the need to keep a dedicated application server running, assuming the API's dependencies and performance requirements are compatible.

EC2 is still useful for applications that need a full guest operating system, continuous processes, or host-level troubleshooting. Containers are another option when an application needs consistent packaging or a specific runtime environment. ECS with Fargate reduces host management but does not remove responsibility for the application and container configuration.

Step Functions would be useful if the reporting process eventually required several coordinated steps, such as generating a report, validating the results, and storing the output. For a single short task, adding a workflow service may not be necessary.

The best choice depends on how the application runs, what it needs access to, and how much maintenance the team is willing to handle.

## Recommendation

I recommend testing AWS Lambda with EventBridge Scheduler for the daily reporting task and evaluating API Gateway with Lambda for the intermittent API.

This setup could reduce idle compute usage and the need to maintain the EC2 operating system for these workloads. AWS would manage the underlying server infrastructure, while HarborTech would still maintain the code, dependencies, permissions, deployment process, monitoring, and error handling.

For applications that need continuous execution and operating system access, I would keep EC2 as the initial option. I would also evaluate ECS with Fargate if the application can run in a container and does not require direct host access.

Before selecting the final design, I would compare actual usage and estimated monthly costs, test the applications, and verify their security and networking requirements. The recommendation is based on the available ticket information, and the final design should depend on the results of those checks.

## Escalation Notes

As an intern, I can review the workload, run approved tests, document the results, and present a recommendation. I should not independently change the production architecture.

I would escalate the following items to the appropriate HarborTech technical lead:

- Approval to migrate the reporting task or API.
- IAM permission changes or access to private network resources.
- Application changes needed to support Lambda or containers.
- Production API routing changes and deployment decisions.
- Final cost review, security approval, and the rollback plan.

The existing EC2 instance should not be shut down until the replacement has been tested, approved, and confirmed to meet the client's requirements.

## Lessons Learned

This assignment helped me understand that not every application needs to run on a virtual machine. A short task that runs once a day may work better with Lambda, while an API that receives occasional requests could use API Gateway and Lambda.

I also learned that containers are useful for packaging applications and their dependencies, but they still require maintenance. Serverless reduces server administration, but the team is still responsible for application security, permissions, monitoring, and error handling.

The most important lesson is to choose the compute model based on the workload instead of using the same solution for everything. Testing the application, checking costs, and getting approval before production changes are also important parts of the process.

## Professional Vocabulary

- **Container:** A package containing an application and the dependencies it needs to run.
- **Serverless Computing:** A model where AWS manages the underlying server infrastructure while the team focuses on running its application.
- **AWS Lambda:** A service that runs code in response to events or requests without requiring the team to maintain a server.
- **API Gateway:** A service that provides an entry point for API requests and can route them to application backends.
- **Execution Role:** An IAM role that gives a service or function permission to perform approved actions on AWS resources.
- **Event Trigger:** An event or scheduled action that starts a function or process.
- **Step Functions:** A service that coordinates multiple steps in an application workflow.
- **VPC-Connected Lambda:** A Lambda function configured to connect to resources in a VPC, subject to the required network and security configuration.
- **Runtime:** The environment used to execute application code, such as a supported Python or Java runtime.
- **Abstraction:** Hiding lower-level technical details so users can work with a simpler interface or service.
- **Virtual Machine:** A software-based computer with its own guest operating system.
- **ECS:** Amazon Elastic Container Service, which manages containerized applications.
- **Fargate:** A managed compute option for running containers without managing the underlying servers.
- **CloudWatch:** An AWS monitoring service for logs, metrics, and alarms.
- **IAM:** AWS Identity and Access Management, which controls who or what can access AWS resources.

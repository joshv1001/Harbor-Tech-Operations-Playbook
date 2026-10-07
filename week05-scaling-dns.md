# Week 5: Scaling, Load Balancing, and DNS

## HarborTech Ticket Summary

TKT-2026-0005 involves Riverside Goods preparing for a seasonal increase in customer traffic. The current application depends on one EC2 instance and one public endpoint. This creates both a capacity limitation and an availability risk because the single instance could become overloaded or unavailable.

The ticket states that previous promotion traffic reached 92% CPU utilization and that the upcoming promotion could double request volume. HarborTech needs to evaluate how additional compute capacity, traffic distribution, health checks, and DNS routing could reduce these risks.

My role as an intern is to investigate the available AWS evidence, analyze the operational risks, and recommend the appropriate controls. I am not approving or implementing production architecture changes.

## Client Impact

The expected traffic increase could place more demand on the existing EC2 instance. If the instance reaches its resource limits, customers could experience slower response times, errors, or an unavailable application.

The single-instance design also creates a single point of failure. If the EC2 instance becomes unavailable, there is no additional application instance available to handle customer requests.

This could affect Riverside Goods by causing lost customer transactions, poor customer experience, and potential business disruption during an important seasonal promotion.

## Provided Ticket Evidence

The following information was provided by the HarborTech ticket and was not personally verified in my AWS environment:

* Previous promotion traffic reached 92% CPU utilization.
* Upcoming promotion traffic is expected to increase sharply and could double request volume.
* Riverside Goods currently depends on one EC2 instance and one public endpoint.
* The proposed Auto Scaling configuration is:

  * Minimum: 2
  * Desired: 2
  * Maximum: 6
* The proposed Application Load Balancer reportedly has two healthy test targets.
* A secondary recovery endpoint is reportedly available.
* Route 53 failover has not been confirmed.

This information was treated as ticket evidence and was kept separate from the results returned by my AWS investigations.

## AWS Commands Used

### Investigation 1 — Auto Scaling Groups

```bash
aws autoscaling describe-auto-scaling-groups \
  --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' \
  --output table
```

### Investigation 2 — Target Groups

```bash
aws elbv2 describe-target-groups \
  --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' \
  --output table
```

The target health command was not executed because the AWS environment returned no target groups. The assignment specifically instructed me to stop the investigation if no target group was returned.

### Investigation 3 — Route 53 Hosted Zones

```bash
aws route53 list-hosted-zones \
  --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' \
  --output table
```

## AWS Evidence Collected

My AWS Learner Lab returned no Auto Scaling groups:

```text
~ $ aws autoscaling describe-auto-scaling-groups \
>   --query 'AutoScalingGroups[].{Name:AutoScalingGroupName,Min:MinSize,Desired:DesiredCapacity,Max:MaxSize}' \
>   --output table
~ $
```

No Auto Scaling group name, minimum capacity, desired capacity, or maximum capacity was returned.

My AWS environment also returned no target groups:

```text
~ $ aws elbv2 describe-target-groups \
>   --query 'TargetGroups[].{Name:TargetGroupName,TargetGroupArn:TargetGroupArn,Protocol:Protocol,Port:Port}' \
>   --output table
~ $
```

Because no target group was returned, I could not personally verify target membership or target health.

Finally, my AWS environment returned no Route 53 hosted zones:

```text
~ $ aws route53 list-hosted-zones \
>   --query 'HostedZones[].{Name:Name,Id:Id,Private:Config.PrivateZone}' \
>   --output table
~ $
```

Therefore, I could not verify a Route 53 hosted zone, DNS records, health checks, or failover routing configuration in my assigned environment.

## Virtualization Connection

EC2 provides virtual compute resources that can be created and replaced without depending on one physical server. This makes it possible to treat application capacity as replaceable resources rather than permanently relying on a single machine.

Auto Scaling can use a repeatable launch configuration to add or remove EC2 instances as demand changes. A load balancer can distribute requests across those virtual instances instead of sending all customer traffic to one server.

This approach allows Riverside Goods to expand capacity during a traffic surge and reduce unnecessary capacity when demand decreases. It also improves availability because the application does not have to depend on one individual EC2 instance.

## Operational Analysis

The ticket evidence shows a significant capacity concern. Previous traffic already reached 92% CPU utilization, and the upcoming promotion is expected to increase request volume substantially. This supports investigating an Auto Scaling design with a minimum of 2, desired capacity of 2, and maximum capacity of 6.

The minimum capacity represents the lowest number of instances that should normally be maintained. Desired capacity represents the number of instances the Auto Scaling group attempts to maintain during normal operation. Maximum capacity establishes the upper limit for automatically adding instances.

However, my AWS investigation returned no Auto Scaling groups. Therefore, the proposed 2 / 2 / 6 configuration is not something I could verify in my assigned environment.

Capacity alone would not solve the traffic distribution problem. If multiple EC2 instances were created but customers still connected directly to only one endpoint or instance, the application could still have an uneven workload or a single endpoint failure. An Application Load Balancer would provide a central traffic endpoint and distribute requests among registered targets.

The ticket states that two proposed targets are healthy. However, my AWS investigation returned no target groups, so I could not verify the target health information myself. Target health would need to be verified after an approved load-balancing configuration is available.

The Route 53 investigation also returned no hosted zones. Therefore, I cannot claim that DNS failover is configured or ready. The ticket's statement that a secondary recovery endpoint exists should be treated as provided evidence until it is independently verified.

The main remaining unknowns are the actual production Auto Scaling configuration, load balancer and target configuration, health-check settings, primary and secondary endpoints, and any Route 53 failover records or health checks.

## Recommendation

Based on the evidence, HarborTech should prepare Riverside Goods for the seasonal traffic increase by evaluating an EC2 Auto Scaling group and an Application Load Balancer.

The proposed Auto Scaling range of 2 minimum, 2 desired, and 6 maximum provides a reasonable starting point for addressing the capacity risk identified in the ticket. The exact scaling metric and thresholds should be validated using application monitoring data before production implementation.

An Application Load Balancer should be used to distribute customer requests across healthy application instances. Target health checks should be configured so that unhealthy instances are removed from normal traffic while the Auto Scaling group maintains the required capacity.

I would not recommend claiming that Route 53 failover is currently configured because my AWS investigation returned no hosted zones. Route 53 failover should only be considered after the primary endpoint, secondary endpoint, DNS records, health checks, and failover routing have been verified and approved.

This approach addresses the two main risks without automatically adding every AWS availability service. Auto Scaling addresses capacity, the load balancer addresses traffic distribution and endpoint dependency, and health checks help maintain availability. DNS failover can be added later if the architecture and business requirements justify it.

## Escalation Notes

Production changes should be escalated for approval before implementation.

The following items require review or approval:

* Creation or modification of an EC2 Auto Scaling group.
* Selection of minimum, desired, and maximum capacity.
* Scaling policy and threshold configuration.
* Creation or modification of an Application Load Balancer.
* Target group and health-check configuration.
* Security group or network changes required by the load balancer.
* Creation or modification of Route 53 DNS records.
* Configuration of Route 53 health checks or failover routing.
* Validation of the secondary recovery endpoint.

As an intern, I can document the evidence and recommend the operational response, but I should not present myself as having approval to make these production changes.

## Lessons Learned

Week 5 showed me that cloud operations should be based on actual evidence instead of assuming that a service is already configured. The AWS CLI investigations showed that my assigned environment did not contain an Auto Scaling group, target group, or Route 53 hosted zone.

I also learned that scaling and availability are related but different problems. Auto Scaling can provide additional compute capacity, but it does not automatically distribute customer traffic. A load balancer can distribute traffic and use health checks, but healthy targets do not automatically provide DNS failover.

Another important lesson was the difference between ticket evidence and AWS evidence. The ticket provided the business situation and proposed design, while the AWS CLI commands showed what resources I could actually verify in my environment.

This investigation reinforced the importance of checking the current state first, documenting the evidence, identifying unknowns, and escalating production changes for proper approval.

## Professional Vocabulary

**Elasticity:** The ability of cloud resources to increase or decrease as workload demand changes.

**Scalability:** The ability of an application or system to handle increasing workload by adding resources or improving its capacity.

**Load Balancer:** A service that receives application traffic and distributes it across available backend resources.

**Target Group:** A collection of targets, such as EC2 instances, that can receive traffic from a load balancer.

**Health Check:** A test used to determine whether a target or endpoint is functioning well enough to receive traffic.

**Auto Scaling Group:** An AWS service configuration that maintains and adjusts a group of EC2 instances based on defined capacity and scaling rules.

**Launch Template:** A reusable EC2 configuration that defines how new instances should be launched.

**Minimum Capacity:** The lowest number of instances an Auto Scaling group is allowed to maintain.

**Desired Capacity:** The number of instances an Auto Scaling group attempts to maintain under normal conditions.

**Maximum Capacity:** The highest number of instances an Auto Scaling group is allowed to launch.

**Route 53:** AWS's DNS service used to manage domain name resolution and routing policies.

**Failover:** A method of directing traffic from a primary resource to a secondary resource when the primary becomes unavailable.

**Availability:** The ability of an application or service to remain accessible and operational when failures occur.

**Capacity:** The amount of workload that a system can handle before its resources become insufficient.

**Single Point of Failure:** A component whose failure can cause the entire application or service to become unavailable.

**Target Health:** The condition reported by a load balancer's health checks that determines whether a target should receive traffic.

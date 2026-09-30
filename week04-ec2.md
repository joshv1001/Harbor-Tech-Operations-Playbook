# Week 4: EC2 Evidence Lab

## HarborTech Ticket Summary

**Ticket:** TKT-2026-0004
**Issue:** The Riverside Goods EC2 server showed as `Running`, but users could not reach the web application.

The purpose of this lab was to prove whether the server was actually healthy instead of assuming that the EC2 `Running` state meant the application was reachable. I provisioned a disposable Amazon Linux EC2 web server, tested it from outside and inside the instance, created a controlled reachability failure, diagnosed the failure using AWS and guest evidence, corrected only the affected layer, verified the result, tested stop/start lifecycle behavior, and cleaned up the resources.

The final root cause was a missing inbound security-group rule for TCP port 80.

---

## Client Impact

The simulated Riverside Goods web application was unavailable from the external client path. The EC2 instance itself was running and AWS status checks were passing, but HTTP requests to the public IP timed out.

The impact was limited to the lab workload. The evidence showed that the Apache web server was functioning locally inside the instance, so the application itself was not the primary failure.

---

## Environment and Resource Names

The workload was created in the AWS Learner Lab using CloudShell.

* **Region:** `us-east-1`
* **VPC:** `vpc-00f98d3086eb0453f`
* **Subnet:** `subnet-0ef323cf996fe8cda`
* **Availability Zone:** `us-east-1d`
* **Subnet CIDR:** `172.31.16.0/20`
* **Instance name:** `HarborTech-Week4-Riverside-26192`
* **Instance ID:** `i-0cf40fe46c1b6c722`
* **Instance type:** `t3.micro`
* **AMI:** `ami-0b245cc5f82576748`
* **Instance profile:** `LabInstanceProfile`
* **Security group:** `harbortech-week4-riverside-10317`
* **Security group ID:** `sg-0d288b3f3dc79b923`
* **Initial private IP:** `172.31.22.15`
* **Initial public IP:** `18.212.217.137`

The instance was created as a disposable lab workload and was not intended to hold production data.

---

## AWS Documentation Evidence

I used the official **Amazon Elastic Compute Cloud User Guide for Linux Instances** as the main AWS documentation source. AWS's current EC2 documentation identifies the Amazon EC2 User Guide as the main guide for configuring and using EC2.

### Security Groups

**Document:** Amazon Elastic Compute Cloud User Guide for Linux Instances
**PDF page:** 6

**Exact quotation recorded for the lab:**

> “Security groups act as a firewall for associated instances, controlling both inbound and outbound traffic at the instance level.”

This applied directly to the incident because the security group controlled whether external HTTP traffic could reach the EC2 instance. The instance was running, but the security group initially had no inbound rules.

### User Data

**Document:** Amazon Elastic Compute Cloud User Guide for Linux Instances
**PDF page:** 409

**Exact quotation recorded for the lab:**

> “User data is treated as opaque data: what you give is what you get back.”

I used user data to install Apache and create the Riverside Goods web page. However, user data alone was not treated as proof that the application was healthy. I still verified the service from inside the instance and tested the application from the external client path.

### IMDSv2 and Lifecycle

**Document:** Amazon Elastic Compute Cloud User Guide for Linux Instances
**PDF page:** 759

**Exact quotation recorded for the lab:**

> “IMDSv2 uses session-oriented requests.”

I used an IMDSv2 token to retrieve the instance ID from inside the EC2 guest. This provided additional evidence that the instance metadata service was accessible and that the metadata request process was working.

AWS documentation also explains that a security group controls which protocols, ports, and source IP ranges can reach an EC2 instance.

---

## CloudShell Command Record

### Set the AWS Region

```bash
export AWS_REGION=us-east-1
export AWS_DEFAULT_REGION=us-east-1
```

### Identify the Instance

```bash
INSTANCE_ID="i-0cf40fe46c1b6c722"
```

### Check the Security Group Before Correction

```bash
aws ec2 describe-security-groups \
  --group-ids sg-0d288b3f3dc79b923 \
  --query 'SecurityGroups[0].IpPermissions' \
  --output json
```

Output:

```text
[]
```

I also specifically checked port 80:

```bash
aws ec2 describe-security-groups \
  --group-ids sg-0d288b3f3dc79b923 \
  --query 'SecurityGroups[0].IpPermissions[?FromPort==`80`]' \
  --output json
```

Output:

```text
[]
```

### External HTTP Test

```bash
PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

curl -I --connect-timeout 10 "http://$PUBLIC_IP"
```

The request timed out:

```text
curl: (28) Connection timed out after 10001 milliseconds
```

### Add the Corrective Security Rule

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-0d288b3f3dc79b923 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

AWS returned:

```text
"Return": true
"SecurityGroupRuleId": "sgr-03122bbb038f834fc"
"GroupId": "sg-0d288b3f3dc79b923"
"IpProtocol": "tcp"
"FromPort": 80
"ToPort": 80
"CidrIpv4": "0.0.0.0/0"
```

### Verify the Security Rule

```bash
aws ec2 describe-security-groups \
  --group-ids sg-0d288b3f3dc79b923 \
  --query 'SecurityGroups[0].IpPermissions[?FromPort==`80`]' \
  --output json
```

Output showed:

```text
"IpProtocol": "tcp"
"FromPort": 80
"ToPort": 80
"CidrIp": "0.0.0.0/0"
```

### Final HTTP Test

```bash
curl -I --connect-timeout 10 "http://$NEW_PUBLIC_IP"
```

Result:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)
Content-Length: 170
Content-Type: text/html; charset=UTF-8
```

---

## Baseline Evidence

The instance initially showed:

```text
Instance state: running
System status: ok
Instance status: ok
```

The initial public IP was:

```text
18.212.217.137
```

The first external HTTP request timed out:

```text
curl: (28) Connection timed out after 10001 milliseconds
```

The important control-plane evidence was that the security group had no inbound rules:

```text
[]
```

The port 80 query also returned:

```text
[]
```

This was significant because the web application was supposed to receive HTTP traffic on TCP port 80.

---

## Root-Cause Analysis

I did not assume that the `Running` state meant the application was healthy. I compared evidence from different layers.

From inside the instance, I ran:

```bash
curl -I http://localhost
```

The result was:

```text
HTTP/1.1 200 OK
Server: Apache/2.4.68 (Amazon Linux)
Content-Length: 170
Content-Type: text/html; charset=UTF-8
```

I also checked the listening port:

```bash
sudo ss -lntp | grep ':80'
```

The result showed Apache listening:

```text
LISTEN 0 511 *:80 *:* users:(("httpd",pid=4063,fd=4),("httpd",pid=3986,fd=4),("httpd",pid=3980,fd=4),("httpd",pid=3864,fd=4))
```

This ruled out an Apache service failure as the primary cause.

The external request was failing while the local request worked. At the same time, the security group had no inbound rule for port 80. The evidence therefore pointed to the security-group layer as the root cause.

---

## Corrective Action

I made the smallest corrective change supported by the evidence.

I added TCP port 80 to the security group:

```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-0d288b3f3dc79b923 \
  --protocol tcp \
  --port 80 \
  --cidr 0.0.0.0/0
```

I did not rebuild the instance, reinstall the operating system, or change the Apache configuration because the existing workload was already functioning locally.

The security rule allowed HTTP access from any IPv4 address. This was acceptable for this disposable lab because the assignment specifically required external reachability testing. I would not make the same change to a production workload without authorization because `0.0.0.0/0` exposes the port to all IPv4 addresses.

---

## Verification Evidence

After adding the rule, I verified the security group again.

The port 80 rule was present with:

```text
IpProtocol: tcp
FromPort: 80
ToPort: 80
CidrIp: 0.0.0.0/0
```

I then tested the new public IP:

```bash
curl -I --connect-timeout 10 "http://$NEW_PUBLIC_IP"
```

The response was:

```text
HTTP/1.1 200 OK
Date: Wed, 30 Sep 2026 03:45:53 GMT
Server: Apache/2.4.68 (Amazon Linux)
Last-Modified: Wed, 30 Sep 2026 03:08:18 GMT
ETag: "aa-65caa9c60c364"
Accept-Ranges: bytes
Content-Length: 170
Content-Type: text/html; charset=UTF-8
```

I also requested the page itself:

```bash
curl --connect-timeout 10 "http://$NEW_PUBLIC_IP"
```

The Riverside Goods page was returned:

```html
<!DOCTYPE html>
<html>
<head>
<title>Riverside Goods</title>
</head>
<body>
<h1>Riverside Goods Web Server</h1>
<p>HarborTech Week 4 EC2 Evidence Lab</p>
</body>
</html>
```

This provided both control-plane verification and workload-path verification.

---

## IMDSv2 and Guest Evidence

I connected to the instance through AWS Systems Manager Session Manager:

```bash
aws ssm start-session --target "i-0cf40fe46c1b6c722"
```

Inside the instance, I requested an IMDSv2 token:

```bash
TOKEN=$(curl -X PUT \
  "http://169.254.169.254/latest/api/token" \
  -H "X-aws-ec2-metadata-token-ttl-seconds: 21600")
```

I then used the token to retrieve the instance ID:

```bash
INSTANCE_ID_IMDS=$(curl \
  -H "X-aws-ec2-metadata-token: $TOKEN" \
  http://169.254.169.254/latest/meta-data/instance-id)

echo "Instance ID from IMDSv2: $INSTANCE_ID_IMDS"
```

The output was:

```text
Instance ID from IMDSv2: i-0cf40fe46c1b6c722
```

This matched the EC2 instance ID from the AWS control plane.

The guest-side Apache test also returned `200 OK`, and the listening-port check showed `httpd` listening on port 80. These checks helped separate application health from network reachability.

---

## Stop/Start Lifecycle Test

I tested the lifecycle behavior of the same instance.

Before stopping it, I checked the public IP:

```bash
INSTANCE_ID="i-0cf40fe46c1b6c722"

PUBLIC_IP=$(aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].PublicIpAddress' \
  --output text)

echo "Instance ID: $INSTANCE_ID"
echo "Public IPv4 before stop: $PUBLIC_IP"

curl "http://$PUBLIC_IP"
```

The recorded public IP was:

```text
Instance ID: i-0cf40fe46c1b6c722
Public IPv4 before stop: 18.212.217.137
```

I stopped the instance:

```bash
aws ec2 stop-instances --instance-ids "$INSTANCE_ID"
```

AWS reported:

```text
CurrentState: shutting-down
PreviousState: running
```

I waited until it was fully stopped:

```bash
aws ec2 wait instance-stopped --instance-ids "$INSTANCE_ID"
```

Then I checked the state and addresses:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress]' \
  --output table
```

The result showed:

```text
i-0cf40fe46c1b6c722
stopped
None
172.31.22.15
```

I then started the same instance:

```bash
aws ec2 start-instances --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-running --instance-ids "$INSTANCE_ID"
aws ec2 wait instance-status-ok --instance-ids "$INSTANCE_ID"
```

After the restart, I checked again:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].[InstanceId,State.Name,PublicIpAddress,PrivateIpAddress]' \
  --output table
```

The result showed:

```text
i-0cf40fe46c1b6c722
running
54.146.209.144
172.31.22.15
```

The important observation was that the instance ID stayed the same and the private IP stayed the same, but the public IP changed from `18.212.217.137` to `54.146.209.144`.

After the restart, HTTP initially timed out because the security group still had no inbound rule at that point. After adding TCP port 80, HTTP returned `200 OK`.

This demonstrated why lifecycle state and network reachability must be tested separately.

---

## Cleanup Evidence

After all testing was complete, I terminated the disposable EC2 instance:

```bash
aws ec2 terminate-instances --instance-ids "$INSTANCE_ID"
```

The actual termination response showed:

```text
InstanceId: i-0cf40fe46c1b6c722
CurrentState:
    Code: 32
    Name: shutting-down
PreviousState:
    Code: 16
    Name: running
```

I waited for termination to complete:

```bash
aws ec2 wait instance-terminated --instance-ids "$INSTANCE_ID"
```

I verified the final instance state:

```bash
aws ec2 describe-instances \
  --instance-ids "$INSTANCE_ID" \
  --query 'Reservations[0].Instances[0].State.Name' \
  --output text
```

The result was:

```text
terminated
```

After the instance was terminated, I deleted the lab security group:

```bash
aws ec2 delete-security-group \
  --group-id sg-0d288b3f3dc79b923
```

AWS returned:

```text
{
    "Return": true,
    "GroupId": "sg-0d288b3f3dc79b923"
}
```

I performed a final verification:

```bash
aws ec2 describe-security-groups \
  --group-ids sg-0d288b3f3dc79b923
```

AWS returned:

```text
InvalidGroup.NotFound
```

This was not an unresolved failure. It confirmed that the security group no longer existed.

The disposable EC2 instance and lab security group were therefore cleaned up successfully.

---

## Escalation and Change-Control Notes

No escalation was required for the lab because the issue was identified, corrected, verified, and cleaned up.

The corrective action was limited to the security-group layer because the evidence supported that layer. Rebuilding the instance was not justified. The instance was passing AWS status checks, Apache was returning `200 OK` locally, and Apache was listening on port 80. Rebuilding would have replaced a functioning workload instead of correcting the configuration that was actually preventing external access.

For a production workload, I would document the proposed security-group change and obtain approval from the system owner and appropriate change-management or security personnel before allowing public HTTP access.

One change I would not make to production without authorization is:

```bash
--protocol tcp --port 80 --cidr 0.0.0.0/0
```

This allows TCP port 80 from any IPv4 address on the internet. The risk is increased exposure to unwanted or malicious traffic. I would first confirm that public access was required, document the existing rule set, obtain approval, and have a rollback plan ready.

---

## Lessons Learned

The biggest lesson from this lab was that an EC2 instance being in the `Running` state does not prove that the application is reachable.

I learned to troubleshoot from multiple layers instead of immediately rebuilding a server. The AWS control plane showed the instance state and security-group configuration. The guest checks showed that Apache was actually running and listening on port 80. The external curl test showed whether the application was reachable through the network path.

I also learned that the smallest supported change should be preferred when the evidence clearly identifies the affected layer. In this case, adding the missing TCP port 80 security-group rule fixed the problem without changing the operating system or rebuilding the EC2 instance.

The lifecycle test also showed that a stop/start can change an auto-assigned public IPv4 address while the instance ID and private IP remain the same.

Finally, I learned the importance of cleanup. After the investigation was complete, I terminated the disposable instance, waited for the `terminated` state, deleted the security group, and verified that the group no longer existed.

---

## Professional Vocabulary

* **AWS control plane:** AWS-side information used to manage and inspect resources.
* **Workload path:** The actual path used by a client to reach the application.
* **Security group:** A virtual firewall controlling traffic to and from an EC2 instance.
* **Inbound rule:** A rule controlling traffic allowed to reach the instance.
* **TCP port 80:** The standard port used for HTTP traffic.
* **Root cause:** The specific condition responsible for the observed failure.
* **Corrective action:** A controlled change made to resolve the identified problem.
* **Verification:** Evidence collected after a change to prove the expected result occurred.
* **IMDSv2:** Instance Metadata Service version 2, which uses session-oriented requests.
* **Lifecycle:** The different states and transitions of an EC2 instance, such as running, stopped, and terminated.
* **Reachability:** Whether a client can successfully communicate with the workload.
* **Least change:** Correcting only the layer supported by the evidence instead of rebuilding or changing unrelated components.
* **Rollback:** Reversing a change if it causes an unexpected result.
* **Change control:** The process of reviewing and approving changes before applying them to a controlled environment.
* **Escalation:** Passing an unresolved or higher-risk issue to the appropriate technical or business owner.
* **Evidence-based troubleshooting:** Using observed command results and system behavior to identify the failure instead of guessing.

## GitHub Playbook Entry URL

Direct public file URL:

https://github.com/joshv1001/Harbor-Tech-Operations-Playbook/blob/main/week04-ec2.md

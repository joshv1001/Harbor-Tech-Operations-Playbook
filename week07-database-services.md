# Week 7: Managed Database Services

## HarborTech Ticket Summary

The Week 7 ticket focuses on helping a café decide how to manage its MySQL database. The scenario describes a database running on an EC2 virtual machine, with concerns about patching, backups, maintenance, availability, and recovery. HarborTech needs to evaluate whether keeping MySQL on EC2 is the best choice or whether a managed service such as Amazon RDS for MySQL would be a better option. Amazon Aurora and DynamoDB are also worth understanding, but the final choice must depend on the application's needs.

## Client Impact

If the database becomes unavailable or runs slowly, café employees may have trouble accessing business information and completing daily tasks. A poorly planned migration could cause downtime, data loss, application connection problems, or unexpected expenses. Choosing a database that does not fit the application's data structure and queries could also require additional development work. The business needs reliable backups and a recovery plan that can restore service within an acceptable amount of time.

## AWS Services Involved

- **Amazon EC2:** Provides virtual machines that can run an operating system and database software. When MySQL runs directly on EC2, the customer manages more of the operating system and database maintenance.
- **Amazon RDS for MySQL:** A managed relational database service that can reduce routine infrastructure and database administration work.
- **Amazon Aurora:** A managed relational database service compatible with MySQL and PostgreSQL. It may be useful if the workload needs its performance, availability, or scaling features and the cost is justified.
- **Amazon DynamoDB:** A managed NoSQL database that works well for supported key-value and document access patterns. It is not an automatic replacement for a relational MySQL database.
- **Multi-AZ:** Uses multiple Availability Zones in supported database configurations to improve availability and resilience.
- **AWS Database Migration Service (AWS DMS):** Helps migrate data between supported database sources and targets. The application and migrated data still need to be tested.
- **Schema conversion:** Helps assess or convert database structures and code when moving between database engines.
- **Security groups:** Control network traffic to AWS resources, including database connections.
- **Database connectivity:** Includes network access, routing, ports, authentication, encryption, and application connection settings.

## Virtualization Connection

When MySQL runs on EC2, the database uses a virtual machine, but the customer remains responsible for much of the operating system maintenance, database installation, patching, backups, and recovery process. A managed service such as RDS handles more of the underlying infrastructure and routine maintenance. This can reduce the team's workload, but it does not remove responsibility for data, access control, application compatibility, database settings, monitoring, performance, costs, and recovery requirements. The main difference is how operational responsibilities are divided between AWS and the customer.

## Evidence Reviewed

I reviewed the AWS Academy environment in the `us-east-1` region and the café database scenario provided in the assignment.

- The scenario describes structured MySQL tables, SQL queries, and transactions. These characteristics generally fit a relational database, but I did not independently inspect the café's actual schema or queries.
- The RDS console's **Databases** page showed zero databases and no database resources to display.
- I inspected the RDS create-database form to understand the available configuration choices. I did not create a database. The console also provided areas for snapshots, automated backups, subnet groups, and parameter groups.
- The DynamoDB console's **Tables** page showed an empty table list. I inspected the create-table form, including the table name, required partition key, optional sort key, and billing or capacity choices. I did not create a table.
- During a separate console investigation, the EC2 Instances page showed no instances listed. This was a separate console observation, not an EC2 command recorded in my original Module 6 activity report.
- I could not confirm the location, engine version, size, configuration, traffic, or dependencies of the café's current production database.
- I did not verify database connectivity, security group rules, application compatibility, backup restoration, recovery time objective (RTO), recovery point objective (RPO), or migration downtime.
- I did not measure actual service costs or performance. Those items need to be evaluated before a final decision.

## AWS Academy Activity: Actions and Results

### Action 1: Check RDS database instances

**Command used:**

```bash
aws rds describe-db-instances --region us-east-1 --query 'DBInstances[].{Name:DBInstanceIdentifier,Engine:Engine,Status:DBInstanceStatus}' --output table
```

**Actual result:** The command did not display any database instance rows.

**What the result demonstrates:** No RDS database instances were returned for the inspected account and `us-east-1` region. This does not prove the café's database does not exist in another region, account, or environment.

### Action 2: Check DynamoDB tables

**Command used:**

```bash
aws dynamodb list-tables --region us-east-1 --output table
```

**Actual result:** The output displayed the `ListTables` table heading but no table names.

**What the result demonstrates:** No DynamoDB table names were returned for the inspected environment and region. This does not establish whether DynamoDB is suitable for the application; that depends on the workload and its data access patterns.

### Action 3: Inspect the RDS console

**Console actions:** Opened Amazon RDS, selected **Databases**, and inspected the **Create database** form without submitting it.

**Actual result:** The Databases page showed zero databases and no resources to display. I also observed console areas for snapshots, automated backups, subnet groups, and parameter groups.

**What the result demonstrates:** The inspected environment did not show an existing RDS database. Reviewing the form helped me understand the configuration options, but I did not create or change any AWS resources.

### Additional Verification: EC2 and DynamoDB Console Pages

During a separate console check, the EC2 Instances page showed no instances listed, and the DynamoDB Tables page was empty. These were console observations separate from the RDS and DynamoDB commands recorded above. I did not create, modify, or migrate any resources during this investigation.

## Operational Analysis

The café scenario describes structured data, SQL queries, and transactions, so a relational database is the most logical starting point. Amazon RDS for MySQL is worth evaluating because it can reduce some of the routine maintenance required when MySQL is managed directly on EC2. However, my AWS Academy checks did not identify the café's actual database, so its current production setup remains unconfirmed.

DynamoDB should only be considered if the application can use a NoSQL data model and its access patterns fit DynamoDB's design. Replacing a relational database with DynamoDB could require major changes to the application's data model and code. Aurora could be considered if testing shows that its performance, availability, or scaling features are needed and the cost makes sense.

The empty resource lists only describe what I found in the inspected AWS environment. They do not prove that the café has no database. HarborTech must confirm the correct account and region, locate the existing database, review its engine version and dependencies, and document the business's availability and recovery requirements before deciding on a migration.

## Recommendation

I recommend keeping the current database in place while evaluating Amazon RDS for MySQL as the first managed-service option. Before recommending a migration, HarborTech should locate the existing database and document its MySQL version, schema, application dependencies, data size, performance needs, and connectivity requirements.

The team should compare the cost and maintenance effort of RDS with the current EC2 approach. Aurora should be considered only if the workload needs its additional capabilities and the cost is justified. DynamoDB should only be considered if the application and its data access patterns are a good fit for NoSQL.

Before any migration, the team should create and verify backups, test the target in a nonproduction environment, validate the data and application, agree on a maintenance window, and document a rollback plan. A supervisor and the client must approve the migration before production changes are made.

## Escalation Notes

As a junior intern, I should not create a production database, change production connectivity, alter security group rules, change permissions, approve additional costs, or migrate production data without authorization.

I would escalate the following items to the database or cloud lead:

- Confirming the correct AWS account, region, and location of the café's existing database.
- Confirming the MySQL version, schema, application dependencies, and compatibility with the proposed target.
- Approving the service choice, costs, network access, security controls, and availability configuration.
- Establishing backup retention, restoration testing, RPO, RTO, and monitoring requirements.
- Approving migration timing, downtime, data validation, cutover, and rollback procedures.

I did not make any production changes during this investigation. The unresolved questions should be escalated before implementation begins.

## Lessons Learned

This activity taught me that database services should be selected based on the workload instead of simply choosing the newest or most managed option. An application that uses relational tables, SQL, and transactions is usually a stronger starting fit for MySQL or another relational database than for a NoSQL service. RDS can reduce some maintenance responsibilities, but the customer still has to manage important decisions about data, security, application behavior, performance, costs, and recovery.

I also learned to record the actual commands and console results instead of assuming what the environment contains. An empty resource list does not prove that a client's database does not exist elsewhere. A safe migration requires discovery, compatibility checks, tested backups, validation, approval, and rollback planning.

## Professional Vocabulary

- **Relational Database:** A database that organizes information into related tables and commonly uses SQL to query and update data.
- **NoSQL:** A category of database systems that uses data models beyond the traditional relational-table approach. The model should match the application's access patterns.
- **Amazon RDS:** An AWS managed relational database service that handles some routine infrastructure and database tasks.
- **Amazon Aurora:** An AWS relational database service compatible with MySQL and PostgreSQL.
- **Amazon DynamoDB:** An AWS managed NoSQL database that supports key-value and document data models.
- **Multi-AZ:** A deployment approach that uses multiple Availability Zones to improve availability and resilience in supported configurations.
- **AWS Database Migration Service (AWS DMS):** An AWS service that helps migrate data between supported database sources and targets.
- **Schema Conversion:** The process of assessing or changing database structures and code so they work with a different database engine or target.
- **Managed Service:** A service where the provider operates some parts of the underlying technology, reducing but not eliminating the customer's responsibilities.
- **Workload Fit:** How well a service's data model, performance, availability, security, and cost match the application's actual needs.

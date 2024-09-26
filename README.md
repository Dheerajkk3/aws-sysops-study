# aws-sysops-study

## RDS
- RDS (Relational Database service) is a AWS service offering to create databases on cloud.

- It has 6 engine types.  

- It is manaaged DB service by AWS which means that users can't SSH to the DB servers.

- Users use SQL as querying language because the data is inserted in tables.

- The following are RDS DB Engines

  - Postgres

  - MySQL

  - MariaDB

  - Oracle

  - MS SQL Server

  - IBM DB2

  - Aurora

- Advantages of RDS over a user provisioned DB on ec2:
  - Automated provisioning , OS Patching
  - Automated backups and point in time restore of DB.
  - Monitoring Dashboards
  - Read replicas and multi AZ features
  - Maintenance windows for upgrades
  - supports horizantal and vertical scaling.
  - EBS backed storage and autosizing
- When does storage auto scaling triggers?
  - Storage is automatically updated when:
    - Free storage is less than 10% of allocated storage
    - low storage lasts for atleast 5 mins
    - 6 hours has passed since last modification
 
*Read Replicas*
- These are used for read scalability.
- You can  provision  a max of 15 read replicas for a  single RDS instance. They can be setup in same AZ, cross AZ, cross region.
- There wont be pay for data transfer if read replicas are setup in same AZ.
- Read Replicas that span across regions incurr charges.
- It does asynchronous replication from main db which results in reads from DB are eventual consistent.
- You can promote read replicas to serve as main db.
- Read replicas  accept only SELECT queries but not UPDATE, INSERT, ALTER, DELETE
- You can setup read replicas as MULTI AZ for disaster recovery purposes.

*Multi AZ*
- It is a standby DB created to perform synchronous replication in another AZ. The main goal of multi AZ is to increase availability.
- It will maintain the same DNS record.
- Synchronous replications means that as soon as a write operation happens to DB, it will replicate to that standby DB.
- In case of failure to the main  db the failover automatically happens to the multi AZ instance automatically.
- Conversion of single AZ to multi AZ will have zero downtown. It can be modified at any time.
- Internally during that conversion:
  - A snapshot of main db is created.
  - New DB in another AZ has been created from the snapshot.
  - Synchronization is enabled

    | Read Replicas | Multi AZ |
    |---------------| ---------|
    | Asynchronous replication | Synchronous Replication|
    | Performance Optimzation| Availability |
    | Only reads from DB | Stand by DB until failover|
    | It can be promoted to main DB | It can be failover to main DB during outages|

*Multi AZ fail over conditions:*
- When Primary DB instance
  - Failed
  - OS of the instance is patching
  - unreachable due to loss of network connectivity
  - Modified ( eg instance type has been changed)
  - Busy and unresponsive
  - underlying storage is failed
- If there is AZ outage
- Manual failover of primary DB instance intiated by Reboot with failure.

*RDS Proxy:*
- Why do we need RDS Proxy?
  - By default lamda functions are created in outside of our VPC, if lambda functions need to access RDS or elastic cache from private subnets then we need to create Lambda functions with VPC ID, Subnets and security groups. As a result lamda creates ENI in the same subnet. That way Lamda function will connect to RDS.
  - The above design leaves the problem of allowing too many open connections to RDS DB instance.
  - RDS Proxy takes care of cleaning up open connections and manages connection pools.
  - RDS Proxy can be deployed in same AZ or different AZ of primary DB instance.
  - If RDS Proxy is deployed in private subnet then we need lamda functions to be deployed in private subnet.
  - If lamda function makes multiple connections to RDS proxy which can in turn make only 1 connection to primary DB instance. This is called as connection pooling.
  - RDS Proxy supports IAM authentication, DB authentication, autoscaling.
*Parameter Groups*
- There are 2 Parameter groups: i. static ii. dynamic
- Dynamic parameter groups are applied immediately
- Static parameter groups are applied after instance reboot.
- Important parameter groups:
  - Postgre/SQLServer: rds.force_ssl=1 to force ssl connections
  - MySQL/MariaDB: require_secure_transport=1 to force ssl connections
   

  - Performance Insights is not supported for db.t2 type instance.


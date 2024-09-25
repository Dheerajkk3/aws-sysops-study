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


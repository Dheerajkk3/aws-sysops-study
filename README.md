# aws-sysops-study

## RDS
- RDS (Relational Database service) is a AWS service offering to create databases on cloud. 

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
 
*Read Replicas*
- These are used for read scalability.
- You can  provision  a max of 15 read replicas for a  single RDS instance. They can be setup in same AZ, cross AZ, cross region.
- There wont be pay for data transfer if read replicas are setup in same AZ.
- Read Replicas that span across regions incurr charges.
- It does asynchronous replication from main db which results in reads from DB are eventual consistent.
- You can promote read replicas to serve as main db.
- Read replicas  accept only SELECT queries but not UPDATE, INSERT, ALTER, DELETE

*Multi AZ*


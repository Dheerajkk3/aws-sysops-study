# aws-sysops-study

## RDS
- RDS (Relational Database service) is a AWS service offering to create databases on cloud.

- It has 6 engine types.  

- It is manaaged DB service by AWS which means that users can't SSH to the DB servers unless you use custom RDS.

- Users use SQL as querying language because the data is inserted in tables.

- The following are RDS DB Engines

  - Postgres (5432)

  - MySQL (3306)

  - MariaDB (3306)

  - Oracle (1521)

  - MS SQL Server

  - IBM DB2 (50000, 250000)

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
- Dynamic parameter groups are applied immediately hence no reboot required.
- Static parameter groups are applied after instance reboot.
- Important parameter groups:
  - Postgre/SQLServer: rds.force_ssl=1 to force ssl connections
  - MySQL/MariaDB: require_secure_transport=1 to force ssl connections
    
*Snapshots vs Backups*

| Backups | snapshots |
|---------|-----------|
| Backups are continuous and allow point in time recovery | It takes IO operations and can stop DB instance from seconds to minutes |
| It will happen only in maintenance windows | Snapshots taken on multi AZ doesn't impact doesn't impact master - just the standby |
| when you delete DB instance, you can retain automated backups | Snapshots are incremental after the first one (which is full)|
| Retention period is 0 to 35 days| You can copy , share snapshots for eg, manual snapshots can be shared with other accounts directly. Automated snapshots can't be shared directly rather it has to be copied first and shared with other accounts next |
| to disable backups, you set retention period to 0| You can take final snapshot before you delete DB instance and manual snapshots doesn't get expired |

RDS events keep track of events related:
- DB instances
- Parameter groups and Security Groups
- Snapshots
  - RDS events can trigger a SNS topic based on event.
  - RDS events can push events to Eventbridge
  - RDS logs can be pushed to cloud watch and you can 
  
*RDS Event Subscriptions*
- subscribe to events to get notified by using SNS topics.
- It needs event source (instances, SGs) and Event catageories (creation or failover).

*Performance Insights*
- Allows to visualize DB Performance and identify issues from troubleshooting.
- Visualize DB load and filter that load by using:
  - By  Waits: Identify which resource is under heavy load like CPU,
  - By SQL Statements: Identifies which SQL statement is causing load
  - By host : Identifies which host is causing the load on DB.
  - By Users: Identifies which user is causing the load or problem.
  - DB Load : It is the number of active connections to DB.
  - Performance Insights is not supported for db.t2 type instance.
 
## AURORA
- AWS owned DB which means there is no opensource.
- Both MySQL and Postgres DB are supported as Aurora DB.
- It is very cloud optimized and claims 5X performance improvement over MySQL and 3X over Postgres.
- Aurora storage automatically grows in increments of 10GB over 128TB
- 20% more cost than RDS
- Failover to Multi AZ is instantaneous (<30sec).
- Support 15 Read replicas and they can have autoscaling too.
- It always have 6 copies of data stored across 3 AZs.
- for write 4 out of 6 copies is sufficient which means even if an AZ is lost the writes do happen.
- for reads 3 out of 6 copies is sufficient.
- Master and 15 Read replicas are present.
- Cross Region replication is allowed.
- Writer Endpoint: DNS entry that always points to master
- Reader Endpoint: It helps with connection load balancing and connects to read replica. If a client connects to reader end point then it connects to one read replica.
- Back tracking: Restore data at any point of time without backups upto 72 hrs. Supported only in MySQL only. It wont create a new DB if back tracked.
- Automated backups: These are available with a retention of 1 - 35 days. Point in time restore to create a  new DB.
- Aurora DB cloning. It will create a new DB by copying the data from original EBS volume. It uses copy-on write protocol. You can create a test DB with production data.

## RDS and AURORA Security:
- Data at rest encryption is possible due to KMS keys (must be enabled during launch time).
- If master is unencrypted then read replicas are unencypted too.
- to encrypt and unencrypted master, we need to snapshot it and then create with encryption.
- With TLS in flight encryption is possible.

## AMAZON ELASTIC CACHE (REDIS and MEMCACHED)
- Caches are in memory databases that produces high performance and low latency.'
- Cache Hit: If an application query is found in Elastic cache, then it is treated as cache hit.
- Cache Miss: If an application query is not found in Elastic cache, then the query reads from RDS and writes it to Redis.
- It makes application stateless.

## HOW?
  Application writes session data to elastic cache, if a user hits/request another session, application will retrieve session from elastic cache.

  | REDIS (6379) | MEMCACHED(11211) |
  |-------|----------|
  | Multi AZ and Read Replicas are supported | Multi Node sharding|
  | Data Durability with AOF persistence | NO HA and NO Persistence|
  | Backup and restore | No Backup and restore|
  | Sets and Sorted Sets| Multi threaded architecture|

  *Cluster Modes:*
  
  Disabled:
  - one primary cache node, 0-5 read replicas. In case of failover one of these 5 can be primary cache node.
  - Asynchronous replication
  - primary node is used for both read and write, others are read-only.
  - one shard, all nodes have all data.
  - Multi AZ enabled by default.
    *Scaling*
    Horizantal: scale in/ out on  read replicas (0-5)
    vertical: scale in/ out to on node  groups results in DNS update.
    
  Enabled:
   - Data is partioned across shards to make writes friendly. Auto scaling is supported on these shards (increase/decrease shards)
   - Auto scaling is enabled in Redis when cluster mode is enabled only.
   - Supports upto 500 shards
     - 500 shards with single master
     - 250 shards with one master and one replica
     - 83 shards with one master and 5 replicas.
    
     Redis connection endpoint:
     - Standalone node will have only 1 endpoint for read/write
     - cluster mode disabled:
       - Primary endpoint : serves all write operations
       - Reader endpoint: reads are split evenly across all read replicas.
       - Node Endpoint: for all read
     - cluster mode enabled:
       - configuration endpoint: for all read / write endpoints.
  


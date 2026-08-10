1. 3 regions
2. VPCs 
    - Created in an AWS account, within a region
    - VPC CIDR: IP address range of VPC
    - default VPCs: 1 per region
        - limited for production deployments
        - default CIDR 172.31.0.0/16
    - /20 subnet in each Access Zone AZ in the region
3. EC2 features:
    - IAAS - provides VM instances
    - Private service-by-default - uses VVPS networking
    - AZ Resilient: instances fails if AZ fails
    - different instance sizes and capabilities
    - On-Demand billing - per second
    - Local on-host storage or Elastic Block Storage
4. Instance lifecycle:
    - has CPU, memory, disk, and networking
    - Running
        - can be stopped or terminated
        - all 4 charged
    - Stopped
        - can be run again or terminated
        - only disk charged
    - Terminated
        - irreversible, no charges
5. AMI - Amazon Machine Image - server image to create VMs
    - Permissions: 
        - Public - All allowed
        - Owner - Implicit
        - Explicit - Specific
    - Root Volume - Boot volume (C:/) 
    - Block Device Mapping - config that links volumes boot to data
        - device ID
6. Connecting to EC2 - connecting through SSH or RDP


Creating an Instance
1. Create Keypair
2. Launch Instance
3. Wait for initialization
4. Right click > Connect
5. Right click Terminate (also remove security group if created)

S3 Basics
- Default services
- Global storage Platform - regionally based and resilient
- Public Service, unlimited data and multi-user
- movies, audio datasets
- economical & accessed via UI/CLI/API/HTTP
- Objects and Buckets

Buckets
- Objects have keys and keys access bucket
- can be 0 bytes to 5TB
- have version ID, metadata, Access control, and subresources
- data sits in bucket in a region and never leaves
- bucket name must be globally unique, among all aws accounts
- folders are viewable if filenames have prefixes, but buckets are not necessarily hierarchical

Bucket names are:
- globally unique
- 3 to 63 chars, all lowercase
- start with lowercase letter or number
- cant be IP formatted e.g. 1.1.1.1
- you can only have up to... soft limit: 100 buckets hard limit: 1000, per account

S3 Patterns...
- is an object storage system, not file or block
- cant mount it 
- great for large-scale data storage, distribution or upload
- great for 'offload'
- INPUT and/or OUTPUT to many AWS products

CloudFormation Basics
- IAC Product allows automation infrastructure creation, update and deletion
- uses YAML or JSON
- First field is TemplateFormatVersion
- Second MUST be Description
- Other things that are defined are Metadata, Parameters, Mappings, Conditions, Transform, Resources, Outputs...

- Cloudformation starts with a Template
- A Template contains resources
- A Logical resource is an instance with properties for configuration
- CloudFormation creates one or more Stacks
- For logical resources in a stack, CloudFormation creates a Physical Resource
- Stacks interact with resources in an AWS account

In Practice:
1. Create an instance by going to CloufFormation > Create Instance
    - optionally upload a template
2. Wait for instance to create
3. Access instance by going to EC2 and right-clicking instance > Connect
    - There is an option to use SSH or SSM, SSM may be more convenient, just type bash
4. Delete Instance by right clicking > Delete
5. Confirm deletion by going to CloudFormation > Stacks

Cloudwatch
- Collects and manages operational data
- Metrics: AWS Products, Apps, on-premises
    - Time ordered data using timestamps and values
    - Dimensions: name value pairs for separating data scopes
    - Some metrics are gathered Natively by a product, e.g. CPU utilization
    - Others need a CloudWatch Agent e.g. Process memory utilizations
- Logs: e.g. Linux server logs
- Events: AWS Services & Schedules
- Alarms: linked to specific metric, detect errors, can trigger action 
    - helpful for auto-scaling
- Statistics: from Console or API
- Namespace - container for monitoring data.
    - All AWS data goes into a namespace 'AWS/service' > AWS/EC2

In Practice:
1. Launch an Instance
2. In Advanced Details select 'Detailed CloudWatch Monitoring'
3. Wait for Launch
4. Go to Cloudwatch in search
5. Click 'Alarms' > 'All alarms'
6. Create Alarm > Select Metric
7. Click EC2 > Per Instance Metrics
8. Find Instance (Refer to Instance Monitor) > Select CPUUtilization
9. 'Select Metric', and Conditions > Static > 'Greater Eqaul > define threshold '15' (percent)
10. 'Next', decide on a notification, if any (mainly used for production)
11. Select as alarm, give name e.g. "High CPU"
12. Alarm State = Insignificant Data means not enough data to trigger
13. Go back to Instance and Connect "EC2 Instance" > Connect
14. Install 'Stress App' (for testing purposes)
    - sudo yum install stress -y
15. Run stress $stress
16. stress -c 1 -t 3600
17. go back to Cloudwatch console, click on Alarm to monitor
 
High Availability (HA)
- Aims to ensure an agreed level of operational performance, uptime per year:
    - 99.9% = 8.77 hours
    - 99.999% = 5.26 minutes
- Can use Redundant Servers
- Minimizing Outages

Fault Tolerance (FT)
- Property that enables a system to continue to operate properly in the event of the failure of on or some its component
- Designed to operate through failure without disruption
- Levels of redundancy and session routing around failed components
- Harder to design, more important than HA

Disaster Recovery (DR)
- Set of policies, tools, and procedures to enable the recovery or continuation of vital technology infrastructure and systems following a natural or human-induced disaster
- Make sure staff has availability, and knowledge of how to access.
- use periodic testing

Route53
- DNS as a Service
- Global Service with a single database
- Globally Resilient
- Two main services:
    1. Register Domains
    2. Host Zones... managed nameservers
Registries manage top-level domains (.com .io .net etc.)
A Zone file is a database with registry info
Route53 puts a zone file into 4 managed nameservers, which is a Hosted Zone
    - Can be Private (linked to VPCs)
NS records delegate ownership into the nameservers (hosted zone)

DNS Records:
- A: maps www to IPv4
- AAAA: maps www to IPv6
- CNAME: Canonical name, an alias
- MX: mail exchange
    - has priority and value. e.g. MX 10 mail
- TXT: prove domain ownership, block spam
TTL: Time-To_Live, numeric value in seconds
Client queries Resolver, which queries Root, top level, then domain. The domain sends an authoritative answer if no provider changed. TTL sets frequency of cache on non-authoritative DNS server.
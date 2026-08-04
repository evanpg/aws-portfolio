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

S3 Patterns
- is an object storage system, not file or block
- cant mount it 
- great for large-scale data storage, distribution or upload
- great for 'offload'
- INPUT and/or OUTPUT to many AWS products


Virtualization
- Operating System kernel runs in Privilege Mode which interacts with underlying hardware(CPU, MEM, NETWORK, DEVICES)
    - Can crash entire network
- Running Virtualization gives each container App its own OS. (APP + OS = VM)
    - Any kernel call goes to Hypervisor (which is the Host OS) via:
        - Binary: Obsolete
        - HyperCall: Better, but Obsolete 
        - Hardware Assisted Virtualization: CPU contains capabilities/knowledge of Virtualization
        - Single Root I/O Virtualization (SRIOV): Allows Network Card: Bypasses Hypervisor for low-level interaction
            - In EC2, this is called 'Enhanced Networking'

EC2 Architecture
- EC2 Instances are virtual machines (OS + Resources)
- EC2 Instances run on EC2 Hosts
- Shared Hosts or Dedicated Hosts
- Hosts run in a single AZ. If AZ fails, Host and instance will fail
- EC2 Host sits outside of subnets, and have: 
    - Local Storage
    - Storage Networking 
    - Data Networking
- When Instances are provisioned in a Subnet in a VPC, a Primary Elastic Interface is mapped to the physical hardware on the EC2 Host
- Instances can have multiple Network Interfaces, even on different subnets ONLY within its AZ
- Elastic Block Store (EBS): let you allocate volumes of persistent storage. Also within only one AZ.
- Volumes: portions of persistent storage allocated to Instances in same AZ
- What is EC2 Good For?
    - Traditional OS+APP compute needs
    - Long-Running compute needs
    - Server style APPS
    - For burst requirements or steady-state load
    - Monolithic APP stack
    - Migrated APP workloads or Disaster Recovery

EC2 Instance Types
- Difference between types: 
    - RAW resources
    - Resource Ratios
    - Storage and Data Network Bandwidth
    - System Architecture, and Vendor (x32, x86)
    - Additional features and capabilities, customizations
- 5 Main categories:
    1. General Purpose: a good starting point
        - Default
    2. Compute Optimized: More CPU offered than memory
    3. Memory Optimized: for in-memory databases
    4. Accelerated Computing - Hardware GPU, field programmable geate arrays (FPGAs)
    5. Storage Optimized
- Instance Type Names: e.g. 'r5dn.8xlarge'
    - R: Instance Family
    - 5: Instance generation
    - 'dn': These are Additional Capabilities (not always there) 
    - 8xlarge: Instance size
- More info:   
    - https://aws.amazon.com/ec2/instance-types/
    - https://ec2instances.info/

In Practice:
EC2 SSH vs. IC2 Instance Connect
- An Amazon Machine Image (AMI) is a template that provides the software, operating system, and configuration settings needed to boot an Amazon EC2 virtual server instance. You must select an AMI every time you launch a new EC2 instance.
- EC2 SSH: 
    1. Go to EC2 > Key Pairs > Create Key pair > Name 'A4L.pem' > Create > Download Key
    2. 1-Click Deployment > Key = 'A4L.pem'
    3. Go to EC2 Instance > Select > Review Security
    4. Connect to Instance
    5. Paste connection command into local terminal
- IC2 Instance Connect
    1. Should connect right away
    2. Go to Instance > Inbound Rules > Edit
    3. Locate inbound IPv4 for SSH
        - Remove 0.0.0.0/0 (allowing all is not good for production usage)
        - Change Source to MyIP
    4. Go back to Instance Connect
        - Won't load now because the connect service does not originate from your IP
    5. Go back to Inbound Rules and Create a rule for Amazon IP (in 'IP ranges.json', attached)
        - now you can connect

Storage Generalizations
- Direct Attached Storage: Storage on the EC2, called Instance Store
- Network Attached Storage (NAC): Volumes delivered over the network, by Elastic Block Storage (EBS)
    - OS boot image in EC2 is stored here
- Ephemeral Storage: Temporary storage
- Persistent Storage: Lives on past the lifetime of the instance
- Block Storage: Volume presented to the OS as a collection of blocks, no structure provided. 
    - Mountable and bootable
    - Can be SSD or HDD, or Logical
- File Storage - Presented as a file share
    - Mountable but NOT bootable
- Object Storage: Metadata returns an Object, from flat storage in a container
    - NOT mountable and NOT bootable
- Storage Performance:
    - IO: Block size, e.g. 16KB...
    - IOPS: Read and Write speeds, e.g. 100IOPS
    - Throughput: Amount of data, e.g. 1.6MB/s
    - IO * IOPS = Throughput, although other specs can cap

Elastic Block Storage (EBS)
- Block Storage: Raw disk allocations (volume) that can be encrypted using KMS
- Instances see block device and create file system on the device (ext3/4, xfs)
- Storage is provisioned in one AZ (resilient in that AZ)
- Attached to one EC2 instance (or other service) over a storage network
- Not linked to just one instance
- Snapshot: A backup into S3. Create volume from snapshot (migrate between AZs)
- Different physical storage types, sizes, and performance profiles
- Billed on GB/Month, (and in some case performance)
- EBS can point from one instance and then to another
- S3 Snapshots can replicate EBS data to make available  across AZs and Regions

EBS General Purpose SSD - GP2
- When you create a Volume:
    - Can be 1GB to 16TB
    - You get an IO Credit = 16KB
        - IOPS assume 16Kb
            - 1 IOPS is 1 IO in 1 second (1 16KB chunk / second)
        - Billing based on usage
    - IO Bucket: fills at baseline performance rate of Volume: 
        - e.g. 100 IO credits per second
        - Bursts are handled up to 3000 IOPS, depletes bucket
    - For volumes larger than 1TB (such as GP2), IO credits system is not used
    - GP3 Standard gets 3000 IOPS and 125Mb/s (20% cheaper than GP2)
        - Pay extra for up to 16000 IOPS and 1000MB/s (still sometimes cheaper than GP2)

EBS Volume Types: IOPS SSD (io1/2)
- For high-performance, latency sensitive workloads, intensive NoSQL

EBS HHD-based
1. st1: Throughput Optimized
    - Cheap
    - Designed for data that is sequentially accessed
    - 125GB to 16TB
    - Max 500 IOPs at 500MB/s
    - Baseline performance of 40MB/s/TB 
    - Burst performance 250/s/TB
2. sc1: Cold HDD
    - Cheaper
    - For infrequent workloads
    - Also 135GB to 16TB
    - Max 250 IOPS at 250MB/s
    - Base 12MB/s/TB
    - Burst 80MB/s/TB

Instance Store Volumes
- For temporary data
- Local Block Storage Devices
- Physically connected to one EC2 Host
- Highest storage performance in AWS
- Included in instance price
- Attached at Instance Launch
- Instance stores data in Ephemeral Storage
    - If Instance migrates to another EC2 host, the data does not transfer
- You pay for ISV already - included at instance price

Choosing between Instance Store and EBS
- EBS is for Persistent Storage, Resilience, and Isolation from Instance Lifecycle
- Instance Store for super high performance, low (no) cost
- SC1 also cheap
- ST1 is cheap and used for throughput or streaming
- EC2 SC1, ST1, CANNOT use to Boot
- GP2/3 is up to 16,000 IOPS
- IO1/2 is up to 64,000 IOPS
- IO2 Block Express up to 256,000 IOPS
- RAID0 + EBS up to 260,000 IOPS (io1/2-BE/GP2/3)
    - Need large instance and large EBS Volume
- Instance Store is > 260,000 IOPS

EBS Snapshots
- Efficient way to back up EBS Volumes to S3
- Snapshots are incremental Volume copies to S3
    - The first snapshot is a full copy of 'data' on the Volume
    - Future snapshots are incremental
- Volumes can be created (restored) from snapshots
    - To move between AZs or Regions
- Performance:
    - New EBS Volume w/o snapshot has full immediate performance
    - Snapshots restore lazily, fetched gradually
    - Requested block are fetched immediately
    - Force a read of data immediately
    - Fast Snapshot Restore (FSR): Immediate restore
        - Up to 50 per Region
        - Set on the Snap & AZ 
        - Encryption also affects Snapshots
- Cost is per month and only on allocated data, not full volume
    - Because snapshots are incremental, it doesn't cost more to take 5 minute snapshots vs. per hour
    
What did I learn?
- EBS volumes are created within one AZ
- can be mounted only to instances in that AZ
- Can move between instances in that AZ
- can be moved between instances retaining data
- you can take a snapshot to create volumes in different AZs
- Snapshot can be copied to other AZ, as data recovery 
- EBS is persistent
- Instance Store Volumes only are persistent while connected to same EC2 host
    - Can reboot instance, but if you stop and start, a new IP is assigned and all data is lost

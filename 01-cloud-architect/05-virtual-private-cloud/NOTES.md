VPC Sizing and Structure
- VPCs run on subnets
- Architecture Design Considerations:
    - Decide on IP plan
        - What size should it be?
        - Are there Networks we can't use?
        - Ranges which other parties use, Partners, vendors, other VPCs
        - Try to predict the future
    - Tiers - WEB, APP, DB, Spare
    - Resiliency - Multiple AZs
    - Each Tier has subnet in each AZ

In Practice
Global Architecture:
- Offices
    1. London
    2. NY
    3. Seattle
    4. Sydney (Head Office)
- IPs used (must be avoided)
    - 192.168.10.0/24   On-premise
    - 10.0.0.0/16       AWS Pilot
    - 172.31.0.0/16     Azure Pilot
    - 192.168.15.0/24   London
    - 192.168.20.0/24   NY
    - 192.168.25.0/24   Seattle
    - 10.128.0.0/9      Google (From previous dev/admin)
        - 10.128.0.0 => 10.255.255.255
- More Considerations
    - VPC minimum /28 (16 IPs), maximum /16 (65536 IPs)
    - Personal Preference for the 10.x.x.x range
    - Avoid common ranges = avoid future issues
    - Reserve 2+ networks per region being used per account, for growth
- Regions our example business has
    - 3 in US, 1 in Europe, 1 in Australia
        - So, 10 Networks
    - Assume 4 Accounts
        - 10 x 4 = 40 Ranges (ideally)
- Potential IP topology 
    - Don't use 10.0.x.x to 10.10.x.x they are too common
    - 10.16.x.x  to  10.127.x.x.x  should be enough
    - Each Region gets four /16 subnets
        - 4 * 4 = 16 total subnets
        - /16 spilt into 16 subnets = /20 per subnet (4091 IPs) 

Custom VPCs
- Created in a region with all AZs in the region
- Isolated network, no in or out without explicit config
- Flexible config - Simple or Multi-Tier
- Hybrid Networking - other cloud & on-premises
- Tenancy: whether or not it goes on shared hardware or dedicated hardware
    - Default: Can decide per Object
    - Dedicated: If selected a VPC level, all MUST BE dedicated
- IPv4 Private CIDR Blocks & Public IPs
    - 1 Primary Private IPv4 CIDR Block
        - Min = /28(16 IPs)
        - Max = /16(65536 IPs)
    - Optional Secondary IPv4 CIDR Block
    - Optional single assigned IPv6 /56 CIDR Block
- Fully-featured DNS via R53
- VPC "Base + 2" Address 10.0.0.2
- Settings:
    - 'enableDnsHostnames' - gives instances DNS Names
    - 'enableDnsSupport' - enables DNS resolution in VPC

In Practice
1. Go to VPCs > Create VPC
    - 'VPC and more' gives a wizard
2. Select 'VPC only' for now and name it 'a4l-vpc1'
3. IPv4 CIDR = 10.16.0.0/16
4. 'Amazon-provided IPv6 CIDR Block'
5. 'Create'
6. Select VPC > Actions > Edit VPC Settings > Enable DNS hostnames and resolution
7. Save


VPC Subnets
- What services run from in VPCs
- A subnetwork within an AZ
- A subnet can never be in more than 1 AZ
    - An AZ can have many subnets
- IPv4 CIDR is a subset of the VPC CIDR
    - Cannot overlap with other subnets
    - Optional IPv6 CIDR (/64 subset of the /56 VPC - space for 256)
- Subnets can communicate with other subnets within a VPC
- Reserved IP Addresses
    - Network Address 10.16.16.0
    - Network +1 Address (VPC Router) 10.16.16.1
    - Network +2 Address (DNS Reserved) 10.16.16.2
    - Network +3 Address (For future requirements) 10.16.16.3
    - Network Broadcast Address (last) 10.16.16.255
- DHCP options
    - Control DNS, NetBios 
- Auto Assign Public IPv4, IPv6

In Practice
- Have a spreadsheet first of names and ranges
1. Go to created VPC > Add Subnet > Select the VPC ID
2. Name it and select AZ
3. IPv4 VPC CIDR block 10.16.0.0/16 
    - Never changes, even between regions
4. Paste in IPv4 subnet CIDR Block 
    - Changes with each subnet and region
5. Click IPV6 VPC CIDR Block dropdown and select
6. For 'IPv4 subnet CIDR block'
    - Click on down arrow until /64 (we dont need so much space)
    - For the first leave as is, but for each other subnet added click Right to give unique IP range to each subnet
7. Repeat 2-6, for each (4) Subnet in that region
8. Start again for next region
    - Remember not to overlap 'IPv4 subnet CIDR block' or 'IPv4 Subnet CIDR Block' 
9. Now go to VPC > Subnets > and for each Subnet instance:
    - 'Enable auto-assign IPv6'

VPC Routers
- Highly available device present in VPC, moves traffic to and from subnets
    - Network +1 Address
    - Controlled by 'route tables', each subnet has one only
        - but a route table can be associated with many subnets
    - VPC has a Main route table - subnet default
        - Destination: Higher CIDR = more specific = higher priority
        - Target: 
        - Status: 
        - Propagated: 

Internet Gateway (IGW)
- Region resilient gateway attached to a VPC
- VPC can have 0 or 1 IGW
- HA as it's attached to VPC
- IGW can be created and not attached to a VPC
- Runs within the AWS Public Zone
- Gateways traffic between the VPC and the Internet or AWS Public Zone (S3, SQS, SNS, etc...)
- Managed - AWS handles performance

In Practice
Creating a Public Subnet:
1. Create IGW
2. Attach IGW to VPC
3. Create custom RT
4. Associate RT
5. Default Routes targeted to => IGW
6. Subnet allocates IPv4

Bastion Hosts and Jumpboxes
- An instance in a public subnet
- Incoming management connections arrive there
    - Then access internal VPC resources
- Often the only way IN to a VPC

In Practice
Add and Attach IGW:
1. Go to VPC > Internet gateways
    - there is one by default when VPC was created
2. Create Internet Gateway 'a4l-vpc2-igw'
    - initially not attached to VPC
3. Go into instance > Actions > Attach to VPC > Select 'a4l-vpc2-igw'
Create Route Table for internet-accessing subnets:
1. Go to VPC > Route Table 
2. Create Route Table, select VPC, name it 'a4l-vpc1-rt-web'
3. Check created route table > Subnet Associations > Edit Associations
4. Select subnets to associate with 'sn-web-a', 'sn-web-b', 'sn-web-c'
5. Save
- These subnets are no longer associate s with the main VPC routing table
6. Go to RT's routes > Edit Routes
7. Add Route where:
    - Destination = '0.0.0.0/0'
        - Anything which is IPv4 not destined for VPC will use this default route
    - Target = Internet Gateway, select 'igw-a4l' created previously
8. Add another Route:
    - Destination = '::/0'
        - Default Route for all IPv6 addresses
    - Target = Internet Gateway, 'igw-a4l'
- Now we need to make sure that resources on web subnets get allocated with public IPv4 addresses
9. Subnets > Select 'web-a' Subnet > Edit
10. Enable auto-assign public IPv4 address
- Test Configuration
1. Go to EC2 > Launch Instance > name = 'a4l-bastion'
2. Use or Create Key pair
3. Network settings > edit 
    - VPC: select 'a4l-vpc-1'
    - Subnet: 'web-a'
4. New Security Group 'A4L-BASTION-SF'
5. Launch
6. Right click > Connect > Web Console or SSH

Stateful vs Stateless Firewalls
- TCP: connection using random port on client (ephemeral) and know port on server (commonly HTTPS 443)
- Stateless: Doesn't understand the state of connection
    - 2 Rules for Requests incoming: 1 IN, 1 OUT
    - 2 Rules for Requests outgoing: 1 OUT, 1 IN
    - Need full ephemeral port range
- Stateful: Intelligent enough to identify the REQUEST and RESPONSE components of a connection
    - Allowing the REQUEST(IN or OUT), means the RESPONSE( IN or OUT) is automatically allowed
    - Less admin overhead
    - Don't need full ephemeral port range

Network Access Control List (NACL)
- Every subnet has an associated NACL, which filters traffic
    - Connections within subnet not affected
- Stateless
- Cannot be assigned to logical resources, only subnets
- Use together with Security Groups to add explicit DENY (bad IPs/Nets)
- A NACL can be associated with many subnets
- Rules match the DST IP /Range, DST Port and Protocol and ALLOW or DENY based on that match
- Rules are processed in order, lowest rule number furst.
- '*' is an implicit DENY, if nothing else matches
- These Rule-pairs (app & ephemeral ports) are needed on each NACL for each communication type occurring:
    1. Within a VPC
    2. To a VPC
    3. From a VPC
- A VPC is created with a default NACL
    - with implicit DENY, ALLOW ALL
        - in effect it does nothing
- Custom NACLs can be created for a VPC and are initially associated with no subnets:
    - By default all traffic is denied because they have:
        - 1 INBOUND rule: implicit DENY
        - 1 OUTBOUND rule: explicit DENY

Network Address Translation (NAT)
- Giving a private resource instances access to the internet
- A set of processes remapping SCR or DST IPs
- Static Nat
    - Resource given Public IPv4 and sent through IG
- IP Masquerading: hiding CIDR Blocks behind one IP
- Public IPv4 addresses running out
- Gives private CIDR range outgoing internet access
- Runs from public subnet
- Elastic IPs (static IPv4 Public addresses) used by NAT
- For region resilience, NATGW in each AZ
    - and RT for each AZ pointed to that NATGW
- Managed, scales to 45Gb/s, data and duration upgrades for $
- Architecture:
    - NAT Gateway in Web Subnet, which is able to access Internet Gateway (Its Default Gateway is IGW)
    - APP subnets sent to NAT Gateway inside WEB Subnet
    - NAT Gateway records SRC, DEST, etc in a Translation Table
    - The IG knows it got packet from NAT
        - Modifies source address to the NAT Gateway address
- Instance vs Gateway
    - Instance: EC2 instance that receives data where source address is other resources, and destination will be host on internet
        - It will drops any data on its Network Card when that Network Card is not Source or Destination 
        - For some traffic, it will neither be source nor destination and will be dropped.
        - Must disable 'Sources/Destination Checks' to carry that traffic
            - Required if you want to use an EC2 instance as a NAT instance (although not preferred over NATGW)
![alt text](image.png)
- For IPv6 NAT is not required
- All IPv6 addresses in AWS are publicly routable
- the IGW works with all IPv6 IPs directly
- NATGWs don't work with IPv6
- ::/0 Route + IGW for bi-directional connectivity
- ::/0 Route + Egress-Only IGW = outbound only

In Practice
1. Connect to a EC2 instance using session manager 
    - you can't ping 1.1.1.1
2. 3x Go to VPC > NATGW > Create 
    - Name: 'a4l-vpc1-natgw-A'
    - Subnet: 'sn-web-A'
    - Assign Elastic IP
3. 3x VPC > Route Tables > Create
    - Name: 'a4l-vpc11-rt-privateA'
    - VPC: 'a4l-vpc1'
4. 3x Select each RT > Routes Tab
    - Add Route: 
        - Destination: '0.0.0.0/0'
        - Target: 'a4l-vpc-natgw-A'
5. Test ping, still doesn't work because it doesn't have explicit RT association
6. x3 Select private RT > Subnet Associations tab > Edit
    - Select all private subnet associations
        - 'sn-reserved-A'
        - 'sn-db-A'
        - 'sn-app-A'    
        - Save Associations
7. Now ping works. Associated RT > NATGW > IGW
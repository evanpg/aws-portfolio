S3 Security (Resource Policies & ACLs)
- S3 is private by default, only account root creator can access by default
- Bucket Policy: Like ID Policy, but attached to bucket
    - A form of Resource Policy
    - Resource perspective permissions
    - Resource Policies Allow / Deny
        - Same or different accounts
        - Anonymous Principals, access...
            - "Principal": "*"  << this field defines a Resource Policy>>
    - Identity Policies only work within that account

Access Control Lists (ACLs)
- AWS prefers you use bucket policies because they're inflexible 
- ACLs on objects and buckets
- A legacy subresource
- 5 conditions of ACL
    - READ: list objects in bucket
    - WRITE: create overwrite, delete objects in bucket
    - READ_ACP: read bucket ACL
    - WRITE_ACP: write bucket ACL
    - FULL_CONTROL: allow all ^^

Block Public Access
- Failsafe to avoid data leaks
- Applies to Anonymous and Public actors

When to Use:
- Identity:
    - Controlling different Resources
    - Have a preference for IAM
    - Same Account
- Bucket
    - Just controlling S3
    - Anonymous or Cross-Account
- ACLs
    - NEVER - unless you must

Static Website Hosting
- Normal Access is via AWS APIs
    - GET object call
- This feature allows access via HTTP, e.g. blogs
- Index and Error documents are set
- Website Endpoint: a specific address where site can be accessed
- Custom Domain via R53 - **Bucket Name matters**
- Offloading: 
    - Moving large data to non-compute
    - Dynamic website runs on compute service(EC2), which is more expensive
    - Needs Access to database, but media is static
    - Move media that compute service hosts to static bucket host, which is cheaper
- Out-of-Band Pages:
    - Change DNS to backup/status static website hosted on S3 
    - Maintenance use case

S3 Pricing (1st Tier)
- Storage: $.023 per GB
- Data Transfer Fee:
    - Transfer IN: $0
    - Transfer OUT: $.09 
- Data Requesting (APIs):
    - $.005 / 1000 Requests

In Practice
1. Create a bucket, giving the name of the static domain you are creating
2. Deselect Block Public
3. Create bucket and go into its properties > Enable Static Website Hosting
4. Specify index.html and error.html
5. Save Changes
5. Scroll all the way down to find URL
6. Upload index.html and error.html to Bucket
7. Upload image folder
8.  Go to site now, and it will be '403 Forbidden', must grant Permissions = a Bucket Policy
    - "Resource":["arn:aws:s3:::top10-animals4life-panda.com/*"]
9. Bucket is now visible to Anonymous Public
- To add custom domain name:
    1. Go to R53 > Hosted Zone > Create Hosted Zone
    2. Go to hosted zone > Create Simple Record > Name it
    3. Select 'Alias' > 'Alias to S3 Endpoint' > Choose Region
    4. Bucket Name must be same as Fully Qualified Domain Name
    5. You can now route to alias

Object Versioning
- Lets you store multiple version of an object within a bucket
    - Operations that modify an object create a new version
- Versioning in a Bucket starts off Disabled, can be Enabled, then Suspended, and ReEnabled, but cannot be reverted to Disabled
1. Take an object - KEY: cat.jpg  ID: null
2. If Versioning Enabled - KEY: cat.jpg  ID: 111111
3. Add another, same key - KEY: cat.jpg  ID: 222222
- Latest version (Current version) always returned
- A Delete Marker hides all previous version of Object, not actually deleting
- Billing occurs over all versions
- Only way to Stop Versioning is to delete and recreate Bucket

MFA Delete
- Enabled in versioning configuration
- MFA is required to change bucket version state, or delete versions
- Serial number (MFA) concatenated with Code passed with API calls is required

In Practice:
1. Create Bucket, Remove Public Block, and Enable Versioning
2. Go to bucket Properties > Enable Static Website Hosting
3. Add file "winkie.jpg"
4. Go to Bucket Policies > "Resource":["arn:aws:s3:::acbucket098765431/*"]
5. View image at index.html
6. Add another version of file, with exact same name
7. View page again and you just see most recent version
8. Viewing image history in Bucket with "Show Versioning" enabled, you can see the Delete Marker.
9. If you Delete the Delete Marker, you can then delete the newer version, restoring the old

S3 Performance
- By default when you upload data, it is a single Stream
- Single PUT upload:
    - Can be problematic because if Stream fails, everything fails
    - Time, Speed, and Reliability not ideal
        - 5GB max, but not recommended
- Multipart Upload
    - Data is broken up into individual parts
    - Min data size: 100MB
    - 10,000 max parts, 5MB => 5GB
    - Last part can be smaller than 5MB
    - Parts can fail, and be restarted
    - Transfer Rate = speed of all parts (Internet Bandwidth)

S3 Accelerated Transfer
- Needs to be enabled in a bucket
    - Bucket Name cannot contain periods
    - Needs to be DNS compatible in its naming
- Uses AWS edge locations as first hop, arrives quicker to bucket destination

In Practice:
1. Create Bucket, name cannot have periods
2. Bucket Properties > Enable Transfer Acceleration
3. Can compare here:  http://s3-accelerate-speedtest.s3-accelerate.amazonaws.com/en/accelerate-speed-comparsion.html

Key Management Services (KMS)
- Regional & Public Service
    - Can use aliases, per region
- Are isolated to a region
- AWS owned or Customer owned
- AWS Managed or Customer Managed, Customer managed are more configurable
- Create, Store, and Manage symmetric and asymmetric cryptographic keys
- Rotation is set for once a year
    - Backing Key, for decrypting files with old keys
- Can also do cryptographic operations, encrypt, decrypt4
- Keys never leave KMS - provides FIPS 140-2(L2) compliance
- KMS Keys (CMKs): Container for Keys
    - include ID, Date, Policy, Description, and State
    - Backed by physical key material, which are Generated or Imported
    - KMS Keys can  be used for up to 4kb of data
- Role Separation: Some users can Encrypt but not Decrypt. or Manage KMS but not Decrypt. etc.
- Data Encryption Keys (DEK): works on > 4kb
    - KMS provides 2 versions of DEK when created:
        1. Plaintext Version: To be used immediately for cryptographic operations (encryption)
            - Discarded after use
        2. Cyphertext Version: Encrypted version of the same DEK using the KMS Key that created it (for decryption)
            - Encrypted key stored with data
            - Can be discarded after use
- Permissions on Keys:
    - Key Policy: type of Resource Policy
        - Every KMS Key has one
    - Key Policies + IAM
    - Key Policies + Grants

In Practice:
1. Go to KMS and create Symmetric Key
2. Create Alias, and select User Permissions
3. Finish
4. Go to CouldConsole:
```bash
echo "find all the doggos, distract them with the yumz" > battleplan.txt
aws kms encrypt \
    --key-id alias/CatRobot \
    --plaintext fileb://battleplan.txt \
    --output text \
    --query CiphertextBlob \
    | base64 --decode > not_battleplans.enc 
    
aws kms decrypt \
    --ciphertext-blob fileb://not_battleplans.enc \
    --output text \
    --query Plaintext | base64 --decode > decryptedplans.txt
```
5. To delete, Go to KMS > Schedule Deletion

S3 Encryption - Server-Side Encryption (SSE)
- Buckets aren't encrypted, Objects are
- Encryption mandatory on Buckets
- S3 is encryption at rest. Data is already encrypted in transit to Bucket
- 2 types, can be used together:
    1. Client-side Encryption:
        - Data is all scrambled from User/App to S3 Endpoint to S3 Storage, S3 never sees it
    2. Server-Side Encryption (SSE):
        - Only encrypted from Bucket Endpoint to S3 Storage
        - 3 Types of SSE:
            1. Server-Side Encryption with Customer-Provided Keys (SSE-C)
                - Customer manages keys, and S3 managed Encryption. Uses HTTPS
                - For Regulation-heavy environments, AWS can't see
            2. Server-Side Encryption with Amazon S3-Managed (SSE-S3)
                - AWS handles keygen, encryption, and management
                - Encrypts per-Object Key, AES-256
                - Not suitable if you need Role-Separation (e.g. Financial and Medical), admin can view all data
            3. Server-Side Encryption with KMS KEYS Stored in KMS Key Management (SSE-KMS)
                - Created and Managed by you, Isolated Permission Configurable
                - Plaintext and Cyphertext Data Encryption Key (DEK)
                - KMS Gives keys to S3
                - S3 Admin can't see objects without KMS Key Permissions (Role Separation)

In Practice:
1. Create a bucket
2. Create a KMS Key, give no explicit permissions
3. Upload file to Bucket > Properties > Server-side Encryption > "Specify an Encryption Key"
    1. Use bucket setting for default encryption
    2. ✅ Override bucket settings for default encryption
        1. (SSE-S3) -  "photo1.jpg"
            - Key is visible in KMS/AWS Managed Keys
        2. (SSE-KMS)
            - Key is visible in KMS/Customer Managed Keys
            - Choose Key:
                1. Bucket Key - "photo2.jpg"
                2. Policy Key - "photo3.jpg"
        3. (DSSE-KMS) 
4. Create a Deny Policy for a user blocking KMS
    - Can open only photo1.jpg
    - But can delete it
5. You can Edit default Bucket encryption in S3 Bucket
    - Can choose Default Key or Customer Managed

S3 with Bucket Keys
- There is a bottleneck if you use KMS for each PUT call to a bucket
- You can use a Time-Limited Bucket Key instead to generate Data Encryption Keys(DEKs) within S3
- This reduces KMS API calls, reducing cost and improve Scalability
- S3 Bucket Keys:
    - If you use CloudTrail for KMS, events now show the Bucket
    - Works with same-region replication and cross-region replication. Object encryption is maintained
    - If replicating plaintext to a Bucket using Bucket Keys, the Object is encrypted at the destination side (ETAG Changes between source and destination)

S3 Object Storage Classes
- Objects are replicated across at least 3 AZs in the AWS Region
    - 99.999999999% durability
    - Redundancy, Content-MD5 Checksums, and CRCs are used to fix data corruption
- When objects are stored a HTTP/1.1 200 OK response is provided at S3 API endpoint
- 3 Classes:
    1. 'S3 Standard': Default Storage Class
        - Billing components:
            1. GB/month for storage
            2. $ for transfer OUT
            3. IN is free
            4. $ per 1000 Requests
            - No specific retrieval fee, no min. duration, no min. size
        - milliseconds first byte latency
    2. 'S3 Standard Infrequent Access' (S3 Standard IA)
        - Storage price much cheaper. Half of Standard
        - Use for data infrequently Accessed
        - Billing Components:
            1. per GB retrieval fee, cost increases with frequency
            2. min duration charge of 30 days
            3. minimum capacity charge of 128kb per Object
    3. 'S3 One Zone-IA'
        - Not copied in other AZ, less resilient
        - Used for IA data that is not critical, and is replaceable
        - Billing Components:
            1. Same as S3 Standard IA
            2. Cheaper as is not redundant

S3 Glacier 
- Instant
    - When you want fast retrieval, but not frequent
    - Billing:
        1. minimum capacity charge of 128kb per Object
- Flexible (formerly S3 Glacier)
    - Same 3 AZs as S3 Standard
    - Cold Objects, cannot be Publicly available [except metadata], must perform retrieval      
    - faster more $$
        - Expedited: 1-5 mins
        - Standard: 3-5 hours
        - Bulk: 5-10 hours
    - Billing:
        1. Storage is about 1/6th of Standard
        2. Pay for retrieval
    - First-byte latency = mins to hours
- Deep Archive
    - 40kb min. billable size
    - 180 day min Duration
    - Data is retrieved to S3 Standard-AI temporarily
    - Standard - 12 hours
    - Bulk - 48 hours
    - first-byte latency - hours to days

S3 Intelligent Tiering
- To use with long-living data when Frequency is unknown
- Object is monitored and automatically migrated
- If not accessed for 30 days, it goes to IA Tier and so on
- 5 Tiers
    1. Frequent Access Tier: = Starts here
    2. Infrequent Access Tier: > 30 days
    3. Archive Instant Access: > 90 days
    4. Archive Access: > 180
    5. Deep Archive: > 270
- If Object begin to go viral, they will move back to Frequent Access
- Billing has a monitoring and automation cost per 1000 Objects

S3 Lifecycle Configuration
- Buckets can have Lifecycle Rules to optimize, when Lifecycle is known
- Lifecycle Configuration is a set of rules, which consist of Actions on a Bucket or Group of Objects
    - Transition Actions - e.g.  when to put into cold storage
    - Expiration Objects - 
- The previously mentioned S3 and Glacier Tiers can all flow one into another, EXCEPT:
    - S3 One-Zone IA cannot go into S3 Glacier Instant Retrieval
- Careful when transitioning:
    1. Smaller Objects from Standard to IA, Intelligent Tiering,  or One Zone-IA because smaller objects cost more
    2. Standard to IA or One Zone-IA because of minimum of 30 days before transition 
    3. A single rule cannot transition to Standard-IA or One Zone-IA and then to Glacier classes within 30 days (because of duration minimums)

S3 Replication
- Architecture is only different depending if Buckets are in same or different AWS accounts
- Replication Configuration needs a Role with permissions to read Object on Source bucket and Replicate on Destination
    - Destination Bucket Account doesn't trust role by default
- Uses SSL
- 2 types of Replication:
    1. Cross-Region Replication: Source and Destination Buckets are in different AWS Region
    2. Same-Region Replication: Same Region
- Replication Options:
    1. All Objects or Subset
    2. Storage Class - default is to maintain same as source, but override may make sense($)
    3. Ownership - default is source account
    4. Replication Time Control (RTC) - adds a guaranteed 15 minute SLA
- Considerations:
    1. By default Replication is not retroactive
    2. Both Source and Destination Bucket need Versioning enabled
    3. Can use S3 Batch Replication to replicate existing Objects
    4. One-way Replication (not bi-directional by default)
    5. Unencrypted, SSE-S3 & SSE-KMS (with extra config), SSE-C
    6. Source Bucket owner needs permissions to Objects
    7. Will not replicate system Events, Glacier or Glacier Deep Archive (which are separate storage products)
    8. No deletes (But can be added ["DeleteMarkerReplication"])
- Use Cases
    - SSR - Log aggregation
    - SSR - PROD and TEST sync
    - SSR - Resilience with strict sovereignty
    - CRR - Global Resilience Improvements
    - CRR - Latency Reduction
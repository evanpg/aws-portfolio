IAM Policy Document
- One or more statements in JSON that grant or deny permissions to AWS Services
- "Sid": StatementID
- "Action": e.g. ["s3:*"]
- "Effect": "Allow" or "Deny"
    - Default is an implicit Deny (if nothing is stated, it is denied)
    - "Deny" [implicit], "Allow" [explicit], "Deny" [explicit] is best practice.
- "Resource": e.g. ["arn:aws:s3:::cats/*"]
- Can allow access to a large set, and in another statement deny a smaller subset

Managed Policies
- Reusable
- Low Management Overhead
- After setting group managed policy, you can add Special or Exceptional inline Policies
- Customer Managed Policy: a standalone, reusable permission policy that you create, own, and manage within your own AWS account. Unlike default AWS-managed policies, it lets you tailor precise least-privilege access rules, attach the policy to multiple users or roles, and track up to five historical versions

IAM users
- Anything requiring long-term AWS access e.g. Humans, Apps, or Service Accounts
- Authentication: uses Username and Password, or Access Keys
- Authorization: after authentication, IAM checks statements applied to Identity
- max 5000 users per account
- IAM user can be part of 10 groups
- IAM roles and Identity Federation can fix this

Amazon Resource Names (ARN)
- Uniquely identify resources within AWS accounts
- Uses following convention:
    - arn:partition:service:region:account-id:resource-type:resource-id
- "arn:aws:s3:::cats" vs "arn:aws:s3:::cats/*" are not the same
    - first is bucket access, second is objects in the bucket
    - in some cases you need both

IAM Groups
- Containers for IAM users
- Can have both Inline and Managed Policies 
- IAM users can be part of multiple IAM groups
- No limit for amount of users in group
- There isn't a built-in AllUsers group
- Can't have any nesting, groups within groups
- 300 groups per account, but can contact support for increase
- groups are not true identity and cannot be referenced as a principle in policy

IAM Roles
- Type of identity that exists in AWS account, Users is the other type (used in the Single Entity Principle)
- Role is for multiple entities accessing AWS services
- many times roles are used to temporary access
- IAM Roles are given 2 Policy types
    - Trust Policy: Identity A is allowed, B is not
        - If an entity is allowed Role
            - given Temporary Security Credentials, given by Secure Token Service (STS), which expire
    - Permissions Policy:
        - Every time Temporary Credentials are used it checks against Permissions Policy
- When to use
    - Lambda Functions use Roles to Execute, avoiding using Access Keys
    - Also used for Emergency or out-of-the-norm situations
        - A helpdesk Group member with read-only capabilities can access a Role to write, all events are logged
    - Roles can be used for SSO or if there are more than 5000 identities
    - External identities cannot directly use AWS resources, need role. ID Federation
    - Most WebApps have you sign in with SSO, giving access to IAM Role, which then access databases.
    - Scaling to 100,000,000+ users
    - Partner AWS accounts, when you need to access other account's resources
- Service-linked Roles
    - IAM role linked to a specific AWS Service
    - Predefined by a service, providing permissions that a service needs to interact with other AWS services on your behalf
    - Service might create/delete Role, or allow you to 
    - You can't delete Service role until it is no longer required
    - Role Separation: give one Group ability to create Roles, and another Group to use them
    - PassRole Permissions: Bob can configure a service with a Role which is already created by a member of the Security team, He just needs ListRole and PassRole Permissions on that specific Role
        - To give permissions to Services that he possibly does not have access to himself

AWS Organizations
- Allows organizations to run multiple AWS Accounts
- Organizational Root: a container that has:
    - "Management" or "Master" Account
    - "Member" Accounts
    - Other Containers aka Organizational Units (OU)
- Used commonly for Consolidated Billing where only one Account is Payer Account (for using AWS services, removing overhead and cost efficiency, volume discounts)
- If you create account within an organization, logins and permissions are simplified
    - Roles can be used to allow access between Organization's accounts
    - IAM is often used in just one Account, and RoleSwitch is used to access other accounts

In Practice:
- Giving production role to a demo account, for pushing changes
1. In admin account of production account go to Organization > Create Organization
2. Create another account and add it, or invite an existing account
3. Create a new IAM Role and select AWS Account as Trusted Entity Type
4. Select 'Another AWS Account' as Entity Type, and paste in the demo account number
5. Give admin access and name it, specifically, "OrganizationAccountAccessRole"
6. You can now switch between Roles in drop-down and access different Accounts

Service Control Policies
- Policy Document attached to root container, or individual sub-accounts
- SCPs are account permissions boundaries
- They limit what the AWS account can do (including root user)
- Account root user always has full permissions, but SCPs restrict what identities within account can do, effectively affecting root user
- They don't grant permissions, but define what can be granted or denied
- Inherit down the organizational tree, if nested Organizational Unit (OU) then they impact all accounts inside OU, and OUs nested within that OU
- If applied to just one account, then only that account is affected
- If Management account has SCP attached directly via OU or on root container, then Management is not applied to Management account. It *cannot be affected* by SCP.
- Can use in 2 ways:
    - Allow list:
        - Full Access: "Allow" "*"
        - Explicit: "Allow" "s3:*"
    - Deny list: 
        - Deny is by default, but can add explicit "Deny" "s3:*"
        - Deny list architecture is recommended for lower overhead
- The only things actually allowed must be in the SCP, and the Identity Policies and Accounts

In Practice:
1. In Organizations, click Root > Actions > Create New Organizational Unit
2. Name it "PROD" and Create Unit
3. Move Accounts into the Units (folders)
4. Go to Prod and create Bucket "cats999"
    - Currently you can Create Upload and view S3 Buckets
5. Switch back to Root
6. Go to Organizations > Policies
7. Create Policy "AllowAllExceptS3"
8. Go to AWS Accounts on left, Select Account, add that Policy
9. Go back to PROD and you can no longer view S3 buckets

CloudWatch Logs
- Public Service - usable from AWS or on-premises
- Store, Monitor, and Access logging data
- AWS Integrations - EC2, VPC Flow Logs, Lambda, CloudTrail, R53, and others...
- Can generate metrics based on logs - metric filter
- Log Events have timestamps and Message Blocks
- Log Streams: Log events from same source, one for each instance
- Log Groups: container for multiple streams
    - Have metric filters, which can have Alarms and trigger Actions

CloudTrail 
- Logs API actions on AWS account activities as a Cloudtrail Event
- By default stores for 90 days by default in Event history
- To customize the service, create 1 or more Trails
- Management Events and Data Events occur in the Region it is applied to
    - One Region Trail: just one Region that it's in
    - All Region Trail: One logical Trail
        - Global Services all log into us-east-1
        - IAM, STS, CloudFront area all global service events
- Data Events are not Enabled by default
- CloudTrails are logged to S3 and can use CloudWatch Logs
- You can also create Organization Trail
- CT is NOT realtime. There is a delay
- Logging Data Events costs $.10/100000 events

In Practice:
1. In management account, go to CouldTrails
2. Click "Create Trail". Give name. Log file validation is useful
3. CloudWatch Logs enabled, but you have to create a new IAM Role to give CloudTrail the ability to enter data into CloudWatch Logs
    - "CloudTrailRoleForCloudWatchLogs_Animals4Life"
4. Click Next and decide if you want Data Events
5. Decide if you want Read and/or write
6. Create Trail
7. Open Trail and click on S3 Bucket
    - You can then click through file structure to view independent logs
8. Go to CloudWatch > Log Management and click on the Log Group you created
9. Prefixes to groups match the Account Number, select a Stream to View Logs
10. You can also go to CloudTrail > Event History to search for individual Events / filter by user
11. You can Stop Logging by selecting Trail in CloudTrail and 'Stop Logging'

AWS Control Tower
- Quick and easy setup  of multi-account environments
- Orchestrates other AWS services to provide this functionality
- Organization, IAM Identity Center, CloudFormation, Config, and more
- Landing Zone: the multi-account environment
    - SSO/ID Federation, Centralized Logging & auditing
    - Home Region: explicitly allow or deny usage in region
    - Built with AWS Organizations, AWS Config, and CloudFormation
    - Security OU: Log Archive & Audit Accounts
    - Sandbox OU: Test/less rigid security
    - You can create other Ous and Accounts
    - IAM Identity Center (AWS SSO): SSOm multiple-accounts, ID Federation
    - Monitoring and Notifications: CloudWatch and SNS
    - End User account provisioning via Service Catalog
- Guard Rails: Rules for multi-account governance
    - Detect and Mandate rules/standards across new account creation 
    - 3 Rule types:
        - Mandatory: always applied
        - Strongly Recommended: recommended by AWS
        - Elective: for niche requirements
    - Guardrails function in 2 ways:
        - Preventative: stops you from doing things (AWS ORG SCP)
            - are enforced (actions prevented in any AWS Account) or not enabled
            - i.e. allow or deny regions to disallow bucket policy changes
        - Detective: compliance checks (AWS CONFIG Rules)
            - clear, in violation or not enabled statutes
            - i.e. Detect Guardrail checks whether CloudTrail is enabled in an AWS Account, or if EC2 instances have Public IPv4 associated with those instances
- Account Factory: Automates and Standardizes new account creation, provisioning long or short term
    - Only can be done by admins or end users with permissions
    - Guardrails are automatically added
    - Account admin given to a named user (IAM Identity Center)
    - Account and Network standard configuration for each
    - Accounts can be closed or repurposed
    - Can be fully integrated with a business Software Development Lifecycle (SDLC)
    - Useful for i.e. client demos or testing
- Dashboard - single page oversight of the entire environment
- When Control Tower is first set up it creates 2 OUs: Foundational OU (called Security) and Custom OU (called Sandbox)

Architecture:
- Outer Control Tower Container: Account Factory automatically creates and deletes Accounts as business needs them. 
    - Management Account has:
        - Inner Control Tower: utilizes CloudFormation this provides baseline for Account Factory. Also Uses AWS Config and Service policies to create guardrails, which prevent drifts from governments standards.
        - AWS Organizations:
            - Foundational OU creates 2 accounts:
                - Audit Account: For audit info made by Control Tower. SNS and CloudWatch
                - Log Archive Account: for users who need log info for all enrolled Accounts in landing zone. Must specifically grant access to Log Archive Account
                    - AWS Config
                    - CloudTrail
            - Custom OU
        - SSO via IAM uses internal or federated identities
AWS uses IAM to control who gets access to what.
You can create ACCESS KEYS to allow a user to access the AWS using command line.

1. In the web portal, create an admin role with Console Access role.
2. Create a user.
3. Give them both Access Keys.
4. Install aws-cli.
5. run:
```bash
    aws configure --profile [profilename]
```
6. paste public and private keys.
7. set region
8. run to make sure properly configured:
```bash 
aws s3 ls --profile [profilename]
```
9. In best practices, delete the files if the ACCESS KEYS were downloaded.
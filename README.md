# Assignment-2.15-secret-management-aws   

## Question 1:  
What is needed to authorize your EC2 to retrieve secrets from the AWS Secret Manager?      

## Answer 1:   
To authorize an EC2 instance to retrieve secrets from AWS Secrets Manager, you need to set up the following:

### 1. Attach an IAM Role to EC2 ###      
Since EC2 instances cannot use AWS credentials directly, you must attach an IAM role with the appropriate permissions.

Steps to create and attach the IAM Role:       
A. Create an IAM Role for EC2:   
   - Go to the AWS IAM Console → Roles → Create Role.
   - Choose AWS service and select EC2.    
   - Click Next to set permissions.
     
B. Attach a Policy to Allow Secrets Manager Access.      
   - Use an inline policy or managed policy (SecretsManagerReadWrite) that grants access to specific secrets.

  ***Example IAM Policy for Secrets Manager Read Access***
  ```
  {
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue"
            ],
            "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:my-secret-name-*"
        }
    ]
}
```
Replace 123456789012 with your AWS account ID.   
Replace my-secret-name-* with your actual secret name or prefix.    

C. Attach the IAM Role to EC2
   - Go to EC2 Console → Instances.   
   - Select the instance → Click Actions → Modify IAM Role.   
   - Choose the IAM role created above and attach it.

### 2. Configure EC2 to Retrieve Secrets ###
Now, your EC2 instance can retrieve secrets using AWS SDKs (Boto3, AWS CLI, etc.), without needing explicit credentials.

Example: Retrieve Secret Using AWS CLI   
Run the following command on the EC2 instance:   
```
aws secretsmanager get-secret-value --secret-id my-secret-name --query SecretString --output text
```
Example: Retrieve Secret Using Python (Boto3)    
```
import boto3
import json

def get_secret(secret_name):
    session = boto3.Session()
    client = session.client(service_name='secretsmanager', region_name='us-east-1')

    response = client.get_secret_value(SecretId=secret_name)
    
    secret = response['SecretString']
    return json.loads(secret) if secret.startswith('{') else secret

secret_value = get_secret("my-secret-name")
print(secret_value)
```

### 3. Additional Best Practices ###
- Least Privilege Principle: Grant access only to required secrets.   
- Use IAM Conditions: Restrict access based on EC2 tags, VPC, or specific instance roles.   
- Enable AWS CloudTrail: Monitor API calls to Secrets Manager.
<br>


## Question 2: 
Derive the IAM policy (i.e. JSON)?    
Using the secret name prod/cart-service/credentials, derive a sensible ARN as the specific resource for access.     

## Answer 2:    
To create an IAM policy that grants access to the secret named prod/cart-service/credentials, you need to derive the correct ARN (Amazon Resource Name) for the secret.       

### 1. Deriving the Secret's ARN ###
The general ARN format for an AWS Secrets Manager secret is:   
```
arn:aws:secretsmanager:<region>:<account-id>:secret:<secret-name>-<random-characters>
```
Since secret ARNs contain a random suffix, you can use a wildcard (*) to match all versions of the secret. Assuming the AWS region is us-east-1 and the AWS account ID is 123456789012, the secret's ARN would be:      
```
arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/credentials-*
```

### 2. IAM Policy to Allow Access to the Secret ###
Here’s the JSON IAM policy that grants read-only access to only this specific secret:    
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/credentials-*"
        }
    ]
}
```

### 3. Expanding the Policy for Additional Permissions (Optional) ###
If your EC2 instance also needs to describe the secret metadata (e.g., checking if the secret exists), you can add secretsmanager:DescribeSecret:   
```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": [
                "secretsmanager:GetSecretValue",
                "secretsmanager:DescribeSecret"
            ],
            "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/credentials-*"
        }
    ]
}
```

### 4. Adding IAM Condition (Optional - Restrict by EC2 Role) ###
To restrict access only to an EC2 instance role, you can use IAM condition keys:   

```
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Effect": "Allow",
            "Action": "secretsmanager:GetSecretValue",
            "Resource": "arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/credentials-*",
            "Condition": {
                "StringEquals": {
                    "aws:PrincipalArn": "arn:aws:iam::123456789012:role/MyEC2Role"
                }
            }
        }
    ]
}
```

- Replace "arn:aws:iam::123456789012:role/MyEC2Role" with your actual EC2 IAM role ARN.

### 5. Attaching the Policy to an IAM Role ###
To use this policy, attach it to an IAM role, then assign that role to your EC2 instance.

### Summary: ###
The derived ARN:
```
arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/cart-service/credentials-*
```
- The policy allows only read access (GetSecretValue) to that specific secret.    
- Optional conditions can be applied for additional security.    






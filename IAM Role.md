# Absence of aws credentials but using only: role assumption: 
### Explanation of the Difference

1. **Assuming Role vs. Providing Direct Access Keys**:
   - The original configuration relies solely on assuming an IAM role (`role-to-assume`). This is useful and secure, as it leverages AWS's STS (Security Token Service) to obtain temporary credentials.
   - However, to assume a role, your GitHub Actions runner needs initial AWS credentials (access key and secret key) to authenticate and request the role assumption.

2. **Adding Direct Access Keys**:
   - The updated configuration includes `aws-access-key-id` and `aws-secret-access-key`. These are required to initially authenticate with AWS before the role can be assumed.
   - This ensures that the GitHub Actions runner can authenticate with AWS, assume the specified IAM role, and obtain the necessary temporary credentials.

### When to Use Each Method

- **Using Role Assumption Only**:
  - If your GitHub Actions runner is already configured to assume roles using AWS's IAM roles attached to the instance or runner itself.
  - This is more common in setups where the runner is within AWS, and instance roles are used.

- **Providing Access Keys**:
  - When your runner needs to authenticate using access keys before it can assume a role. This is typically the case for GitHub-hosted runners or any external runners that need initial AWS credentials.

#how can i know if my GitHub Actions runner is already configured to assume roles using AWS's IAM roles attached to the instance or runner itself?


# How IAM Roles Work with EC2

When you attach an IAM role to an EC2 instance, the instance can use the role to obtain temporary credentials that allow it to access AWS resources. This eliminates the need to hard-code AWS credentials in your applications running on the EC2 instance.
Example Scenario:

    Create an IAM Role: The role includes policies that define the permissions needed.
    Attach the Role to an EC2 Instance: The EC2 instance automatically obtains temporary credentials that allow it to assume the role.
    Use the Role: Applications running on the EC2 instance can access AWS resources based on the permissions granted by the role.

# Setting Up and Using IAM Roles in GitHub Actions

Since your EC2 instance has an IAM role attached, you don't need to provide AWS access keys. Instead, your EC2 instance will use its IAM role to access AWS resources.


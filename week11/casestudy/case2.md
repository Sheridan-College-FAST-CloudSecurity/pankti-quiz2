# Leaked API Keys in Terraform State Files

## Example:

### Scenario:

A team used Terraform to provision AWS resources and configured an IAM user with programmatic access for application deployment:

```hcl
resource "aws_iam_access_key" "example" {
  user = "app-deployment-user"
}
```

Terraform state files (`terraform.tfstate`) automatically saved the generated AWS access key and secret key in plain text:

```json
{
  "resources": [
    {
      "type": "aws_iam_access_key",
      "instances": [
        {
          "attributes": {
            "id": "AKIAIOSFODNN7EXAMPLE",
            "secret": "wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY"
          }
        }
      ]
    }
  ]
}
```

The state file was inadvertently uploaded to a public GitHub repository during development.

### Incident:

In 2018, security researchers discovered that the state file of a major tech startup, "Harvested.io," was exposed in a public repository on GitHub. The Terraform state file contained unencrypted AWS API keys and other sensitive credentials. Attackers leveraged these credentials to create unauthorized EC2 instances and access private S3 buckets containing client data.

### Consequences:

- The company incurred significant financial losses due to unauthorized resource usage.
- Client trust was damaged as sensitive data was leaked.
- The incident forced the company to implement a comprehensive review of its security practices and adopt tools like Vault to manage secrets.

### Real-Life Example:

In 2023, the ["SCARLETEEL"](https://sysdig.com/blog/cloud-breach-terraform-data-theft/) attack showcased the dangers of exposed API keys in Terraform state files. In this incident, attackers gained access to a misconfigured Terraform state file stored in a publicly accessible S3 bucket. The file contained unencrypted AWS API keys and IAM roles with broad permissions. By leveraging these credentials, the attackers deployed unauthorized EC2 instances for crypto-mining and exfiltrated sensitive data from S3 buckets.

This breach highlights the importance of securing Terraform state files by encrypting sensitive data, implementing least-privilege principles for IAM roles, and using secure storage solutions such as encrypted S3 buckets with strict access controls. Adopting tools like HashiCorp Vault or AWS Secrets Manager to handle secrets can help prevent such incidents.


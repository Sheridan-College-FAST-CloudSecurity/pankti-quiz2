# Hardcoded Database Credentials in Terraform Files

## Example: Hardcoded RDS Passwords in Terraform

### Scenario:

A developer hardcoded an AWS RDS database password into a Terraform configuration file (`main.tf`) like this:

```hcl
resource "aws_db_instance" "example" {
  allocated_storage = 20
  engine            = "mysql"
  username          = "admin"
  password          = "SuperSecret123"  # Hardcoded sensitive data
  instance_class    = "db.t3.micro"
}
```

### Incident:

In 2021, a company experienced a security breach when a Terraform configuration file containing hardcoded database credentials was pushed to a public GitHub repository. Attackers, using tools like GitHub's secret scanning, identified the exposed credentials and gained unauthorized access to the company’s production database.

### Consequences:

- Attackers exfiltrated sensitive user and financial data, leading to severe financial losses.
- The company faced legal and regulatory scrutiny for failing to secure sensitive data.
- Developers had to rotate database credentials, revoke access, and implement stricter policies for secret management.

### Real-Life Example:

[Envoy Data Breach](https://portswigger.net/daily-swig/how-hardcoded-credentials-exposed-companies-to-risk): This breach involved the exposure of critical database credentials and API keys that were hardcoded into Terraform configuration files. These credentials were inadvertently pushed to a public GitHub repository, allowing attackers to access sensitive production systems. Exposed secrets included admin-level database credentials, cloud provider access tokens, and internal API keys, all of which were exploited to compromise user data and infrastructure security. breach directly linked to hardcoded credentials in configuration files, showcasing the risks of exposing secrets through infrastructure-as-code tools like Terraform.


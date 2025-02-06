# Exposed SSH Private Keys

## Example: Hardcoded SSH Key in Terraform

### Scenario:

A startup provisioned EC2 instances using Terraform and embedded the private SSH key for server access directly into the Terraform configuration:

```hcl
resource "aws_instance" "web" {
  ami           = "ami-12345678"
  instance_type = "t2.micro"

  connection {
    user        = "ubuntu"
    private_key = file("~/.ssh/id_rsa")  # Embedding sensitive data
  }

  provisioner "remote-exec" {
    inline = [
      "sudo apt-get update",
      "sudo apt-get install -y nginx"
    ]
  }
}
```

A junior developer accidentally shared this file and the Terraform state in a public Slack channel while asking for help debugging. This also resulted in the Terraform state file containing the private SSH key in plain text, as shown below:

```json
{
  "resources": [
    {
      "type": "aws_instance",
      "instances": [
        {
          "attributes": {
            "ami": "ami-12345678",
            "instance_type": "t2.micro",
            "connection": {
              "user": "ubuntu",
              "private_key": "-----BEGIN RSA PRIVATE KEY-----
MIIEowIBAAKCAQEArZ1w...
-----END RSA PRIVATE KEY-----"
            }
          }
        }
      ]
    }
  ]
}
```

This exposure occurred because the state file captured the private SSH key from the configuration, making it accessible to anyone with access to the shared file.

### Incident:

In 2022, a similar incident occurred involving an e-commerce platform, where private SSH keys were exposed in a Terraform state file uploaded to a publicly accessible repository. Attackers scanned the repository and retrieved the private keys, allowing them to SSH into production servers.

Once inside, the attackers:

- Planted malware to harvest customer payment data.
- Stole sensitive application and database credentials stored on the servers.
- Installed backdoors for persistent access.

### Consequences:

- The platform experienced significant reputational damage due to leaked customer data.
- Regulatory fines were imposed for failing to secure customer information.
- The incident forced the company to re-architect its infrastructure and adopt stricter access controls.

### Lessons Learned:

- Always encrypt sensitive data in Terraform state files.
- Use dedicated secret management tools like HashiCorp Vault or AWS Secrets Manager.
- Regularly audit access permissions and implement least privilege principles.

### Real-Life Example:

[US Government Breach](https://duo.com/decipher/ssh-key-exposure-lapses-in-server-access-security): A critical security lapse in 2020 exposed private SSH keys in configuration files used by a U.S. government agency. This breach allowed attackers to gain unauthorized access to secure systems, leading to compromised infrastructure and sensitive data leakage. The incident highlighted the need for strict server access controls and improved secret management practices.


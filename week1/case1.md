### Case Study 1: Capital One Data Breach (2019)

**Summary**: In 2019, Capital One suffered a major data breach that exposed sensitive data of over 100 million individuals.
The breach occurred because of a vulnerability in the cloud infrastructure, exploited by an attacker who gained access via a misconfigured Web Application Firewall (WAF). 
The attacker managed to retrieve AWS credentials, enabling them to access Capital One’s AWS environment and extract sensitive data stored in S3 buckets.
[Capital One Data Breach 2019](https://medium.com/nerd-for-tech/capital-one-data-breach-2019-f85a259eaa60)

**Discussion Questions**:

**Root Cause Identification**:

- What were the misconfigurations that led to this breach?
- How might insecure configurations in the DevSecOps pipeline have contributed?

**Suggested Remediations**:

- Which specific DevSecOps practices could have prevented this incident?
- What security controls could have alerted the team to unusual behavior earlier?
- How could Infrastructure as Code (IaC) and automated configuration scanning tools have helped secure the AWS environment?

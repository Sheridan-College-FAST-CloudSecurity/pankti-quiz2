
### Case Study 2: Uber Data Leak (2016)

**Summary**: In 2016, Uber experienced a data breach in which attackers accessed personal information of around 57 million Uber users. 
The breach occurred after attackers accessed Uber’s private GitHub repositories, where they found AWS credentials embedded in the code. 
Using these credentials, they accessed Uber’s data on AWS without encountering sufficient barriers.
[Uber Data Leak 2016](https://www.trendmicro.com/vinfo/us/security/news/cybercrime-and-digital-threats/uber-breach-exposes-the-data-of-57-million-drivers-and-users)
**Discussion Questions**:

**Root Cause Identification**:

- What security misconfigurations were present in Uber’s CI/CD and repository practices?
- How did the improper handling of secrets contribute to this incident?

**Suggested Remediations**:

- Which DevSecOps tools or processes could have prevented this issue (e.g., secrets management, CI/CD security)?
- How could Uber have used automated scans for secrets detection before code commits?
- How can restricted access and least-privilege principles prevent similar breaches?


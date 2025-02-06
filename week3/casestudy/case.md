## Case Study: Equifax Data Breach (2017)

### Background
In 2017, Equifax, a major credit reporting agency, experienced a data breach that exposed sensitive personal information of over 147 million people. 
This breach occurred due to a vulnerability in the open-source Apache Struts web application framework, which was not patched in a timely manner despite the availability of a fix.

[Equifax Data Breach (2017)](https://www.csoonline.com/article/567833/equifax-data-breach-faq-what-happened-who-was-affected-what-was-the-impact.html)

[Did lack of visibility into Apache Struts lead to the Equifax breach?](https://www.blackduck.com/blog/what-caused-equifax-breach.html)

### Key Issues

#### Delayed Patch Application
The vulnerability (CVE-2017-5638) was disclosed and patched by Apache in March 2017, but Equifax didn’t apply the patch until after the breach occurred, resulting in significant exposure.

#### Lack of Automated Security Testing
Equifax lacked automated security scanning tools in its CI pipeline that could have identified vulnerable dependencies like Apache Struts.

#### Limited Visibility and Reactive Security
Security practices were more reactive than proactive, with no processes to integrate vulnerability detection and patching into early stages of development and deployment.


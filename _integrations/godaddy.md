---
integration_point: gateway
name: GoDaddy
description: |
    "[GoDaddy](https://ae.godaddy.com/) is a domain registrar, web hosting company, and most relevant here, a public certificate authority.  This gateway allows the management of certificates on the GoDaddy hosting platform.
repo_url: https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-godaddy
binary_link: "[Latest Version - v1.0.0](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_build/results?buildId=13001&view=logs)"
---

### Description
[GoDaddy](https://ae.godaddy.com/) is a domain registrar, web hosting company, and most relevant here, a public certificate authority.

### Status
Most functionality is complete (see below).  At least one client is expected to be using this in a production environment sometime in 1st quarter 2021. 

### Source Code
[DevOps Repository](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-godaddy)

### Documentation
[AnyGateway Install & Configuration](https://kfeaus00web-01.corp.keyfactor.com/keyfactordocs/AnyGateway/v20.7/Generic/Content/AnyGateway/Introduction.htm)
**Note: as of 10/20 the product documentation does not cover the PowerShell cmdlets released with 20.9**
[GoDaddy AnyGateway Integration Configuration/README](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-godaddy?path=%2FREADME.md&version=GBmaster)
[GoDaddy API Documentation](https://developer.godaddy.com/doc)


### Supported Functionality
- Certificate Enrollment.  Domain Validated product suite supported (Regular, with SANs, and Wildcard)
- Certificate Revocation
- Certificate Sync


### Not Implemented/Supported
- Certificate Enrollment for Organization Validated product suite (Regular, with SANs, and Wildcard)
- Certificate Enrollment for Extended Validation product suite (Regular, with SANs, and Wildcard)
- Certificate Renewal/Reissue
- Approval/Denial of Enrollment Requests (not possible through the GoDaddy API library.


### Binary/Install
[Latest Version - v1.0.0](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_build/results?buildId=13001&view=logs)
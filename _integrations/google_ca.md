---
name: Google CA
integration_point: gateway
repo_url: https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-godaddy
description: 
binary_link:
---

### Description
The  [Google Certificate Authority Service](https://cloud.google.com/certificate-authority-service) is a highly available, scalable Google Cloud service that enables you to simplify, automate, and customize the deployment, management, and security of private certificate authorities (CA).

### Status: [Prototype](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_wiki/wikis/SolutionEngineering.wiki?wikiVersion=GBwikiMaster&pagePath=%2FIntegration%20Phases&anchor=pilot)

The initial development has been completed and documentation updated.  Validating test cases before scheduling a demo and review sessions with the team. 

### Build for AnyGateway 20.9
[![Build status](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_apis/build/status/Integration-AnyGateway-GoogleCA)](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_build/latest?definitionId=152)

### Documentation
[Google CA Service Quickstart Guide](https://cloud.google.com/certificate-authority-service/docs/quickstart) *documentation below assumes a CA is up and running
[AnyGateway 20.9 Reference Guide](https://kfeaus00web-01.corp.keyfactor.com/keyfactordocs/AnyGateway/v20.9/Generic/Content/AnyGateway/Introduction.htm)
[Google CA AnyGateway Plugin Readme](https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-googlecaservice?path=%2FREADME.md&version=GBmaster&_a=preview)

### Supported Functionality
* Synchronization (Enterprise Tier CA only) - Full and Incremental
* PFX & CSR Enrollment via Keyfactor Command
* Certificate Revocation


### Not Implemented/Supported
* Template Sync
The Google CA doesn't have the concept of products/templates.  The validity period and key size are determined by the AnyGateway configuration and CSR

### Feature Backlog
* Enable [Cloud Trace API](https://cloud.google.com/trace/docs)
* Enable [Logging of gRCP](https://googleapis.github.io/google-cloud-dotnet/docs/faq.html#how-can-i-trace-grpc-issues) calls to Keyfactor Logs
   * [Environment Variables](https://github.com/grpc/grpc/blob/master/doc/environment_variables.md)
* Determine product enhancements required to support enrollment via DevOps tier CAs. 
   * Changes will be required to the Google CA design for DevOps tier CAs as you cannot list issued certificates. As a result Sync for a DevOps tier CA will not be possible



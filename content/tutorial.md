---
sort: 5
---

# Quickstart Guides

To get you up and running quickly with custom orchestrator extensions, we have created step-by-step guides for both the latest **Universal Orchestrator** that is provided with Keyfactor version 9 and above, and the previous version; the **Windows Orchestrator**.  

## Universal Orchestrator
The Universal Orchestrator is the latest version of the Orchestrator that ships with Keyfactor 9.  The key improvements are:
- The Universal Orchestrator is written with the .NET Core portable libraries, so can run on nearly any operating system.
- The introduction of _Entry Parameters_; values that can be associated with individual certificate entries that allow you to track and manage certificates with improved granularity.
- We've provided a public NuGet package server that hosts all of the packages you need to create your own extension.

### <a href="{{'/examples/universal_orchestrator.html' | relative_url }}">Click here</a> for the Universal Orchestrator walkthrough.


## Windows Orchestrator
The Windows Orchestrator is still supported for clients who have not yet upgraded.  
This step-by-step example will guide you through the process of configuring, building, installing and testing your first Keyfactor integration.  We will use a simple Windows User Orchestrator example that will demonstrate how to implement custom functionality and provide a solid understanding of the process for creating your own integration.  

### <a href="{{'/examples/orchestrator.html' | relative_url }}">Click here</a> for the Windows Orchestrator walkthrough.




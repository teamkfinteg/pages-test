---
sort: 3
---

# Environment Setup
In order to get started developing your own custom Keyfactor integrations you will need to have an environment with the necessary prerequisites.  While some tool choices and IDE's could be replaced with your preferred version; for the purposes of this guide we will be assuming the following setup:

- [Visual Studio](https://visualstudio.microsoft.com/) (2015 or later)
- *For on premise installations, reach out to your Keyfactor Account Manager to ask about a virtual machine image for development.*
- To develop against a sandbox environment of the Keyfactor Cloud platform, your Keyfactor Account Manager can help set it up for you.

## Setting up your Keyfactor Cloud Sandbox
Once you've gotten the credentials and URL of your sandbox instance, confirm you can access the platform by logging in.

## Setting up the Virtual Machine
If you are developing against a virtual machine image, once you've gotten it up and started, you can log into it using the following credentials:

**username** `Administrator`

**password** `Password1`

You may want to provide a more friendly DNS name when referencing the Command platform locally rather than the IP address.  In order to do this; 
- open a powershell terminal on the virtual machine.  
- run the command `ipconfig`.  
- note the ip address
- on your development machine, open the file located at `c:\Windows\System32\drivers\etc\hosts` in a text editor running with administrator priveledges and add an entry such as `192.168.1.111      kftrain.keyfactor.lab`.

**For an example of debugging an interaction between the orchestrator and your non-production instance of Keyfactor, refer to [The Orchestrator quickstart](/examples/orchestrator/walkthrough).


## Package References
In visual studio you will need to add the Keyfactor public NuGet repository to the list of package sources.
`https://nuget.pkg.github.com/Keyfactor/index.json`
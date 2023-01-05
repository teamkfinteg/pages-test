---
name: A10vThunder
integration_point: orchestrator
description: | 
    The [A10 vThunder](https://www.a10networks.com/products/thunder-adc/) Orchestrator extension allows an organization to inventory and deploy certificates in any domain that the appliance services.  The Orchestrator deploys the appropriate files (.cer, .pem) within the defined directories and also performs and Inventory on the Items.
repo_url: https://devops.corp.keyfactor.com/MainCollection/SolutionEngineering/_git/anygateway-godaddy
binary_link: https://devops.corp.keyfactor.com/MainCollection/_apis/resources/Containers/21561?itemPath=Integration-AnyAgent-A10vThunder&$format=zip
---

### Intro Video
https://web.microsoftstream.com/video/0eca40c7-e364-4c99-b3b3-fb9c2590a3b0

### Documentation

- [A10 CLI Commands](/.attachments/A10Commands-06a0e42c-0002-4ee4-8715-150819df667c.pdf)
- [AXAPI](https://documentation.a10networks.com/ACOS/411x/411-P1/ACOS_4_1_1-P1/html/axapiv3/index.html) 
- [Postman Sample API Calls](https://app.getpostman.com/join-team?invite_code=07d8bd3cb44e42e10b5f1c298e07556a)


### Test Environment
The VM auto-shutsdown at 7PM every evening to keep costs down.  If you need to spin up the VM, here is a link to it.
1. [Powershell to Re-Create Azure vThunder VM](/.attachments/InstallVM2-ed903290-60a5-4054-8e3e-5f27b81bf9a3.ps1) (If needed)
2. [VM Link](https://portal.azure.com/#@csspkioutlook.onmicrosoft.com/resource/subscriptions/b3114ff1-bb92-45b6-9bd6-e4a1eed8c91e/resourceGroups/kVThunderA10/providers/Microsoft.Compute/virtualMachines/kVThunderA10/overview)
3. [A10 vThunder Web Management GUI](https://vthunder.eastus.cloudapp.azure.com:1113) (Turn on VM First)
   - UserId: admin
   - Password:ThunderStruck1!
4. SSH Settings to Access CLI (UserId:admin,Password:ThunderStruck1!)

   ![image.png](/.attachments/image-dedbf607-dd2d-435d-9460-b62b89187157.png)


### Supported Functionality
- Add/Delete/Replace Root Certificates
- Add/Delete/Replace Certificates with Public and Private Keys
- Inventory Root Certificates
- Inventory Certificates with Public and Private Keys

### Assumptions:
- In order for the Certificates and Keys to renew or reenroll correctly, they need to derive of the <alias> which is passed into the any agent.  The <alias> drives the files and object creation and is essentially how we are able to relate them to each other.

### Not Implemented/Supported
- Reenrollment, Management, Discovery


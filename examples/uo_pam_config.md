---
sort: 5
---

# Universal Orchestrator PAM Extension Quickstart Guide

This guide will explain how to configure the Universal Orchestrator to use local PAM Providers to resolve job parameters with PAM on the orchestrator.

## Overview

This capability requires using Universal Orchestrator framework 10.1+. When it is enabled, the Orchestrator will attempt to retrieve job parameters directly from a PAM Provider, instead of having the Keyfactor Platform interact with the PAM Provider. This use case helps when an Orchestrator exists within a firewalled environment where it can access a secure PAM Provider, but the Keyfactor Platfrom cannot.

### When can a PAM Provider be used?

A PAM Provider can be used on a compatible orchestrator when a secret value needs to be retrieved. In the Keyfactor Platform, these values can be stored encrypted in the database or retrieved with a PAM Provider on the platform. If the PAM Provider needs to be accessed from the Orchestrator, the PAM Secret Resolver is used.

### How can the PAM Secret Resolver be added to a Universal Orchestrator?

The following code fragment shows the needed `using` statement and addition to the Job Type constructor. This construction parameter is resolved automatically by the UO framework, and gives the Orchestrator job access to the `IPAMSecretResolver`.

~~~ c#
using Keyfactor.Orchestrators.Extensions.Interfaces;

namespace Keyfactor.Orchestrator.Sample.Jobs
{
    public class Inventory : IInventoryJobExtension
    {
        private IPAMSecretResolver SecretResolver { get; }

        public Inventory(IPAMSecretResolver secretResolver)
        {
            SecretResolver = secretResolver;
        }

        public JobResult ProcessJob(InventoryJobConfiguration config, SubmitInventoryUpdate submitInventoryUpdate)
        {
            // example of getting a Password from a PAM Provider
            string password = SecretResolver.Resolve(config.ServerPassword);
        }
    }
}
~~~

In this example, the value of the Server Password set in the Keyfactor Platform needs to be a JSON object containing the "instance-level" parameters for the PAM Provider. For the specific "instance-level" parameters needed, refer to the PAM Provider documentation of the provider being used.
Example JSON Object passed with 2 parameters:
~~~ json
{ "Parameter1": "Value1", "Parameter2": "Value2" }
~~~

### What needs to be configured on the Universal Orchestrator to enable PAM Provider usage?

When a UO Extension is being used that already supprots PAM Provider usage, some configuration needs to be done on the installed Universal Orchestrator, including adding the PAM Provider to be used.

#### Register the PAM Provider

A PAM Provider needs to be __registered__ on the Universal Orchestrator in the same way other extensions are. Add a folder for the specific PAM Provider to be added, and place the contents of the PAM Provider into the folder. There needs to be a `manifest.json` with the PAM Provider:

example `manifest.json` for `MY-PROVIDER-NAME`
~~~ json
{
    "extensions": {
        "Keyfactor.Platform.Extensions.IPAMProvider": {
            "PAMProviders.MY-PROVIDER-NAME.PAMProvider": {
                "assemblyPath": "my-pam-provider.dll",
                "TypeFullName": "Keyfactor.Extensions.Pam.MyPamProviderClass"
            }
        }
    }
}
~~~

After a `manifest.json` is added, the final step for configuration is setting the "provider-level" parameters for the PAM Provider. These are also known as the "initialization-level" parameters. These need to be placed in a `json` file that gets loaded by the Orchestrator by default - either the same `manifest.json`, the Orchestrator's `appsettings.json`, or added as an encrypted parameter in the `orchestratorsecrets.json` file which encrypts on the first start on Windows. If added to the encrypted location, you will need to re-enter all encrypted secret values so they can all be re-encrypted.

The settings should look like this if they were added to the `manifest.json` above:

example `manifest.json` for `MY-PROVIDER-NAME`
~~~ json
{
    "extensions": {
        "Keyfactor.Platform.Extensions.IPAMProvider": {
            "PAMProviders.MY-PROVIDER-NAME.PAMProvider": {
                "assemblyPath": "my-pam-provider.dll",
                "TypeFullName": "Keyfactor.Extensions.Pam.MyPamProviderClass"
            }
        }
    },
    "Keyfactor:PAMProviders:MY-PROVIDER-NAME:InitializationInfo": {
        "InitParam1": "InitValue1",
        "InitParam2": "InitValue2"
    }
}
~~~
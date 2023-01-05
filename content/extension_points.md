---
sort: 2
---

# Extension Points

The Keyfactor platform has a variety of integration points that can be extended to support nearly any enterprise or custom workflow scenario.

## Orchestrator

An Orchestrator (sometimes referred to as an Agent) is an service that typically runs on a machine for which CMS is managing the certificates.  The primary purposes of an orchestrator is to provide certificate inventory information to Keyfactor Command and to allow the platform to manage the certificates on that machine.  

The Orchestrator can be extended to apply custom handling of the orchestrator actions including:
- Certificate Inventory
- Certificate Management (Add, Remove)
- External system interaction
- 3rd Party vault integration
- Create Certificate Store
- Discovery (discover certificate stores).

## Registration Handler
Registration Handlers are called every time an orchestrator registers (logs in to) Keyfactor.  They are used to automatically approve orchestrators, assign blueprints, issue orchestrator certificates, and apply metadata to them.  These actions are performed by the Keyfactor Command platform when an orchestrator registers.

## CA Gateway
The Keyfactor Command platform natively supports Microsoft Certificate Authorities.
Certificate Authority Gateway Integrations allow the Keyfactor Command platform to interact with non-Microsoft Certificate Authorities; translating them into a format that the platform can interpret; and translating them back into the specified non-Microsoft Certificate Authority format.

## Approval Handler
Approval Handlers are custom actions to perform when a certificate is about to expire or when a CA workflow event (Waiting on approval, Approved, Rejected) fires.

## Metadata Generation
A Metadata Generation integration is code that gets called once per certificate in our database - has the ability to parse the certificate and generate related metadata, by retreiving information from an external system if necessary.

## Orchestrator Registration Parameters

When an Orchestrator authenticates with the Keyfactor platform it has the opportunity to request a new client authentication certificate and to specify set of client parameters to be passed to the Registration Handler. This is an extension point for the Windows Orchestrator to allow for CSR and client parameter submission.

## Powershell Handlers

Workflow events in the Keyfactor platform such as Certificate Expiration, Issuance, or Renewal, as well as SSH Key Rotation. The event handlers associated with these workflow events can be configured to run a Powershell script.

[About Workflow Handler Powershell Scripts]({% link examples/powershell_handlers.md %})

## Job Complete Handlers

Keyfactor Orchestrators completing a job such as Certificate Store management or SSL scanning report back to the Keyfactor platform with their final status of the job. A Job Complete Handler can run .NET code extensions on the Keyfactor platform after a job reports its complete status.

[About Job Complete Handlers]({% link examples/job_complete_handlers.md %})

[Job Complete Handler Technical Reference]({% link references/job_complete_handler_reference.md %})
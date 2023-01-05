# Job Complete Handler Object Reference

## Overview

A Job Complete Handler, once added as an extension and configured, runs when a Keyfactor Orchestrator checks in to the platform with its job status after completing a job.

Below are references for the interface and context object used to create a Job Complete Handler.

### IOrchestratorJobCompleteHandler Interface

This interface comes from the NuGet package `Keyfactor.Platform.Extensions.IOrchestratorJobCompleteHandler`. A new handler class needs to implement this interface to be registered and used as a Job Complete Handler.

~~~ c#
namespace Keyfactor.Platform.Extensions
{
    //
    // Summary:
    //     Keyfactor extensibility point to allow additional or custom functionality following
    //     agent job completion.
    public interface IOrchestratorJobCompleteHandler
    {
        //
        // Summary:
        //     A comma separated list of the guid job type ids for which this handler should
        //     run.
        string JobTypes { get; }

        //
        // Summary:
        //     RunHandler is called after an agent job completes.
        //
        // Parameters:
        //   context:
        //     An object containing the details of the job that just completed.
        //
        // Returns:
        //     A boolean indicating whether the handler completed successfully or not.
        bool RunHandler(OrchestratorJobCompleteHandlerContext context);
    }
}
~~~

### OrchestratorJobCompleteHandlerContext Object

This object also comes from the NuGet package `Keyfactor.Platform.Extensions.IOrchestratorJobCompleteHandler`. The `OrchestratorJobCompleteHandlerContext` object comes in as the "context" of the job completion event when the Job Complete Handler executes. 

The `Client` property has a HTTP Client pre-configured to target the Keyfactor API endpoint, and set with `UseDefaultCredential = true`. This means that the API request will be made from the Keyfactor Server itself using the App Pool user credentials by default.

~~~ c#
namespace Keyfactor.Platform.Extensions
{
    //
    // Summary:
    //     Agent and job information given to the handler.
    public class OrchestratorJobCompleteHandlerContext
    {
        //
        // Summary:
        //     AgentId declared by agent
        public Guid AgentId;
        //
        // Summary:
        //     Username declared by agent
        public string Username;
        //
        // Summary:
        //     ClientMachine declared by agent
        public string ClientMachine;
        //
        // Summary:
        //     The final status of the job.
        public OrchestratorJobStatusJobResult JobResult;
        //
        // Summary:
        //     JobId declared by the job.
        public Guid JobId;
        //
        // Summary:
        //     The type of job.
        public string JobType;
        //
        // Summary:
        //     The JobTypeId declared by the job (Id of the JobType).
        public Guid JobTypeId;
        //
        // Summary:
        //     The type of operation for the job.
        public CertStoreOperationType OperationType;
        //
        // Summary:
        //     The Keyfactor id of the certificate being managed (if applicable).
        public int? CertificateId;
        //
        // Summary:
        //     The time the job was requested (if available).
        public DateTime? RequestTimestamp;
        //
        // Summary:
        //     The number of times the current job has retried.
        public int CurrentRetryCount;
        //
        // Summary:
        //     A pre-configured System.Net.Http.HttpClient object for making additional calls
        //     to Keyfactor for more information.
        public HttpClient Client;

        public OrchestratorJobCompleteHandlerContext();
    }
}
~~~
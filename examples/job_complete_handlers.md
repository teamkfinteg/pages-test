---
sort: 4
---

# Orchestrator Job Complete Handlers

This guide will explain how a Keyfactor Orchestrator interacts with Job Complete Handlers, including how they are configured, what they can do, and what context information is available when writing one.

Technical object references [can be found here]({% link references/job_complete_handler_reference.md %}).

## Overview

Job Complete Handlers are a new feature in Keyfactor 9. They enable execution of code extensions after Orchestrator jobs complete.

### What is a Job Complete Handler used for?

Keyfactor Orchestrators check in with the main platform when they complete a job, whether it succeeds, fails, or has another status. An orchestrator may be performing an SSL scan, inventorying certificates, or managing certificates in a certificate store.

A Job Complete Handler can run custom code in order to take additional actions after that job completes, like sending an email or making additional API calls to Keyfactor or other systems. You can define custom actions in this way that you want to execute after an orchestrator job is finished.

### When can a Job Complete Handler run?

All orchestrator job types available to Keyfactor Orchestrators will be able to execute a handler when the job completes and the orchestrator checks back in with Keyfactor. SSL jobs, built-in Keyfactor certificate store jobs, custom integration jobs, and custom job types are all supported.

A Job Complete Handler can be configured to run for every single job type, or can be set to only run for specific job types. It will automatically run for any job status, success or failure, and you can handle the specific completed job's status in your handler code.

### What can a Job Complete Handler do?

The handler gets various fields with context information loaded into it when it executes. Control logic in the handler can be designed to perform reporting with your systems, make additional Keyfactor API calls, or even send emails in specific scenarios where you want to be directly notified of a failure.

Here is a code snippet showing how a Job Complete Handler might send an email:

~~~ c#
public bool RunHandler(OrchestratorJobCompleteHandlerContext context)
{
    if (context.JobResult != OrchestratorJobStatusJobResult.Failure
        || context.CurrentRetryCount < int.Parse(RetryThreshold))
    {
        // job did not fail outside of Retry Threshold
        return true;
    }

    var smtp = new SmtpClient();
    smtp.Connect(SmtpServer, int.Parse(SmtpPort));
    smtp.Authenticate(Username, Password);
    string message = BuildMessageBody(context);

    var mail = new MimeMessage();
    mail.From.Add(MailboxAddress.Parse(MailFrom));
    var addressess = new List<MailboxAddress>();
    foreach (var recipient in EmailRecipients.Split(','))
    {
        addressess.Add(MailboxAddress.Parse(recipient));
    };
    mail.To.AddRange(addressess);
    mail.Subject = $"Orchestrator {context.JobType} Job Failed";
    mail.Body = new TextPart("plain")
    {
        Text = message
    };

    smtp.Send(mail);
    smtp.Disconnect(true);
    smtp.Dispose();

    return true;
}

private string BuildMessageBody(OrchestratorJobCompleteHandlerContext context)
{
    StringBuilder builder = new StringBuilder();
    builder.AppendLine("Orchestrator Job failed too many times:");
    builder.AppendLine($"\tType: {context.JobType}");
    builder.AppendLine($"\tJobId: {context.JobId}");
    builder.AppendLine($"\tRetries: {context.CurrentRetryCount}");
    return builder.ToString();
}
~~~

In the above example, the passed in `context` is used to check for the job's retry count and status, so that emails can be sent after repeated failures of the job. The context is also used later to populate information for the email body itself.

Within the handler, an SmtpClient is set up (using the open-source MailKit in this example) using properties (not-shown) with values defined during the configuration of the handler. 

<!-- #### Calling Keyfactor APIs -->
<!-- Section to be completed. -->

### Writing a new Job Complete Handler Implementation

Writing a new Job Complete Handler is done by implementing the `IOrchestratorJobCompleteHandler` interface. This interface is found in the NuGet package `Keyfactor.Platform.IOrchestratorJobCompleteHandler`. Including this package will bring in dependencies that also include the definition of the context object, `OrchestratorJobCompleteHandlerContext`.

~~~ c#
using Keyfactor.Platform.Extensions;

public namespace MyNamespace
{
    public class MyHandler : IOrchestratorJobCompleteHandler
    {
        public bool RunHandler(OrchestratorJobCompleteHandlerContext context)
        {
            // return true or false for successful completion of the handler
        }
    }
}
~~~

### Adding and Configuring the Job Complete Handler

The Job Complete Handler is configured and enabled via editing of a `web.config` within a Keyfactor installation. Within the Keyfactor installation directory, you will head to _WebAgentServices_ and edit the `web.config` there. Looking at the default contents of the file, you can locate where configuration details for Job Complete Handlers go:

~~~ xml
<!--The following are Orchestrator Job Complete Handler registrations. Uncomment to use.-->
<!--<register type="IOrchestratorJobCompleteHandler" mapTo="CSS.CMS.Agents.Server.Handlers.SendEmailOrchestratorJobCompleteHandler, CSS.CMS.Agents.Server" name="SendEmailOrchestratorJobCompleteHandler">
<property name="EmailRecipients" value="" />
<property name="JobTypes" value="" />
</register>-->
~~~

Uncommenting the above `<register>` block would enable the built-in Job Complete Handler, which sends an email with basic details whenever a job completes. Note the two properties also defined:
- "EmailRecipients" : a property defined for this specific handler, a comma separated list of emails to send notices to
- "JobTypes" : the __required__ property for all Job Complete Handlers

If we wanted to enable this handler, we would need to provide the _GUID_ for each of the JobTypes that we want the handler to run for. As an example, if the handler should run for all SSL "Monitoring" and "Compliance" jobs, I would look up the _GUID_ for both types of jobs, and put them as the `value` with a comma separating them.

#### Adding a new Job Complete Handler

Instead of enabling the default handler, I want to add a new handler. Here is a snippet of the handler I am going to add:

~~~ c#
public class SendEmail_RepeatedFailureHandler : IOrchestratorJobCompleteHandler
{
    public string SmtpServer { get; set; }
    public string SmtpPort { get; set; }
    public string MailFrom { get; set; }
    public string Username { get; set; }
    public string Password { get; set; }
    public string EmailRecipients { get; set; }
    public string RetryThreshold { get; set; }
    public string JobTypes { get; set; }

    // etc...
}
~~~

My handler has properly chosen to implement the interface necessary, and there are also several properties added to the class. The __required "JobTypes"__ property is also included.

In order to add this handler, I would first build the project it is part of. Then I would copy the DLLs and dependent libraries over to the _WebAgentServices\bin_ directory, which is the _bin_ directory under where the `web.config` we will edit is located.

> Important note: when copying over dependencies for the handler, it is important to not override existing DLLs in the target location. You will need to make sure that the handler references the same versions of libraries already in use in the _WebAgentServices_ location. A compatibility list for library dependencies is in the works.

After adding the built code for the handler, the `web.config` needs to be edited to register the handler:

~~~ xml
<register type="IOrchestratorJobCompleteHandler" mapTo="SampleJobCompletionHandlers.SendEmail_RepeatedFailureHandler, SampleJobCompletionHandlers" name="SendEmail_RepeatedFailureHandler">
    <property name="SmtpServer" value="my.smtp.email" />
    <property name="SmtpPort" value="587" />
    <property name="Username" value="myuser@domain.local" />
    <property name="Password" value="somepassword" />
    <property name="MailFrom" value="my.name@smtp.email" />
    <property name="EmailRecipients" value="the.recipient@smtp.email" />
    <property name="RetryThreshold" value="2" />
    <property name="JobTypes" value="7B6BBC1B-D802-4B09-BEC9-F5C221083130,44340A94-9B99-410D-8011-598FB176C3C5" />
</register>
~~~

This Unity registration will require that the web server is restarted, which can be done when safe by running `iisreset` in a command console. The Agent API server should be checked at this point as errors in the handler registration can prevent other Keyfactor Orchestrators from communicating with the platform.



## Additional References
Some snippets in this document can be found from the original source in the [samples-jobcomplete-handler](https://github.com/keyfactor/samples-jobcomplete-handler) repository. Full samples can be found there as well.
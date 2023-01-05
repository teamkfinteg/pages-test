---
sort: 3
---

# Workflow Handler Powershell Scripts

This guide will show how to set up a Powershell script for use with various types of Workflow Handlers. A code sample will be included, but a more complete set of examples is available in the Keyfactor GitHub.

## Overview

### When can a workflow handler run?

Workflow Handlers can be associated with various certificate states, and will run when a certificate enters that state. There are different actions that can be run at these different states.

The workflow states that can have handler actions run are:
- Expiration (before and at the expiration date)
- Pending (awaiting approval)
- Issuance (and approval of a pending certificate)
- Denial (rejecting a pending request)
- SSH Key Rotation (when using Keyfactor to manage SSH keys)

Additionally, restrictions can be placed in the Keyfactor Platform during configuration of the handler to limit the action to one certificate template type, or to specific timeframes in the case of expiration.

### What can a workflow handler do?

This document will mainly look at how the Powershell handler can be used. There are other built in handler types that are able to run special logging procedures or in the case of an expiration, run a renewal of the certificate. When using a Powershell handler, we will be able to do arbitrary actions from Powershell with data that is passed in from the specific workflow event.

Passing in data from the workflow event is how specific actions can be taken based on the certificate information in the event. The data we want available in the Powershell script will need to be known and configured with the Event Handler in the Keyfactor Platform.

Adding a parameter to pass data from the workflow event to be used in Powershell is straightforward.

1. First, make sure that a Powershell handler is selected. You will have to have filled out the other information for the alert beforehand.

![image]({{site.baseurl}}/assets/images/powershell_handlers/handler-configure.png)

1. When configuring the handler, you can add static values under variable names, or use special values that will pass dynamic values to the Powershell such as the certificate email.

![image]({{site.baseurl}}/assets/images/powershell_handlers/handler-special-value.png)

In the Powershell, we can then access that `cert-email` value:

~~~ powershell
[hashtable]$context

$context["cert-email"]
~~~

For a more complete example of configuring the handler for Powershell and how to pass values, check the official Keyfactor Reference Guide included with your installation, and see the document titled "Adding PowerShell Handlers to Alerts".

## Usage Scenarios

A couple of example scenarios for using a Powershell handler to accomplish a business example will follow. Code samples have been included but should be tested out before being used in your production environment.

### Expiration

An expiration alert handler can be useful for customizing auto-renewal logic, especially if additional conditional logic needs to be applied. Another option would be to customize the emails to be sent to notify about an expiring certificate, if the built-in email customization in the Keyfactor Platform is not sufficient.

The following example executes a renewal on an expiring certificate via Keyfactor API calls.

~~~ powershell
[hashtable]$context
$DN = $context["dn"]
$CA = $context["ca"]
$Thumb = $context["thumb"]

# script variables
#############################################################
$apiUrl = "https://keyfactor01.mykeyfactorinstance.com/KeyfactorAPI" # update to be Keyfactor API endpoint
$pfxUrl = "https://keyfactor01.mykeyfactorinstance.com/KeyfactorAPI/Enrollment/PFX"
$ApiKey = "ABCmybase64key=="
$Template = "MyCertificateTemplate"
$username = "domain\api-user" #To Do: Change to your API User
$password = "api-user-password" | ConvertTo-SecureString -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($username, $password)
$PFXPwd = "Password1234" # password for the private keys being entered into Keyfactor for the PFX certificates
$CA = "MyCA.domain.local\\MyCA-CA" # change to your issuing CA - might require format of: <CA Host Name>\\<CA Logical Name>
$LogDest = "C:\Scripts\RequestNoSansCert_Sectigo\Logs\log.log" # the location for the error log file, the script will create this file
#############################################################

Function LogWrite($LogString)
{
    Add-Content $LogDest -value $LogString
}

Function ReplaceCert ($OldID, $NewID)
{
    try
    {
        $replaceURL = $pfxUrl + "/Replace"
        $timeStamp = (Get-Date).ToUniversalTime()

        $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
        $headers.Add('content-type', 'application/json')
        $headers.Add("X-Keyfactor-Requested-With", "APIClient")

        $Body = @"
{
    "ExistingCertificateId": $OldID,
    "CertificateId": $NewID,
    "Password": "$PFXPwd",
    "JobTime" : "$timeStamp"
}
"@  

        $certificateResponse = Invoke-RestMethod `
        -Method Post `
        -Uri $replaceUrl `
        -Credential $credential `
        -Headers $headers `
        -Body $Body `
        -ContentType "application/json"

    }
    catch
    {
        LogWrite "An error occurred replacing the bindings of the cert in the store"
        LogWrite $_
        return "REPLACE_ERROR" 
    }
}

Function GetId
{
    try
    {
        $searchURL = $apiUrl + "/certificates/?verbose=1&maxResults=50&page=1&query=Thumbprint%20-eq%20`""+$Thumb+"`""

        $certificateResponse = Invoke-RestMethod `
        -Method Get `
        -Uri $searchUrl `
        -Credential $credential `
        -ContentType "application/json"

        Return $certificateResponse.Id
    }
    catch
    {
        LogWrite "An error occurred looking up the certificate in keyfactor"
        LogWrite $_
        return "SEARCH_ERROR"        
    }
}

Function BuildCertRequest($ID)
{
    #Build Request
    $timeStamp = (Get-Date).ToUniversalTime()
    $Body = @"
{
    "timestamp" : "$timeStamp", 
    "IncludeChain": true,
    "CertificateAuthority": "$CA",
    "Template": "$Template",
    "RenewalCertificateId": "$ID",
    "Subject": "$DN",
    "Password": "$PFXPwd"
}
"@
    return $Body
}


Function MakeRequest($Body,$name,$API)
{
    #create secure headers
    # ===== Construct Headers
    $headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
    $headers.Add('content-type', 'application/json')
	$headers.Add("X-Keyfactor-AppKey", $API)
    $headers.Add("X-CertificateFormat", "STORE")
    $headers.Add("X-Keyfactor-Requested-With", "APIClient")

    try
    {
        #make certificate request
        $certificateResponse = Invoke-RestMethod `
	        -Method POST `
	        -Uri $pfxUrl `
	        -Headers $headers `
	        -Body $Body `
	        -ContentType "application/json" `
	        -Credential $credential

        return $certificateResponse.CertificateInformation.KeyfactorId
    }

    catch
    {
        LogWrite "An error occurred requesting the certificate from keyfactor"
        LogWrite $_
        return "REQUEST_ERROR"
    }
}

try
{
    LogWrite (Get-Date).ToUniversalTime().ToString()
    LogWrite $DN
    LogWrite $Thumb
}
catch
{
    LogWrite "An error occurred reading info into the logs"
    LogWrite $_
    return "INFO_ERROR"
}

try
{
    #get CertID
    $CertID = GetId        
    #generate a pfx request
    $Request = BuildCertRequest $CertID
    #make API request and store new CertID
    $NewID = MakeRequest $Request $PFXName $ApiKey
    #Replace the Bindings on the new cert
    ReplaceCert $CertID $NewID
}
catch
{
    LogWrite (Get-Date).ToUniversalTime().ToString()
    LogWrite "Script Failed Gracefully"
}

~~~

### Pending

A pending alert handler can look at certificates that are still awaiting approval. If your certificate approval is a manual process, it's possible that the approver missed the original request or subsequent alerts. A Powershell handler could calculate a pre-determined time window, and then send a high-priority notice to the approver that the certificate is still awaiting review.

~~~ powershell
[hashtable]$context
$pendingWindow = $context["timeframe"] # hardcoded parameter
$requestDate = $context["requestDate"] # Submission Date variable

# calculate timespan since request date
$currentDate = Get-Date
$dateDiff = New-TimeSpan -Start $requestDate -End $currentDate

if ($dateDiff.days -gt $pendingWindow) {
    # send a priority email
    $smtp = New-Object Net.Mail.SmtpClient("mykeyfactorinstance.com")
    $mail = New-Object System.Net.Mail.MailMessage

    $mail.To.Add($context["Recipient"])
    $mail.Subject = "PRIORITY - Pending Certificate - " + $context["Subject"]
    $mail.Body = $context["Body"]

    $mail.Priority = [System.Net.Mail.MailPriority]::High
    $smtp.Send($mail)

    # tell Keyfactor Platform not to send its own email
    $context["SendEmail"] = "false"
}
~~~

### Issuance

An issuance alert handler might be used if special business actions need to be taken when a certificate is successfully issued. The email notifying about certificate issuance may be customized in the Powershell to include the certificate as an attachment - a behaviour not supported by default in the Keyfactor Platform.

~~~ powershell
[hashtable]$context
$thumb = $context["Thumbprint"]

$apiUrl = "https://keyfactor01.mykeyfactorinstance.com/KeyfactorAPI" # update to be Keyfactor API endpoint
$username = "domain\api-user" #To Do: Change to your API User
$password = "api-user-password" | ConvertTo-SecureString -AsPlainText -Force
$credential = New-Object System.Management.Automation.PSCredential($username, $password)


# download Keyfactor Cert
$downloadUrl = $apiUrl + "/certificates/download"

$Body = @"
{
    "Thumbprint": $thumb,
    "IncludeChain": "false"
}
"@

$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add('content-type', 'application/json')
$headers.Add("X-Audit-Requested-With", "APIClient")
$headers.Add("x-certificateformat", "PEM")

$downloadResponse = Invoke-RestMethod `
-Method Post `
-Uri $downloadUrl `
-Headers $headers `
-Body $Body `
-Credential $credential `
-ContentType "application/json"

$path = ".\" + $thumb + ".pem"
Set-Content -Path $path -Value $downloadResponse

# send email with an attachment
$smtp = New-Object Net.Mail.SmtpClient("mykeyfactorinstance.com")
$mail = New-Object System.Net.Mail.MailMessage
$attachment = New-Object Net.Mail.Attachment($path)

$mail.To.Add($context["Recipient"])
$mail.Subject = $context["Subject"]
$mail.Body = $context["Body"]

$mail.Attachments.Add($attachment)

$smtp.Send($mail)

# tell Keyfactor Platform not to send its own email
$context["SendEmail"] = "false"
~~~

### Denial

A denial alert Powershell handler might execute an API call to create a special event in your business system. This audit record or security alert could then be used to keep track of denied certificates in case they need to be reviewed for suspicious activity.

~~~ powershell
[hashtable]$context
# many variable values are passed in the context to provide information from the Keyfactor Platform

$headers = New-Object "System.Collections.Generic.Dictionary[[String],[String]]"
$headers.Add('content-type', 'application/json')
$headers.Add("X-Audit-Requested-With", "api")

$content = "A certificate was denied for issuance. The request information:"
$content = $content + "`r`n Requested CN: " + $context["CN"]
$content = $content + "`r`n Requested DN: " + $context["DN"]
$content = $content + "`r`n Requested SANs: " + $context["SANs"]
$content = $content + "`r`n Requested Issuing CA: " + $context["CA"]
$content = $content + "`r`n Requested Template: " + $context["template"]
$content = $content + "`r`n Requestor: " + $context["firstName"] + " " + $context["lastName"]

$subject = $context["Subject"]
$denialReason = $context["denialReason"] 
$Body = @"
{
    "subject": $subject,
    "content": $content,
    "reasonCode": "CERTIFICATE_DENIED",
    "reasonText": $denialReason
}
"@

$certificateResponse = Invoke-RestMethod `
-Method Post `
-Uri "https://myauditserver.com/api/audits" `
-Headers $headers `
-Body $Body `
-ContentType "application/json"

~~~


## Additional References

For more information about configuring the Keyfactor Platform to use Powershell Handlers, check the Reference Guide included in your Keyfactor installation.

These code samples and others can be found in the [keyfactor-samples-alerthandler](https://github.com/keyfactor/keyfactor-samples-alerthandler) repository.
# LetsEncrypt

This repository contains a sample script to setup and use the free SSL certificates issued by [Lets Encrypt](https://letsencrypt.org/) together with Business Central Docker containers.

The PowerShell script in this repository (SetupCertificate.ps1) can be used as an override to the default way [BcContainerHelper](https://github.com/microsoft/navcontainerhelper) creates self-issued SSL certificate.

## Usage

You can use the script in two ways. Either by specifying it when creating the container with New-BcContainer or by copying it into the container.

### Using it with New-BcContainer

To use it when creating a new container, then you need to specify the url to script in the myScripts parameter. 
You also need to specify the email address used Lets Encrypt to register the certificate and a password allowing you to reuse the same certificate if you restart the container.

The script sample below shows the minimum required parameters to use the script.

    $Params = @{
        accept_eula          = $true
        artifactUrl          = Get-BCArtifactUrl
        useSSL               = $true
        PublishPorts         = @(8080, 443, 7046, 7047, 7048, 7049)
        PublicDnsName        = 'myTestVm.westeurope.cloudapp.azure.com'
        myScripts            = @("https://raw.githubusercontent.com/NAVDEMO/LetsEncrypt/master/SetupCertificate.ps1")
        additionalParameters = @("--env ContactEMailForLetsEncrypt=test@nodomain.com", "--env CertificatePfxPassword=aPa55w0rd")
    }
    New-BcContainer @Params

### Using it with an existing container

It is also possible to use it with an existing and already running BC container. In that case you need to download the script file and copy it to the containers My folder (i.e. C:\ProgramData\BcContainerHelper\Extensions\BcServer\My) as **SetupCertificate.ps1**.

You also need to specify the ContactEMailForLetsEncrypt and CertificatePfxPassword environment variables. This can be done by using another script override **SetupVariables.ps1** and also copy this file to your containers My folder.

    $Env:publicDnsName = 'myTestVm.westeurope.cloudapp.azure.com'
    $Env:ContactEMailForLetsEncrypt = 'test@nodomain.com'
    $Env:CertificatePfxPassword = 'aPa55w0rd'

    $certificatePfxFile = Join-Path $myPath "certificate.pfx"
    if (-not (Test-Path $certificatePfxFile -ErrorAction SilentlyContinue)) {
        $certificatePfxFile = ""
    }
    $Env:certificatePfxFile = $certificatePfxFile

    # Invoke default behavior
    . (Join-Path $runPath $MyInvocation.MyCommand.Name)

    . (Join-Path $PSScriptRoot "updatehosts.ps1") -hostsFile "c:\driversetc\hosts"

The sample script also checks if a certificate.pfx already exists in the containers My folder. If it does then it avoids requesting a new certificate, but will reusing the one already created.

## Notice

The Lets Encrypt certificates are only valid for 90 days and you are only allowed to request a few new certificates for the same domain name per day.

If the script fails to run successfully, then it will revert to the default functionality and create a self-signed SSL certificate.

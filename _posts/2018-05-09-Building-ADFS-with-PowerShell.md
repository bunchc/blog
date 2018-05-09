---
title: "Building ADFS with PowerShell"
date: 2018-05-09
layout: "post"
categories: powershell, windows, automation, windows 2012R2, windows 2016, win2012R2, win2016, federation, active directory, AD, ADFS
---

I have had a need recently to have a number of OpenSource projects authenticate against Microsoft Active Directory. While there are many ways to do this, ADFS, or Active Directory Federation Services allows us to use SAML, which in turn can be tied into 3rd party Single Sign On tools (Okta, Facebook, etc.)

## Getting started

In order to use this script, you will need:

* A Windows server, either 2012R2 or 2016
* Active Directory
    - Schema level of at least 2012R2
* User account with Domain Admin permission
* PowerShell 5.x
    - Older versions _may_ work, but are untested

## Installing ADFS with PowerShell

To install ADFS with powershell, log into the Windows server where ADFS is to be deployed, and:

1. Open PowerShell
2. Download the [script](https://gist.githubusercontent.com/bunchc/616446049eec0b2497051ca292e816b7/raw/cefb4852ed132dccd95886534363168a49a26b4d/install_adfs.ps1) (Full script also included below)
3. Review & run the script

<script src="https://gist.github.com/bunchc/616446049eec0b2497051ca292e816b7.js"></script>

## How it works

Now that you've installed ADFS, let's examine what we actually ran.

The script first installs NuGet. This is used to install 3rd party modules.

```
Get-PackageProvider -Name NuGet -ForceBootstrap
Install-PackageProvider nuget -Force
```

Next, the PSPKI module is installed and loaded into the current shell. We use this module to create the self-signed SSL certificate needed to install ADFS:

```
Install-Module -Name PSPKI -Force
Import-Module -Name PSPKI
```

With the PSPKI module loaded, we can now create a self-signed SSL certificate, and install it into the Windows certificate store:

> __Note:__ Replace $fqdn with the FQDN for the ADFS host.

```
$selfSignedCert = New-SelfSignedCertificateEx -Subject "CN=$fqdn" `
    -ProviderName "Microsoft Enhanced RSA and AES Cryptographic Provider" `
    -KeyLength 2048 -FriendlyName 'OAFED SelfSigned' `
    -SignatureAlgorithm sha256 `
    -EKU "Server Authentication", "Client authentication" `
    -KeyUsage "KeyEncipherment, DigitalSignature" -Exportable `
    -StoreLocation "LocalMachine"

$certThumbprint = $selfSignedCert.Thumbprint
```

When creating the SSL certificate, we stored the thumbprint for the certificate in a variable so we can use it again when configuring ADFS.

The next several commands are responsible for installing and configuring the ADFS role:

```
$user  = "$env:USERDOMAIN\$env:USERNAME"
$credential = New-Object `
    -TypeName System.Management.Automation.PSCredential `
    -ArgumentList $user, $securePassword

Install-WindowsFeature -IncludeManagementTools -Name ADFS-Federation

Import-Module ADFS
Install-AdfsFarm -CertificateThumbprint $certThumbprint `
    -FederationServiceName $fqdn `
    -ServiceAccountCredential $credential
```

This final bit of the script grabs the username of the current user, and then creates a credential object for the service account that ADFS will use.

Next it installs the ADFS feature ```Install-WindowsFeature```.

The final bit imports the ADFS PowerShell module and configures ADFS to:

* Use the SSL certificate created earlier
* Assign a service name. (All the ADFS URLs use this)
* Assign the service account

## Test it out

You can validate that the ADFS role was installed and is running by browsing to ```https://<FQDN OF HOST>/adfs/fs/federationserverservice.asmx``` after the certificate warning you should get a bunch of XML.

You can also validate ADFS with the following PowerShell commands:

```
Install-Module ADFSDiagnostics -Force
Import-Module ADFSDiagnostics -Force

Test-AdfsServerHealth | ft Name,Result -AutoSize
```

If ADFS is working, you'll see something like this:

![ADFS is working](https://i.imgur.com/5Kfgunh.png)

## There is more!

The script provided creates a self-signed SSL certificate. While that will get you up and running in the lab, is not how you should deploy this in production. If you have a different certificate, say from an internal CA, or otherwise trusted CA, you can use it with this script. First ensure it is part of your Windows certificate store, then substitute your certificate's thumbprint in the following line and continue to use the script:

```
$certThumbprint = "Your SSL Cert Thumbprint here"
```

## Summary

In this post we used PowerShell to install, configure, and validate Active Directory Federation Services (ADFS). This in turn enables you to use Active Directory as an identity provider with all manner of 3rd party SSO tools.

## Resources

* [https://docs.microsoft.com/en-us/previous-versions/orphan-topics/ws.11/dn313041(v=ws.11)](https://docs.microsoft.com/en-us/previous-versions/orphan-topics/ws.11/dn313041(v=ws.11))
* [https://github.com/Crypt32/PSPKI](https://github.com/Crypt32/PSPKI)
* [https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/configure-corporate-dns-for-the-federation-service-and-drs](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/configure-corporate-dns-for-the-federation-service-and-drs)
* [https://blogs.msdn.microsoft.com/card/2010/06/21/a-quick-walkthrough-setting-up-ad-fs-saml-federation-with-a-shibboleth-sp/](https://blogs.msdn.microsoft.com/card/2010/06/21/a-quick-walkthrough-setting-up-ad-fs-saml-federation-with-a-shibboleth-sp/)
* [https://github.com/Azure-Samples/active-directory-lab-hybrid-adfs](https://github.com/Azure-Samples/active-directory-lab-hybrid-adfs)
* [https://blog.kloud.com.au/2016/09/27/automate-adfs-farm-installation-and-configuration/](https://blog.kloud.com.au/2016/09/27/automate-adfs-farm-installation-and-configuration/)
* [https://social.technet.microsoft.com/Forums/windows/en-US/24a33c1b-b802-4a60-8863-eaca22e8d52e/what-is-the-default-federation-metadata-url?forum=ADFS](https://social.technet.microsoft.com/Forums/windows/en-US/24a33c1b-b802-4a60-8863-eaca22e8d52e/what-is-the-default-federation-metadata-url?forum=ADFS)
* [https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/verify-that-a-federation-server-is-operational](https://docs.microsoft.com/en-us/windows-server/identity/ad-fs/deployment/verify-that-a-federation-server-is-operational)
* [https://gallery.technet.microsoft.com/scriptcenter/AD-FS-Diagnostics-Module-8269de31](https://gallery.technet.microsoft.com/scriptcenter/AD-FS-Diagnostics-Module-8269de31)
* 
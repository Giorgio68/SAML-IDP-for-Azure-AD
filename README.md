# SAML IDP for Azure AD
_Federate an Azure AD domain using a SAML Identity Provider._

## Setting up the identity provider

When adding the service provider to your IDP, be sure to include these parameters:

ACS URL: `https://login.microsoftonline.com/login.srf` (this requires a `POST` request)

Entity ID: `urn:federation:MicrosoftOnline`

Additionally, the response should be signed, and the Name ID should be in format: `persistent`

## Setting up Azure AD
### Requirements

Your domain has to be verified in Azure AD before you can continue with this process.

You will also need to install the powershell module `MSOnline`:

Start a powershell console as an administrator and enter:
```
Install-Module MSOnline
```
When asked if you're sure you want to install this module press `Y`, then wait for it to finish.

Note: if the domain you want to federate is a child domain, you will either need to federate its parent domain or change its status to be a root domain. For additional information, you can follow [this guide.](https://docs.microsoft.com/en-us/azure/active-directory/enterprise-users/domains-verify-custom-subdomain)

### Configuring the domain's federation status

Open a powershell console and type: 
```
Connect-MsolService
```
Then enter your Azure AD credentials (Note: you need to be either a `Global Administrator` or a `Domain Name Administrator` to federate a domain).

Now you can edit the sample `samlp-config.xml` file with values for your IDP:

- Change `FederationBrandName` to an appropriate name
- Change `IssuerUri` to the `Entity ID` of your IDP
- Change `PassiveLogOnUri` to the SSO url of your IDP (i.e. where azure will send the login request)
- Change `LogOffUri` to the logoff url of your IDP
- Copy paste your certificate to `SigningCertificate` (make sure there are no spaces and it's one line)

Then move to the directory the file is stored in and import it into powershell:
```
cd C:\fakepath\
$idp = Import-Clixml samlp-config.xml
```

And change the domain's status as federated:
```
Set-MsolDomainAuthentication -DomainName "{your-domain}" -FederationBrandName $idp.FederationBrandName -Authentication Federated -PassiveLogOnUri $idp.PassiveLogOnUri -ActiveLogOnUri $idp.ActiveLogonUri -SigningCertificate $idp.SigningCertificate -IssuerUri $idp.IssuerUri -LogOffUri $idp.LogOffUri -PreferredAuthenticationProtocol "SAMLP"
```

Once the domain has been federated, you can use this command to export the federation settings:
```
Get-MsolDomainFederationSettings -DomainName "{your-domain}" | Export-Clixml samlp-config.xml
```

Or this to see the federation settings in the console:
```
Get-MsolDomainFederationSettings -DomainName "{your-domain}" | Format-List *
```

## Credits
This guide is based on [IAmFrench](https://github.com/IAmFrench/)'s guide on [federating an Azure AD domain with GSuite.](https://github.com/IAmFrench/GSuite-as-identity-Provider-IdP-for-Office-365-or-Azure-Active-Directory)

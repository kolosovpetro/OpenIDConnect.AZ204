# OpenIDConnect Demo

[![Run Build and Test](https://github.com/kolosovpetro/OpenIDConnect.AZ204/actions/workflows/run-build-and-test-dotnet.yml/badge.svg)](https://github.com/kolosovpetro/OpenIDConnect.AZ204/actions/workflows/run-build-and-test-dotnet.yml)

Single and Multi tenant authorization using ASP NET Core and Active Directory

## Infrastructure provision

- Create resource group: `az group create --name "rg-open-id-connect" --location "centralus"`
- Create AD app registration: `az ad app create --display-name "webappoidc"`
- Get Application (client) ID from Azure portal: `d9334586-4f28-4277-9358-18d6bc025638`
- Get Directory (tenant) ID from Azure portal: `b40a105f-0643-4922-8e60-10fc1abf9c4b`

## Update Authentication blade inside app registration

Set the following parameters inside Authentication blade,
Click "Add platform" of type "Web"

And enter the following values:

- Redirect URI: `https://localhost:44302/signin-oidc`
- Front-channel logout URL: `https://localhost:44302/signout-oidc`
- ID Tokens (used for implicit and hybrid flows): `Checked`

As per screenshot below

## Create AD User via Azure Powershell

**Requires:**

- Update Windows Powershell as Administrator using: `Install-Module PSWindowsUpdate`
- Install [Azure PowerShell](https://docs.microsoft.com/en-us/powershell/azure/install-az-ps)
- Install as Powershell Administrator: `Install-Module AzureAD`

**Create User**

- Connect to AD: `Connect-AzureAD -TenantId "tenantId"`
- Define AD domain variable: `$aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name`
- Create password profile:
    - `$passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile`
    - `$passwordProfile.Password = 'Pa55w.rd1234'`
    - `$passwordProfile.ForceChangePasswordNextLogin = $false`
- Create new
  user: `New-AzureADUser -AccountEnabled $true -DisplayName 'aad_lab_user1' -PasswordProfile $passwordProfile -MailNickName 'aad_lab_user1' -UserPrincipalName "aad_lab_user1@$aadDomainName"`
- Print new user principal name: `(Get-AzureADUser -Filter "MailNickName eq 'aad_lab_user1'").UserPrincipalName`

## Create ASP NET Core MVC application

- Connect to AD: `Connect-AzureAD -TenantId "tenantId"`
- Define AD domain variable: `$aadDomainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name`
- Create application using client ID and tenant ID:
  `dotnet new mvc --auth SingleOrg --client-id "d9334586-4f28-4277-9358-18d6bc025638" --tenant-id "b40a105f-0643-4922-8e60-10fc1abf9c4b" --domain $aadDomainName`
- Update nuget packages:
    - `Microsoft.AspNetCore.Authentication.JwtBearer`
    - `Microsoft.AspNetCore.Authentication.OpenIdConnect`
    - `Microsoft.Identity.Web`
    - `Microsoft.Identity.Web.UI`


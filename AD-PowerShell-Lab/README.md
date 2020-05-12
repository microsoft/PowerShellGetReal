# Create a small on-premises lab environment for learning PowerShell

## Quick Start

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2FPowerShellGetReal%2Fmaster%2FAD-PowerShell-Lab%2Fdeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

## Details

Deploys the following infrastructure:

* Virtual Network
  * 1 subnet
* 1 Domain Controller
  * DSC installs AD
* Any **#** of Windows Server Machines
  * Joined to "OU=Servers,OU=Production,DC=contoso,DC=com"
* Any **#** of Windows Client Machines
  * Joined to "OU=Workstations,OU=Production,DC=contoso,DC=com"

## Notes

* All VMs deployed are the same size.

## Lab Walkthrough

* [Lab Overview](./Labs/readme.md)

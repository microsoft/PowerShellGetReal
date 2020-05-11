# Create a small on-premises lab environment for learning PowerShell

## Quick Start

<a href="https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2Fmicrosoft%2FPowerShellGetRealChalktalk%2Fmaster%2FAD-PowerShell-Lab%2Fdeploy.json" target="_blank"><img src="http://azuredeploy.net/deploybutton.png"/></a>

## Details

* Deploys the following infrastructure:
* Virtual Network
  * 1 subnet
* 1 Domain Controller
  * DSC installs AD
* 1 File Server
* **X** (Configurable) # of Windows Client Machines

## Notes

* One VM size is specified for all VMs

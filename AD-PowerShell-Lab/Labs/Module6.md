# $Automation.InvokeAll('GetReal')

## 6.1 Examine AD Objects

>**Note:** AD DS and GPMC RSAT tools are required to complete this section:

* Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
* Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0

1. Run the following commands

    ```PowerShell
    Get-ADUser –filter {samaccountname –eq $env:UserName –properties * }
    ```

    ```PowerShell
    Get-ADUser –filter {objectclass –eq “user”} –properties * | Select samaccountname,passwordneverexpires,enabled
    ```

    ```PowerShell
    (Get-ADUser -Identity User1<YourInitials> -Properties memberof | Select-Object MemberOf).memberof
    ```

    >**Extra:** Using AD users and Computers **DSA.msc**, review the groups your user account is a **Member Of.** Does the PoSH output **MATCH?**

2. Now try this:

    ```PowerShell
    Get-ADUser –filter * -searchbase “OU=PRODUCTION,DC=DOMAIN,DC=LAB”
    ```

    >**Question:** What CmdLet can be used to pull all the computers within the same OU?

    ```PowerShell
    Get-ADComputer –filter * -searchbase “DC=DOMAIN,DC=LAB” –properties passwordlastset | Export-Csv pcpwdlist.csv

    Get-ADComputer –filter * -properties * | Select-Object name, operatingsystem, operatingsystemservicepack | Out-GridView -title "Windows OS Inventory"


    $oneyearago = (Get-Date).addYears(-1) <enter>
    Get-ADComputer –Filter {lastlogontimestamp –lt $oneyearago} | Out-File c:\scripts\stalePCs.txt
    ```

## 6.2 Group Information

1. Now let's shift gears and look at AD Group information

    ```PowerShell
    Get-ADGroup  -filter {(cn –like  “*Admins*”)} –searchbase “CN=Users,DC=domain,DC=lab” | ForEach-Object {Get-ADGroupMember –Identity $_.samaccountname –recursive} | Export-Csv –path C:\scripts\Adminmembers.csv
    Get-ADGroup –Filter {GroupCategory –eq  “Security” –and groupScope –eq “Universal”}
    Get-ADGroupMember “Domain Admins” –recursive | select samaccountname,name | Format-Table –autosize
    ```

## 6.3 Automating NIC settings inventory

```PowerShell
Invoke-Command -computername <name> -scriptblock {Get-WmiObject win32_networkadapterconfiguration | where {$_.IPEnabled -eq "True"} | Select-Object ipaddress,defaultipgateway,ipsubnet,dnsserversearchorder}
```

## 6.4 Query Remote Registry

```PowerShell
Invoke-Command  -computername ws2008r2-dc01,ws2008r2-srv01,win7-01 {Get-ItemProperty "HKLM:\Software\Microsoft\Virtual Machine\Auto" | select integrationservicesversion} | Out-GridView
```

## 6.5 Check for Hotfix(es)

1. Develop some code to check hotfixes across computers as well as search for a specific hotfix

>**Note:** See if you can accomplish this on your own, if not, use some of the notes below:

```PowerShell
Get-Hotfix
Get-Hotfix | Out-GridView
Get-Hotfix –computername <name> | ConvertTo-Html > c:\scripts\patches.html
Get-Hotfix –computername <name> | ConvertTo-Html –cssuri sample.css > c:\scripts\patches.html
Get-HotFix –computername <name> | Export-Clixml C:\scripts\Patchlist.xml
#Then type:
Import-Clixml C:\scripts\Patchlist.xml | Out-GridView
```

## 6.6 Collect Remote MachineInfo

1. Develop some code to collect the following information:

    * Operating System Version
    * Operating System Build
    * Manufacturer
    * Model
    * Number of Processors
    * Number of Cores (logical)
    * Total Physical RAM
    * CPU Architecture

    >Hint:
    >Use PowerShell ISE to develop this code
    >Use Get-CimClass -ClassName Win32_* to help identify the ClassNames that can be used to gather the above information.
    After identifying all the ClassNames to be used, setup some code to create a persistent session to a remote computer.

    ```PowerShell
    $computer = ‘<Name>’
    $option = New-CimSessionOption -Protocol Wsman
    $sess_params = @{'Computername'=$computer
                    'SessionOption'=$option
                    }
    $Session = New-CimSession @sess_params
    ```

    Now develop some code to query the remote machine over the new CIM session, adding ALL information to Different variables.
    For Example:

    ```PowerShell
    $os_params = @{'ClassName'='Win32_OperatingSystem'
                'Cimsession'=$session}
    $os = Get-CimInstance @os_params
    ```

    Then, create a hashtable with the information collected from the remote machine. Use the below as a starting point and fill in the information after the = sign with the relevant information from the variables assigned in the previous step.

    ```PowerShell
    $Props = @{
            ComputerName = $computer
            OSVersion    =
            OSBuild      =
            Manufacturer =
            Model        =
            Processors   =
            Cores        =
            RAM          =
            Architecture =
            }
    ```

After populating the hashtable, create a new PSObject and output the object to the host.
Hint: Get-Help New-Object

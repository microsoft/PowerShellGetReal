# $Automation.InvokeAll('GetReal')

## 6.1 Examine AD Objects

>**Note:** AD DS and GPMC RSAT tools are required to complete this section:

* Rsat.ActiveDirectory.DS-LDS.Tools~~~~0.0.1.0
* Rsat.GroupPolicy.Management.Tools~~~~0.0.1.0

1. Run the following commands

    ```PowerShell
    Get-ADUser -Filter "samaccountname -eq '$env:USERNAME'" -Properties *
    ```

    ```PowerShell
    Get-ADUser -Filter "objectclass -eq 'user'" -Properties * | Select samaccountname,passwordneverexpires,enabled
    ```

    Now that we know how to get all user objects with and select specific attributes, we can take that a step further and filter on
    user objects that are enabled with a password that is set to never expire.

    For example:

    ```PowerShell
    Get-ADUser -Filter "objectclass -eq 'user'" -Properties * | Where-Object {($_.passwordneverexpires -eq 'True') -and ($_.enabled -eq 'True')} | select -ExpandProperty name
    ```

    >**Note:** After, we could generate reports for further review or layer in further automation tasks such as disabling accounts not authorized in this configuration.

    Now let's look at how to use dot notation to expand group membership

    ```PowerShell
    (Get-ADUser -Identity $env:username -Properties memberof | Select-Object MemberOf).memberof
    ```

    >**Extra:** Using AD users and Computers **DSA.msc**, review the groups your user account is a **Member Of.** Does the PowerShell output **MATCH?**

2. Now try this:

    ```PowerShell
    $RootDSE = (Get-ADDomain).distinguishedname
    Get-ADUser -Filter * -SearchBase "OU=Production,$RootDSE"
    ```

    In the above command using the **SearchBase** parameter we found all users in a specific OU.

    >**Question:** What changes need to be made to the **SEARCHBASE** parameter below in order to get a list of all computers in a the same OU?

    ```PowerShell
    Get-ADComputer –filter * -searchbase “DC=DOMAIN,DC=LAB” –properties passwordlastset | Export-Csv pcpwdlist.csv
    # or
    Get-ADComputer –filter * -properties * | Select-Object name, operatingsystem, operatingsystemservicepack | Out-GridView -title "Windows OS Inventory"
    ```

3. How about finding stale computers?

    ```PowerShell
    $oneyearago = (Get-Date).addYears(-1)
    Get-ADComputer –Filter {lastlogontimestamp –lt $oneyearago}
    ```

    >**Note:** This lab environment doesn't have any stale computers, so no information will return.

## 6.2 Group Information

1. Now let's shift gears and look at AD Group information

    >**Note:**A prime activity might be to **MONITOR** well known group membership. Once way you could do that is by doing the following.

    ```PowerShell
    #Build an array that contains all the well known groups
    $WellKnownGroups = @('Enterprise Admins'
    'Domain Admins'
    'Schema Admins'
    'Administrators'
    'Account Operators'
    'Backup Operators'
    'Print Operators'
    'Server Operators'
    'Domain Controllers'
    'Read-only Domain Controllers'
    'Group Policy Creator Owners'
    'Cryptographic Operators'
    'Distributed COM Users'
    )
    #Find all the groups within your domain
    $ADWKG = $WellKnownGroups | Get-ADGroup
    #Loop through all the users and computers within those groups
    $MembersOfWKG = foreach ($WKG in $ADWKG) {
        #create a filter to create an IF / Else statement
        $filter = 'Domain Controllers', 'Read-only Domain Controllers'
        #if not a in the above well-known groups do this
        if (($WKG.Name -notin $filter)) {
            $users = Get-ADGroupMember -Identity $WKG -Recursive | Get-ADUser
            foreach ($user in $users){
                [pscustomobject]@{
                    Name = $user.name
                    LastLogon = [datetime]::FromFileTime($user.lastLogonTimestamp)
                    Group = $WKG.Name
                }
            }
        #If the filter returned false then do this
        } else {
            $Computers = Get-ADGroupMember -Identity $WKG -Recursive | Get-ADComputer
            foreach ($Computer in $Computers){
                [pscustomobject]@{
                    Name = $Computer.name
                    LastLogon = [datetime]::FromFileTime($Computer.lastLogonTimestamp)
                    Group = $WKG.Name
                }
            }
        }
    }
    #Now we have all users in a variable called $MembersOfWKG
    ```

    Now that we have all users contained in a variable we could then build a report

    ```PowerShell
    $MembersOfWKG | ConvertTo-Html | Out-File $env:userprofile\Desktop\WellKnownGroup_Report.html
    ```

## 6.3 Automating NIC settings inventory

```PowerShell
Invoke-Command -computername File1 -scriptblock {Get-WmiObject win32_networkadapterconfiguration | where {$_.IPEnabled -eq "True"} | Select-Object ipaddress,defaultipgateway,ipsubnet,dnsserversearchorder}
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

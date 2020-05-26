# Cmdlets to Scripts

## 3.1 System Management with WMI

1. Use the “Win32_BIOS” WMI class to list details of the BIOS on a list of  systems from a txt file (examine the syntax of the file):

    ```PowerShell
    Get-Content $env:SystemDrive\Labfiles\computers.txt  | Foreach-Object {Get-WmiObject -computername $_ win32_bios}
    ```

2. Now, use a WMI class to list details of Windows.  

    Format the output in a table containing the following details:

    | Machine Name | Operating System Version Number | Operating System Architecture | Installation Date|
    |--------------|---------------------------------|-------------------------------|------------------|
    |The table headings | should be listed above and the returned | data will be populated in these  | rows of the table|

    >**Note:** Use this script codeblock to help **FIND** the WMI class that contains the information we are looking for.

    ```PowerShell
    $allWMI = Get-WmiObject -List
    $props = ForEach ($wmi in $allWMI) {
        [pscustomobject]@{
            Name       = $wmi.name
            Properties = "$($wmi.properties.name)"
        }
    }
    $props | Out-GridView
    ```

    Now that we have a listing of WMI Objects in a gridview. How can we sort to determine an object that has the data we need?

    >**For Example:**
    ![Out-GridView](./src/03-01-02-Out-GridView.png)

    ```PowerShell
    Get-WMIObject Win32_OperatingSystem | Format-Table PSComputerName, Version, OSArchitecture, InstallDate
    ```

3. How about converting property values displayed in DMTF to be human readable?

    In the last example, the WMI class **Win32_OperatingSystem** has a property called LastBootUpTime

    When using the **Get-WmiObject** the output of that property is not in a familiar time standard format.

    ```PowerShell
    (gwmi win32_OperatingSystem).lastbootuptime
    20200423212428.513878-240
    ```

    >How can we convert this to something more useful? Let's walk through this together

     PowerShell's object oriented language is built on top of .NET which also has a Class called **[ManagementDateTimeConverter]**
     within the **SystemManagement** namespace. Perhaps we can use this as a means to convert the time. *Let's Try!*

    ```PowerShell
    #First let's see what kind of Methods [ManagementDateTimeConverter] has available
    [System.Management.ManagementDateTimeConverter] | gm -s

    TypeName: System.Management.ManagementDateTimeConverter

    Name               MemberType Definition
    ----               ---------- ----------
    Equals             Method     static bool Equals(System.Object objA, System.Object objB)
    ReferenceEquals    Method     static bool ReferenceEquals(System.Object objA, System.Object objB)
    ToDateTime         Method     static datetime ToDateTime(string dmtfDate)
    ToDmtfDateTime     Method     static string ToDmtfDateTime(datetime date)
    ToDmtfTimeInterval Method     static string ToDmtfTimeInterval(timespan timespan)
    ToTimeSpan         Method     static timespan ToTimeSpan(string dmtfTimespan)

    ## Notice this has a ToDatetime static method. Let's see if we can use that!

    [System.Management.ManagementDateTimeConverter]::ToDateTime((gwmi win32_operatingsystem).lastbootuptime)
    Thursday, April 23, 2020 9:24:28 PM
    ```

    >**Viola!** We now know when the last time our machine powered on. Looks like this machine is due to reboot soon. Patch Tuesday!

## 3.2 Doing more with Get-Content

1. Often another data source will have information about a task we need to complete.

    One of those data sources is simply a text file. How can we read in the content of a file and then accomplish a task?

    Using **Get-Content** of course!

    The following example would most likely be run on a routine basis by Active Directory Admins.

    ```PowerShell
    $root = New-Object System.DirectoryServices.DirectoryEntry
    Get-content C:\labfiles\StaleComputers\StalePCs-<yourlogonname>.txt  | ForEach-Object { Move-ADObject -identity $_ -targetpath "OU=Stale,OU=PCs,OU=Production,$($root.distinguishedName)" }
    # Example
    $root = New-Object System.DirectoryServices.DirectoryEntry
    Get-Content C:\labfiles\StaleComputers\StalePCs-POSHUser-0.txt | ForEach-Object {Get-ADComputer $_ | Move-ADObject -TargetPath "OU=STALE,OU=PCs,OU=PRODUCTION,$($root.distinguishedName)”}
    ```

## 3.3 Moving to functions

Let's close out this module with a scenario intended to step from running simple CmdLets to a Script and then finally a function.

>**Note:** Some of the concepts here are not covered in the previous modules. Please see the following concept articles.

```Powershell
Get-Help about_Functions_Advanced_Methods
Get-Help about_Functions_Advanced_Parameters
Get-Help about_If
Get-Help about_Try_Catch_Finally
```

Let’s say within your environment you have an application service that is absolutely critical to be running at certain times for the business to maintain revenue. In the past, during business critical windows, the application has gone offline due to the underlying service being **Stopped**. Your manager has asked for your help to monitor the service and produce a log if the service is still running or not. Additionally, if it is not running provide an option to start the service.

You decide to create this solution in **PowerShell**.

Your goal is to create something that can be completely automated. However, you first need to do some investigation.

Initially, you write down all the tasks you want to accomplish.

>**Note:** ***Try this without looking ahead. After jotting down your own thoughts checkout our take.***

1. Able to specify **how often** the service is checked
2. Monitor the service for the entire duration of the critical window: **varying amount of time**
3. Specify a directory **path** for the log file
4. Option to not output to a log file.
5. Option to **Start** the service

While starting out the first thing to do is consider how we might provide a method for checking a service over a variable amount of time.

Let's look through some of the PowerShell about concept articles!

```PowerShell
Get-Help about_While
Get-Help about_Do
Get-Help about_for
```

Out of those 3 help files is there any one in particle that looks promising to achieve our goal?

>**Note:** Technically all of these could be used, with the right setup. But for our demonstration we will use the **Do {} Until ()** loop which is designed to run a block of code **first**, then **test** a condition. The statement block will run repeatedly until the condition statement evaluates to **true**.

The next thing we need to determine is how can we use this loop based on a time. For that we need to look to see if there are any cmdlets available that will display the date/time.

>**Hint:** Get-Help **Date*

Now that we have all the basics to build our Do loop, try and focus on building something that will query our critical service. While there are other requirements to complete this project for our manager, let’s try to tackle one objective at a time.

>***See if you can come up with some code on your own that will query a service until the specified time***  
>**Hint:** Use one of the methods from Get-Date to add time that will then be evaluated in the Until condition statement.

[**Here is our first attempt**](./answer/3.3.1_Do_Loop.md)

At this point we have come up with a way to query a service for a specified amount of time. Now how can we integrate some of the other bits of information?

>**Try** to come up with a way to integrate the other requirements.

[**Our take on a complete solution**](./answer/3.3.1_Do_Loop_Function.md)

To test out our suggested solution do the following:

1. Copy and paste the [function](./answer/3.3.1_Do_Loop_Function.md) into the ISE
2. Save the function as a ps1 file within your profile
3. Open an **Elevated** PowerShell console
4. Change directories to the root of your profile

    ```PowerShell
    Set-Location $Home
    ```

5. Dot source the script to import the function into your current scope

    ```PowerShell
    . $Home\PathToWhereYouSavedTheScript.ps1
    ```

6. Execute the function by running the following

    ```PowerShell
    Test-CriticalService -ServiceName WinRM -TimeEnd 2 -PauseTimer 5 -StartService -Verbose
    ```

7. Take a look at the log which will be at the root of your profile if you had executed **Step 4**

    ```PowerShell
    notepad $home\WinRM #Use tab completion to determine the rest of the file name
    ```

8. Finally lets use the Get-Help cmdlet to find out some information about this function

    ```PowerShell
    Get-Help Test-CriticalService -Full
    ```

    Notice how PowerShell displays the syntax information. It is able to do this based on the Param block.

    Also look at the specific parameters. For all parameters that accept input, the type of input is displayed. In this case **String** or **int**

Hopefully, this was an interesting first look at how to go from cmdlets to full scripts and tools. Toolmaking using PowerShell takes time to grasp completely. Use this as a starting point and I hope what you find is that the information to learn how to create useful tools is all contained within the PowerShell concept help! ***Perhaps after reading through some you will find ways to improve this example solution!*** There are many ways it could be improved.

# PowerShell Pipeline

## 2.1 Working with tables

1. Create a table of all the running processes:

    ```PowerShell
    Get-Process | Format-Table –AutoSize
    ```

    >**Question:** True or False?  The table contains all the properties of the processes.  
    >**Question:** How could you determine what other properties exist?  
    >**Challenge:** Create a **Pipeline** that displays just the process names and their PID.
    >**Question:** What syntax is required to restrict the table to displaying only processes starting or ending with “s”?

    **Hint:** Look back at exercise 1-7

    ```PowerShell
    Get-Process -Name s*,*s | Format-Table
    ```

2. How about creating a table for all running processes with a Working Set of 30Mb or more:

    ```PowerShell
    Get-Process | Where-Object{$_.ws -ge 30Mb} | Format-Table Name,ID,WorkingSet,VirtualMemorySize –AutoSize
    ```

## 2.2 More Pipelines, Formatting, and Practice

1. Create a table of the OS version and other information.  

    ```PowerShell
    # Review the output of each command below and observe the differences in output as it builds:
    Get-WmiObject win32_operatingsystem
    Get-WmiObject win32_operatingsystem | Format-list *
    Get-WmiObject –Computername File1 win32_operatingsystem | Format-Table –property caption,version,csname –Auto
    ```

    >**Challenge:** Edit the code so that the first column in the table shows the **ComputerName**

2. Explore some PowerShell code to search the System event log for event ID 1074 (which indicates a system reboot), and display the results in a table.

    ```PowerShell
    Get-Eventlog –Computername File1 -log system |
    Where-Object {$_.eventid –eq ‘1074’} |
    Format-Table machinename, username, timegenerated –autosize
    ```

    >**Question:** Are there any other useful events you could scrape from the event logs with this command?
    >For example: BSOD

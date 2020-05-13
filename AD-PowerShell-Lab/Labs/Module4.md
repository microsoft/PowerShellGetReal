# PowerShell Remoting

## 4.1 Working with remote commands

1. Based on the command below, build a one-liner that runs displaying the results in a table, with the target computer name listed first:

    ```PowerShell
    Get-WMIObject "Win32_LogicalDisk" | Where-Object {$_.DriveType -eq '3'}
    ```

    >**Note:** As usual there is more than one way to do this :-)

    Checkout the Microsoft Docs page for more information on DriveTypes

    [WMI Drive Types](https://docs.microsoft.com/en-us/windows/win32/cimwin32prov/win32-logicaldisk#properties)

## 4.2 Working with remote sessions

1. First verify if remoting is enabled (The following command must be run as admin):

    ```PowerShell
    Get-PSSessionConfiguration
    ```

    >Move on to Step 2 only after the first command completes successfully.

2. Now run

    ```PowerShell
    Get-NetFirewallRule -PolicyStore activeStore | where {($_.Enabled -eq 'True') -and ($_.name -like "WinRM*")}
    ```

3. Now connect to a remote machine:

    Open a new session to your one of the other client machines.  Execute some commands in the session that give machine-specific output (e.g. “ipconfig”, “hostname”, etc).

    ```PowerShell
    New-PSSession
    Get-PSSession
    Enter-PSSession -Id <"ID number from get-pssession">
    Exit-PSSession
    Remove-PSSession
    ```

4. Next launch notepad.exe in a remote session

    >**Question:** Do you think logged on users will see NOTEPAD launch?  Why or why not?

5. Leverage PowerShell to discover a way to end the notepad process you launched above.

    >**Note:** Do **NOT** use CTRL-C or Task Manager, instead leverage what you’ve learned during this class.  Only read the hints below if you are stuck…

    >**Hint:**  You’ll need to leave the ‘hung’ session open and launch a new PoSH window.

    >**2nd Hint:**  You’ll need to open a second remote session to your neighbor’s PC.

    >**3rd Hint:**  You’ll need to leverage the cmdlet used in Lesson 2.11.

    >**4th Hint:**  If you need another hint you probably need to get-help!

# Fundamentals

## 1.1 PowerShell Versions and Transcripts

1. Execute these cmdlets using PowerShell ISE, one at a time, and review the PowerShell version details:

    ```PowerShell
    Get-Host
    $PSVersionTable
    $PSVersionTable.PSVersion
    ```

2. **Sneak Peak** explore the output of the following to get a hint of what you can easily accomplish with just a sprinkle of PowerShell:

    ```PowerShell
    Get-WmiObject –computername file1 `
    win32_operatingsystem  | Select-Object pscomputername,caption,installdate,`
    lastbootuptime | Format-table -auto
    ```

3. Open the PowerShell console and execute the cmdlet:

    ```PowerShell
    Start-Transcript $env:USERPROFILE\Desktop
    ```

4. Execute these cmdlets, one after the other:

    ```PowerShell
    Get-Culture
    Get-PSdrive
    Get-Host
    ```

    >**Note:** Don’t worry about what these cmdlets are used for, it’s not important for this exercise

5. Examine the contents of the **transcript.txt**

6. Stop the Transcript

    ```PowerShell
    Stop-Transcript
    ```

    **Question:** What does this do?

## 1.2 Accessing Your PowerShell History

1. Clear the host

    ```PowerShell
    Clear-Host
    ```

2. Now execute the following commands, reviewing each one at a time

    ```PowerShell
    Get-Process
    Get-Service
    Get-Alias
    Get-Command
    Get-Host
    ```

3. Execute this command:

    ```PowerShell
    Get-History
    ```

    What is displayed is a historical record of the commands recently run in the PowerShell host.

4. Now execute this command:

    ```PowerShell
    Get-History | select -ExpandProperty commandline | Out-GridView -OutputMode Multiple | Invoke-Expression
    ```

    >Compare this with the output of Get-History. This provides the same information but provides the ability to rerun

## 1.3 Using the Built-In Help System

1. Execute the command:

    ```PowerShell
    Get-Help #choose N if prompted to update
    Get-Help -ShowWindow
    ```

    >Use the window and examine the parameters Get-Help has

2. Update the help on your system

    >Ensure you are running PowerShell **elevated**

    ```PowerShell
    Update-Help –sourcepath \\file1\PoshHelp -force
    ```

## 1.4: Using Get-Command

1. Use *Get-Command* to list all PowerShell and external discoverable commands

    ```PowerShell
    Get-Command
    ```

2. Now execute these command:

    ```PowerShell
    Get-Command –verb w*
    Get-Command –noun A*
    ```

    >**Question:** What does this command seem to do?

## 1.5: Using Get-Member and the Pipeline

1. Use the Get-Member command to explore the properties available when using some common cmdlets:

    ```PowerShell
    Get-Process
    # Review the output and then execute…
    Get-Process | Get-Member
    ```

    View the members of another cmdlet

    ```PowerShell
    Get-Date
    # Review the output and then execute…
    Get-Date | Get-Member
    ```

    >**Note:** Take compare the Typename information from Get-Process versus Get-Date
    >**Question:** Are they the same? How come?

## 1.6: An Alias by any other name

1. Explore aliases:

    ```PowerShell
    Get-Alias
    Get-Alias –name s*
    Get-Alias –definition get-service
    ```

2. Create an alias for the Get-Help cmdlet:

    ```PowerShell
    New-Alias PleaseGetMeSomeHelp Get-Help
    PleaseGetMeSomeHelp # Enter
    New-Alias gh Get-Help
    gh # Enter
    ```

3. Close your PowerShell console, open another, then type this command:

    ```PowerShell
    gh
    ```

    >**Question:** What happens? Why?

### 1.6.1 Filtering Challenge

**Challenge 1:** Figure out the syntax required to list all aliases starting with “s”, “m” or “t”.

**Challenge 2:** Figure out how to list all aliases **EXCEPT** those starting with “s”, “m” or “t”.

>**Hint:** Checkout the parameters of Get-Alias using the built-in Help System

```PowerShell
Get-Help Get-Alias -Parameter Name
-Name <String[]> # Does the [] following the word "String" mean anything?
```

## 1.7 Fundamentals.Blitz()

1. Explore common CMD tools running from the PowerShell host

    ```cmd
    Ping
    IPConfig
    cd "c:\"
    ```

2. Then try:

    ```PowerShell
    Get-Command CD
    #Now run the named cmdlet
    Get-Command Set-Location
    ```

    >**Notice** the difference between an alias and the named command?

3. Now Try:

    ```PowerShell
    Get-Command set-location | Format-list
    ```

    >**Question:** Is there a pre-defined alias for Format-List? How can you find out?

4. Next explore the cmdlets

    ```PowerShell
    Get-Command –commandtype  cmdlet
    ```

    >**Notice** the various cmdlets and their associated modules

5. Comparing Get-ChildItem and Dir

    ```PowerShell
    Get-Childitem c:\
    # Then type
    dir c:\
    ```

    >Compare the output of these commands. How do they compare and more importantly **WHY**?

6. Lets mess with a Date

    ```PowerShell
    $Now = Get-Date
    $Now.AddDays(-60)
    ```

    >**Question:** What does the **"$Now** text string represent?
    >Is there a cmdlet that can display all Automatic and current user-created stored values?

7. Moving on to the logs

    ```PowerShell
    Get-EventLog -List
    # Copy one of the log names such as Application
    Get-EventLog -LogName 'Application'
    ```

    >**Note:** Press **CTRL + C** to break

    ```PowerShell
    Get-EventLog  -LogName  Application  -Newest 10
    ```

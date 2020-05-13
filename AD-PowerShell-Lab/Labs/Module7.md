# Having Some Fun

## Make your computer talk using PowerShell and COM

>**Note:** This will not work when connected to an RDP session without sound redirection. Skip this item if you don’t have sound capability.

```PowerShell
$voice = New-Object –com SAPI.SPvoice
$voice.rate = 0
$voice.speak(“I’m letting my PowerShell skills do the talking!”)
or
$voice.speak(“C:\scripts\read_me.txt”, 5)
```

### Doing the same with .NET versus COM

```PowerShell
Add-Type -AssemblyName System.Speech
$speech=New-Object -TypeName System.Speech.Synthesis.SpeechSynthesizer
$speech.SpeakAsync(“Invoking speech using .NET”)
```

Of you can read in a text file:

```PowerShell
$SpeechText = gc -path C:\scripts\read_me.txt
$voice.speak($SpeechText)
```

## Creating a GUI with WinForms

Enter this PowerShell code in a script and execute it

```PowerShell
Add-Type –Assemblyname “System.Windows.Forms”
$form = new-object Windows.Forms.Form
$Form.Text = "PowerShell Lab"
$form.Height=300
$form.Width=300
$Button = new-object Windows.Forms.Button
$Button.Text = "Click Me!"
$Button.Top=50
$Button.Left=100
$Button.add_click({[System.Windows.Forms.MessageBox]::Show(“You created a window with PowerShell!”, "This is a message for you ...", 0, [System.Windows.Forms.MessageBoxicon]::Information)})
$form.controls.Add($button)
$form.Add_Shown({$form.Activate()})
$form.ShowDialog()
```

## How About a Challenge

### The Evil Object Challenge: The Greatest Storage Consumers

Search your user profile and find:

- The five file-extensions that in **SUM** (length) are the greatest storage consumers
- For each of those extensions, find the six largest files

*(optional)* Sort them by the month they were last accessed

Limitations:

- Your code during development may only be executed in PowerShell.exe (no ISE or VSCode or other smart-alec editor should help you).
- Storing Snippets in plain notepad is legal.
- You may not search the internet for help
- You may not ask human beings for help
- You may not use commands that are not part of the default PowerShell
- Your solution may not use `foreach` loops, though it may use `ForEach-Object`, if so desired.
- Your solution should `NOT` use more than two variable assignments

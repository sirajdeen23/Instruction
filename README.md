Standard Operating Procedure (SOP)Centralized Deployment Guide: 

Automatic
---------------------------------------------------------------------------
(
# 1. Define path in the user's personal AppData folder (No Admin required)
$dirPath = "$env:LOCALAPPDATA\LogonNotice"
$batPath = "$dirPath\LaunchNotice.bat"

# 2. Create the folder if it doesn't exist
if (!(Test-Path $dirPath)) {
    New-Item -ItemType Directory -Force -Path $dirPath | Out-Null
}

# 3. Create the batch file
$batContent = @"
@echo off
set "LOCAL_FILE=%temp%\notice.hta"
set "GITHUB_URL=https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta"

:: Added -UseBasicParsing to prevent PowerShell from freezing
powershell -Command "Invoke-WebRequest -Uri '%GITHUB_URL%' -OutFile '%LOCAL_FILE%' -UseBasicParsing"

start mshta.exe "%LOCAL_FILE%"
"@

Set-Content -Path $batPath -Value $batContent -Encoding ASCII

# 4. Create and register the Task Scheduler task for the CURRENT USER only
$taskName = "GitHub Logon Notice"

$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c `"$batPath`""
$trigger = New-ScheduledTaskTrigger -AtLogOn
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries

# Registers the task specifically for the currently logged-in user
Register-ScheduledTask -TaskName $taskName -Action $action -Trigger $trigger -Settings $settings -User $env:USERNAME -Force | Out-Null

Write-Host "Setup Complete! Installed for the current user without Admin rights." -ForegroundColor Green
)

---------------------------------------------------------


Method B:

Manual Setup SOP (Step-by-Step for Technicians)Best for: Non-domain PCs without automated tools.Create Directory:Create a folder at C:\ProgramData\LogonNotice\.Save Batch File:Save LaunchNotice.bat inside C:\ProgramData\LogonNotice\ with this content:DOS
----------------------------------------------------------------
@echo off
set "LOCAL_FILE=%temp%\notice.hta"
set "GITHUB_URL=https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta"

:: Added -UseBasicParsing to prevent PowerShell from freezing
powershell -Command "Invoke-WebRequest -Uri '%GITHUB_URL%' -OutFile '%LOCAL_FILE%' -UseBasicParsing"

start mshta.exe "%LOCAL_FILE%"

------------------------------------------------------------------------------------------------
Configure Task Scheduler:Open Task Scheduler $\rightarrow$ Click Create Task.General Tab: Name = GitHub Logon Notice. Select Run only when user is logged on.Triggers Tab: New $\rightarrow$ Begin the task: At log on.Actions Tab: New $\rightarrow$ Action: Start a program.Program: C:\ProgramData\LogonNotice\LaunchNotice.batConditions Tab: Uncheck Start the task only if the computer is on AC power.Click OK to save.3. How to Update Content for All UsersTo push a new message to every user across all machines:Edit the notice.hta file directly in your GitHub repository (sirajdeen23/Instruction).Commit the changes.Every user will automatically download and view the updated content the next time they log on to their computer.

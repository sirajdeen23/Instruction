Standard Operating Procedure (SOP)Centralized Deployment Guide: 
GitHub Logon Notice AppThis document outlines how to deploy the automated GitHub logon notice screen across multiple Windows machines in an organization or multi-user environment.1. Overview & ArchitectureSource File: [https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta](https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta)Local Storage Path: C:\ProgramData\LogonNotice\LaunchNotice.batTrigger: Windows Logon Event (Task Scheduler)Execution Flow: User logs on $\rightarrow$ Task Scheduler triggers batch file $\rightarrow$ Script downloads latest notice.hta from GitHub to %temp% $\rightarrow$ Launches via mshta.exe.2. Deployment MethodsChoose the deployment method that fits your network environment:Method A: One-Click PowerShell Deployment (Recommended)Best for: Manual setup across multiple PCs, or deployment via RMM / remote administration tools.Run this single PowerShell command as Administrator on each target machine. It automatically creates the necessary folder, builds the batch script with your GitHub URL, and registers the scheduled task for all users.

PowerShell
# Run in PowerShell as Administrator

(# 1. Define paths and GitHub URL
$dirPath = "C:\ProgramData\LogonNotice"
$batPath = "$dirPath\LaunchNotice.bat"
$githubUrl = "https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta"

# 2. Create local program directory
if (!(Test-Path $dirPath)) {
    New-Item -ItemType Directory -Force -Path $dirPath | Out-Null
}

# 3. Write the batch file content
$batContent = @"
@echo off
set "LOCAL_FILE=%temp%\notice.hta"
set "GITHUB_URL=$githubUrl"

:: Download latest HTA from GitHub
powershell -Command "Invoke-WebRequest -Uri '%GITHUB_URL%' -OutFile '%LOCAL_FILE%'"

:: Launch the HTA window
start mshta.exe "%LOCAL_FILE%"
"@

Set-Content -Path $batPath -Value $batContent -Encoding ASCII

# 4. Create Task Scheduler task for ALL users logging onto the machine
$taskName = "GitHub Logon Notice"
$action = New-ScheduledTaskAction -Execute "cmd.exe" -Argument "/c `"$batPath`""
$trigger = New-ScheduledTaskTrigger -AtLogOn
$settings = New-ScheduledTaskSettingsSet -AllowStartIfOnBatteries -DontStopIfGoingOnBatteries

Register-ScheduledTask -TaskName $taskName -Action $action -Trigger $trigger -Settings $settings -User "Users" -Force | Out-Null

Write-Host "Deployment completed successfully for task: $taskName" -ForegroundColor Green)

---------------------------------------------------------
Method B: Group Policy Object (GPO) DeploymentBest for: Active Directory Domain networks (Deploying to hundreds of domain-joined PCs).Open Group Policy Management (gpmc.msc) on your Domain Controller.Edit an existing GPO or create a new one targeted at your computers/users.Navigate to: Computer Configuration $\rightarrow$ Preferences $\rightarrow$ Control Panel Settings $\rightarrow$ Scheduled Tasks.Right-click $\rightarrow$ New $\rightarrow$ Scheduled Task (At least Windows 7).Configure the following settings:Action: CreateName: GitHub Logon NoticeUser Account: NT AUTHORITY\Authenticated Users (or run under current user logon)Triggers Tab: New Trigger $\rightarrow$ At log on $\rightarrow$ Any user.Actions Tab: New Action $\rightarrow$ Start a program:Program/script: powershell.exeAdd arguments: -WindowStyle Hidden -Command "Invoke-WebRequest -Uri '[https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta](https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta)' -OutFile '$env:TEMP\notice.hta'; start mshta.exe '$env:TEMP\notice.hta'"Apply the GPO to the target Organizational Unit (OU).


Method C:

Manual Setup SOP (Step-by-Step for Technicians)Best for: Non-domain PCs without automated tools.Create Directory:Create a folder at C:\ProgramData\LogonNotice\.Save Batch File:Save LaunchNotice.bat inside C:\ProgramData\LogonNotice\ with this content:DOS
----------------------------------------------------------------
@echo off
set "LOCAL_FILE=%temp%\notice.hta"
set "GITHUB_URL=https://raw.githubusercontent.com/sirajdeen23/Instruction/refs/heads/main/notice.hta"
powershell -Command "Invoke-WebRequest -Uri '%GITHUB_URL%' -OutFile '%LOCAL_FILE%'"
start mshta.exe "%LOCAL_FILE%"

------------------------------------------------------------------------------------------------
Configure Task Scheduler:Open Task Scheduler $\rightarrow$ Click Create Task.General Tab: Name = GitHub Logon Notice. Select Run only when user is logged on.Triggers Tab: New $\rightarrow$ Begin the task: At log on.Actions Tab: New $\rightarrow$ Action: Start a program.Program: C:\ProgramData\LogonNotice\LaunchNotice.batConditions Tab: Uncheck Start the task only if the computer is on AC power.Click OK to save.3. How to Update Content for All UsersTo push a new message to every user across all machines:Edit the notice.hta file directly in your GitHub repository (sirajdeen23/Instruction).Commit the changes.Every user will automatically download and view the updated content the next time they log on to their computer.

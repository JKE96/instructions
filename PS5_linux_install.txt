#Hi level notes
WSL seemed to not work great for me, so I went FULL ON WINDOWS. The following commands sets up the umtx jailbreak and sends a payload with python. I recommend running this locally. I have no idea if it's safe, so there is that, but it should be fine.

#0 First step, read this page and configure the PS5 settings
https://github.com/ps5-linux/ps5-linux-loader

#1 Download the Linux ISO, attempting to build it myself failed. Apparently that's better to do though
https://github.com/ps5-linux/ps5-linux-image/releases/tag/latest

#2 Windows (Balena Etcher):
Download Balena Etcher, select the .img file, select your USB drive, click Flash.

#3 Plug in the USB drive into your PS5
The following USB ports are supported for booting:
Front bottom Type-C port
Rear Type-A ports
The front top Type-A port is USB 2.0 which is slower and thus not recommended.

#4 Run the jailbreak after reading and running the comments below
Firmware 3.00-5.50
Clone via: git clone https://github.com/idlesauce/umtx2
Configure fakedns via dns.conf to point manuals.playstation.net to your PCs IP address
Run fake dns: sudo python fakedns.py -c dns.conf
In a different terminal, run HTTPS server: sudo python host.py
Go into PS5 advanced network settings and set primary DNS to your PCs IP address and leave secondary at 0.0.0.0
Go to user manual in settings and accept untrusted certificate prompt, run.
## For this step, take out the "sudo" as part of the command and instead run as admin in powershell. You will probably want to kill the windows network sharing stuff that WSL latches onto for port 53 which can cause chaos.
## Ensure wsl is shut down if it was ever started
wsl --shutdown

## Kill the active process and block the service from being able to start again
# 1. Block the service from being allowed to start at all
Set-Service -Name "SharedAccess" -StartupType Disabled
# 2. Kill the active process immediately
Get-WmiObject Win32_Service -Filter "Name='SharedAccess'" | ForEach-Object { Taskkill /F /PID $_.ProcessId }
## (If you want to turn the service back on, I do not recommend doing that until you have things running)
##Start-Service -Name "SharedAccess"
##Set-Service -Name "SharedAccess" -StartupType Manual

#5 Send the payload
Download the elf file from here: https://github.com/ps5-linux/ps5-linux-loader/releases/
Then install it on windows using this command and ensure the PS5 IP Address is correct. Go to IPV4 in PS5 network settings to find it.
python -c "import socket; s=socket.socket(); s.connect(('192.168.1.71', 9021)); s.sendall(open('ps5-linux-loader.elf', 'rb').read())"

##The PS5 will restart with the payload installed and ready to go.
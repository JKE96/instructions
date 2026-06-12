# PS5 Linux Loader and Jailbreak Guide

## High-Level Notes

WSL seemed to not work great for me, so I went **FULL ON WINDOWS**. The following commands set up the umtx jailbreak and send a payload with Python. I recommend running this locally. I have no idea if it's safe, so there is that, but it should be fine.

## 0. First Step: Configure PS5 Settings

Read this page and configure the PS5 settings:
`https://github.com/ps5-linux/ps5-linux-loader`

## 1. Download the Linux ISO

Attempting to build it myself failed. Apparently that's better to do though. Go to this location and download the latest release image:
https://github.com/ps5-linux/ps5-linux-image/releases/tag/latest

## 2. Windows (Balena Etcher)

Go to this location and download Balena Etcher:
https://balena.io

Once installed:
1. Select the `.img` file.
2. Select your USB drive.
3. Click **Flash**.

## 3. Plug the USB Drive into your PS5

The following USB ports are supported for booting:
* Front bottom Type-C port
* Rear Type-A ports

*Note: The front top Type-A port is USB 2.0, which is slower and thus not recommended.*

## 4. Run the Jailbreak

Read the comments below before running the setup. This applies to Firmware 3.00-5.50.

### Prerequisites (Windows PowerShell as Admin)
Take out the `sudo` from the repository commands and instead run your console as Administrator in PowerShell. You will probably want to kill the Windows network sharing stuff that WSL latches onto for port 53, which can cause chaos.

Ensure WSL is shut down if it was ever started:
```powershell
wsl --shutdown
```

Kill the active process and block the service from being able to start again:
```powershell
# 1. Block the service from being allowed to start at all
Set-Service -Name "SharedAccess" -StartupType Disabled

# 2. Kill the active process immediately
Get-WmiObject Win32_Service -Filter "Name='SharedAccess'" | ForEach-Object { Taskkill /F /PID \$_.ProcessId }
```

*(If you want to turn the service back on, I do not recommend doing that until you have things running)*
```powershell
# Start-Service -Name "SharedAccess"
# Set-Service -Name "SharedAccess" -StartupType Manual
```

### Repository Setup
Clone the repository:
```bash
git clone https://github.com/idlesauce/umtx2
```

1. Configure `fakedns` via `dns.conf` to point `manuals.playstation.net` to your PC's IP address.
2. Run fake DNS:
   ```bash
   python fakedns.py -c dns.conf
   ```
3. In a different terminal, run the HTTPS server:
   ```bash
   python host.py
   ```

### PS5 Configuration
1. Go into PS5 advanced network settings.
2. Set the **Primary DNS** to your PC's IP address.
3. Leave the **Secondary DNS** at `0.0.0.0`.
4. Go to the **User's Guide** in settings, accept the untrusted certificate prompt, and run.

## 5. Send the Payload

Go to this location and download the `.elf` file:
https://github.com/ps5-linux/ps5-linux-loader/releases/

Install it using Windows PowerShell/Command Prompt. Ensure the PS5 IP Address is correct (check IPv4 in PS5 network settings to find it):

```bash
python -c "import socket; s=socket.socket(); s.connect(('192.168.1.71', 9021)); s.sendall(open('ps5-linux-loader.elf', 'rb').read())"
```

The PS5 will restart with the payload installed and ready to go.
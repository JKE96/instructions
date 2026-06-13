# Ubuntu to Windows SMB File Sharing Guide

A complete step-by-step technical reference for creating a local network shared folder on Ubuntu and mounting it securely as a network drive on Windows.

---

## Phase 1: Ubuntu Configuration (Host)

### 1. Install Samba Utilities
Ensure the core Samba binaries and command-line management tools are installed:
```bash
sudo apt update && sudo apt install -y samba samba-common-bin smbclient
```

### 2. Create the Physical Directory
Create the underlying folder on the Linux filesystem and open global read/write/execute permissions so Windows can access and modify files inside it:
```bash
mkdir -p /home/$USER/ubuntuShare
sudo chmod -R 777 /home/$USER/ubuntuShare
```

### 3. Configure the Samba Share Daemon
I did not have to do this step
Open the Samba configuration file:
```bash
sudo nano /etc/samba/smb.conf
```

I also did not have to do this step
Scroll to the **`[global]`** section near the top and add compliance protocols for modern Windows clients:
```text
server min protocol = SMB2
server max protocol = SMB3
lanman auth = yes
ntlm auth = yes
```

Scroll to the **very bottom** of the file and append the explicit share definition path:
```text
[ubuntuShare]
    path = /home/YOUR_UBUNTU_USERNAME/ubuntuShare
    browsable = yes
    writable = yes
    read only = no
    guest ok = no
```
*(Replace `YOUR_UBUNTU_USERNAME` with your actual Ubuntu account username).*

### 4. Configure User Account and Password Complexity
Samba enforces secure credential rules. Create an account with a strong password (minimum 8 characters, letters, and numbers) and explicitly toggle its status to enabled:
```bash
# Add user and set strong password
sudo smbpasswd -a YOUR_UBUNTU_USERNAME

# Force enable the network user profile
sudo smbpasswd -e YOUR_UBUNTU_USERNAME
```

### 5. Restart Services
Restart the host daemons to apply all configuration changes:
```bash
sudo systemctl restart smbd nmbd
```

### 6. Retrieve Host Network Identity
Find the IP address assigned to your network card (look under the `inet` line of your active interface):
```bash
ip a
```

---

## Phase 2: Windows Connection (Client)

### 1. Map the Drive Letter
Run this command in the Windows Command Prompt to instantly bind the Ubuntu folder to the **Z:** drive using the isolated network domain workaround:
```cmd
net use Z: \\YOUR_UBUNTU_IP\ubuntuShare /user:WORKGROUP\YOUR_UBUNTU_USERNAME YOUR_STRONG_PASSWORD
```

---

## Phase 3: Verification (Optional)

To check what share resources the Ubuntu computer is actively broadcasting over your local network, run this command from Windows:
```cmd
net view \\YOUR_UBUNTU_IP
```


If for some reason you need to reset the user with samba such as with bad login requests you can run the following command
sudo smbpasswd -e YOUR_UBUNTU_USERNAME


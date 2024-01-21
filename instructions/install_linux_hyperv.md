# 1. [Setup HyperV](https://phoenixnap.com/kb/hyper-v-ubuntu)
1. [Enable virtualization in BIOS](https://www.virtualmetric.com/blog/how-to-enable-hardware-virtualization)
   1. Reboot system
   2. Go to BIOS/UEFI
   3. CPU or Advanced BIOS Settings
   4. Set Virtualization Enabled
2. Enable HyperV in Windows  
   On [Windows Home](https://www.makeuseof.com/install-hyper-v-windows-11-home/) it is needed to create ```run.bat``` file and run it with PowerShell as AdminWindows Home:
```   pushd "%~dp0"
dir /b %SystemRoot%\servicing\Packages\*Hyper-V*.mum >hv.txt
for /f %%i in ('findstr /i . hv.txt 2^>nul') do dism /online /norestart /add-package:"%SystemRoot%\servicing\Packages\%%i"
del hv.txt
Dism /online /enable-feature /featurename:Microsoft-Hyper-V -All /LimitAccess /ALL
pause
```

# 2. Install Virtual Machine
It is possible to use ```Quick create``` in Hyper-V, but I will list steps with manual installation.
1. Run [Hyper-V Manager](/images/install_hyperv/run_hypev.png) by searching in Start
2. [Click ```New/Virtual Machine```](/images/install_hyperv/new_vm.png)
3. Specify common VM settings step-by-step
   1. Set VM name and location
   2. Choose Generation 2 (for modern OSes, or read about differences)
   3. Set RAM
   4. Set Network: Default
   5. Set hard drive size
   6. Choose ISO with OS
4. [Disable Secure Boot in VM settings](/images/install_hyperv/disable_secureboot.png)
5. Run VM by double click on it and press ```Start```
6. Install OS (e.g. Ubuntu) as usual

# 3. Turn off autorun GUI after reboot
If you want to use VM only with VSCode, it is not nessesary to run GUI. So it possible to turn it off.
1. Edit ```/etc/default/grub``` as a root. Change lines to:
* ```GRUB_CMDLINE_LINUX_DEFAULT="quiet splash"``` -> ```GRUB_CMDLINE_LINUX_DEFAULT="text"```
* ```GRUB_TERMINAL=console```
2. Run ```sudo update-grub```
3. If your computer uses systemd, you must tell systemd to skip the default login GUI. Run:
```
sudo systemctl enable multi-user.target --force
sudo systemctl set-default multi-user.target
sudo reboot
```
4. Reboot ```sudo reboot```

In case you need to run gui use command: ```startx```

# 4. Setup SSH
### Install and setup autorun SSH:
```
sudo apt install ssh
sudo systemctl start ssh
sudo systemctl enable ssh
sudo systemctl status ssh
sudo ufw allow ssh
```

### Temporary allow access by password
Edit the ```/etc/ssh/sshd_config``` and modify or add the following line: ```PasswordAuthentication yes```. Restart ssh by: ```sudo systemctl restart ssh```  
Now it's possible to connect to VM from Windows [by command in PowerShell via password](/images/install_hyperv/ssh_pass.png):
```ssh user@domain```

### Login by key
1. Gen key in PowerShell: ```ssh-keygen.exe```  without passphrase
2. [Copy pub key to VM](/images/install_hyperv/ssh_gen.png). e.g. ```scp "YOUR_PATH\.ssh\vm_rsa.pub" user@domain:~/.ssh/vm_rsa.pub```
Don't forget to create ```mkdir ~/.ssh``` folder on VM before executing this command
3. Add pub-key to ```authorized_keys``` on VM:  
   ```cat ~/.ssh/vm_rsa.pub > ~/.ssh/authorized_keys```
4. Change rights on file and folder:  
   ```
   sudo chmod 600 ~/.ssh/authorized_keys
   sudo chmod 700 ~/.ssh
   ```
Connect to VM from Windows via key [by run in PowerShell](/images/install_hyperv/ssh_key_login.png):
```ssh user@domain -i "C:\Users\USER\.ssh\vm_rsa"```  
Now you can remove password auth.

*Possible problem with key on Windows - [change rights of key folder](https://superuser.com/questions/1296024/windows-sshpermissions-for-private-key-are-too-open)

# 5. [Setup VSCode](https://code.visualstudio.com/docs/remote/ssh-tutorial)
1. Download and install VSCode: [https://code.visualstudio.com/download](https://code.visualstudio.com/download)
2. [Install Remote - SSH extension](/images/install_hyperv/install_extension.png)
3. Optional (if you want to turn off VM from the internet):  
   Go to [Remote SSH extension settings](/images/install_hyperv/local_download.png): ```Ctrl+,```; Search ```Local Server Download```. Set to ```Always```
4. Go to [Remote-SSH: Open Configuration File](/images/install_hyperv/vscode_ssh.png). Fill:
```
Host Some_name
    HostName domain
    User user
    IdentityFile C:\\Users\\USER\\.ssh\\vm_rsa
```
5. Connet to VM by double-click on the created connection
6. Enjoy and use your VS Code

# Extra
## Copy files from host to VM
```Copy-VMFile -Name 'Ubuntu 22.04 LTS' -SourcePath 'C:\Temp\sources.list' -DestinationPath '/home/user/' -FileSource Host```


## Turn off automatic checkpoints
Right-click on the VM/```VM Settings```/```Managment```/```Checkpoints Standart```/```Use Automatic Checkpoints```

## [Install vm tools (turn on clipboard)](https://www.nakivo.com/blog/install-ubuntu-20-04-on-hyper-v-with-enhanced-session/)
Instruction in the next repo: ```git clone https://github.com/Hinara/linux-vm-tools.git```

## Setup static local IP for VM
1. Create an adapter by running next script in PowerShell:
```
New-VMSwitch -SwitchName "switch_for_statitc_ip" -SwitchType Internal
Get-NetAdapter #(note down ifIndex of the newly created switch as INDEX)
New-NetIPAddress -IPAddress 192.168.0.1 -PrefixLength 24 -InterfaceIndex <INDEX>
New-NetNat -Name NATnetworkForStaticIp -InternalIPInterfaceAddressPrefix 192.168.0.0/24
#This uses 192.168.0.0/24 as the subnet for the virtual switch, where 192.168.0.1 is the IP of the host, which acts as a gateway.
```
2. Go to VM settings. Add Hardware/Network Adapter 
3. Change created [Network Adapter for new interface](/images/install_hyperv/static_na.png)
4. [Set Network interface in VM: Static IP, e.g. 192.168.0.3/24](/images/install_hyperv/network_settings.png)
5. Now you can remove default switch from VM and use only lan

## Use sudo in linux without pass
1. Run ```sudo visudo```  
2. Add line: ```YOUR_USERNAME_HERE ALL=(ALL) NOPASSWD: ALL```  
3. Run ```sudo group: usermod -aG sudo YOUR_USERNAME_HERE```
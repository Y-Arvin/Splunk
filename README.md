# Splunk

## Objective

The Splunk project aims to use a controlled environment for performing brute force attacks for educational purposes ONLY. While monitoring systems to capture and analyze system events caused by the brute force attack. This hands-on experience is designed to allow monitoring of endpoint devices and ethical hacking practices in a safe and controlled setting. 

### Skills Learned

- Establishing cross-platform file sharing through virtual machine configurations
- Deployment of Splunk Enterprise on Ubuntu and Splunk Universal Forwarder on Windows 10
- Understanding data collection, indexing, and performing data query with real-time data in Splunk
- Utilizing Splunk for security information and event management
- Perform controlled brute force attacks against authentication mechanisms. 
- Use of automation to automate RDP brute force login attempts

### Tools Used

- Hypervisor Software: [Oracle VM VirtualBox](https://www.virtualbox.org/wiki/Downloads) Utilized for the creating, managing, and running virtual machines on a host system.
- Splunk Server: [Splunk Enterprise](https://www.splunk.com/en_us/download/splunk-enterprise.html) Handles data ingest and indexing.
- SIEM Tool: [Splunk Universal Forwarder](https://www.splunk.com/en_us/download/universal-forwarder.html) Used for data collection on an endpoint PC to a centralized dashboard for event analysis.
- System Monitor: [Sysmon](https://learn.microsoft.com/en-us/sysinternals/downloads/sysmon); [Olaf-Configuration](https://github.com/olafhartong/sysmon-modular/blob/master/sysmonconfig.xml) Specific configuration to monitor specific types of events. 
- Brute Force Tool: [Crowbar](https://www.kali.org/tools/crowbar/) Performs brute force during the penetration test, View telemetry on Splunk created by the brute force attack.

## Preconfigured network settings
Refer to [VirtualBox & Active Directory](https://github.com/Y-Arvin/VirtualBox-ActiveDirectory) for setting up the network that will be used. The first nine reference photos will be used as a guide for setting up the network. 

## Walkthrough
**Update Ubuntu Server & Create a shared folder** <br>
On the host machine go to the [Splunk website](https://www.splunk.com/en_us/download/splunk-enterprise.html) and download `Splunk Enterprise Linux .deb`<br>
On the Ubuntu Server, update all repositories and install VirtualBox dependencies.
```bash
$ sudo apt-get update && sudo apt-get upgrade -y
$ sudo apt-get install virtualbox-guest-additions-iso
$ sudo apt-get install virtualbox-guest-utils
```
On the menubar of the Virtualbox, click the devices tab click shared folders>share folder settings. Click the green plus icon on the right to add a shared folder. The folder path is the location where `Splunk.deb` was downloaded. Name the shared folder as this will be important later to mount the folder.`Project_AD` click ok and reboot the VM.

<img src="https://i.imgur.com/lo0ydAy.png" width="400"> <br> <sup>Ref 1: Shared Folder to VM </sup>

**Add User & Mount shared folder to VM** <br>
Add *user* to the sfgroup. After, create a directory called `share` <br> Mount the shared folder `Project_AD` to the directory `share`
```bash
$ sudo adduser arvin vboxsf
$ mkdir share
$ sudo mount -t vboxsf -o uid=1000,gid=1000 Project_AD share/
```
<img src="https://i.imgur.com/YCAvDO2.png" width="500" > <br> <sup>Ref 2: Command to mount folder to the VM </sup>

**Install Splunk on Ubuntu Server** <br>
Now change the directory to the `share` and download Splunk. Change to the directory of `opt/splunk`, Now change to the user Splunk. Go to the directory `bin` and now run the installer.
It will prompt you to create an account. Used to log on to the Splunk web interface 
```bash
$ cd share/
$ sudo dpkg -i splunk-9.2.2-d76edf6f0a15-Linux-2.6-amd64.deb
$ cd /opt/splunk
$ sudo -u splunk bash
$ cd bin
$ ./splunk start
```
<img src="https://i.imgur.com/VOvlFY3.png" width="500" > <br> <sup>Ref 3: Install Splunk </sup>

**Make Splunk boot up every time as User Splunk** <br>
Change the directory to `bin` and type the command in. 
```bash
$ cd bin
$ sudo ./splunk enable boot-start -user splunk"
```
<img src="https://i.imgur.com/XsTHUPn.png" width="500" > <br> <sup>Ref 4: Make Splunk boot up as Splunk </sup>

## ⚠️⚠️ Following will need to be done for both Windows 10 & Windows Server⚠️⚠️

**Install Splunk Universal Forwarder & Sysmon with Olaf configuration** <br>
During the setup process for Splunk Universal Forwarder the receiving indexer is the Splunk server IP `192.168.10.10` with port `9997`

<img src="https://i.imgur.com/ZbWst8O.png" width="500" > <br> <sup>Ref 5: Receiving Indexer 192.168.10.10:9997 </sup>

Extract the Sysmon download and copy the file path where it has been extracted.<br>
Open up Windows Powershell as admin. Change the directory to the file path that was copied. Install Sysmon with the Olaf configuration.
```bash
cd C:\User\Win1\Downloads\Sysmon
.\Sysmon64.exe -i ..\sysmonconfig.xml 
```
<img src="https://i.imgur.com/beU5hsy.png" width="500" > <br> <sup>Ref 6: Install Sysmon with Olaf config  </sup>

**Configure inputs.conf** <br>
Run Notepad as an admin. Type the commands below and save the file as `inputs.conf` in the file path of `programfiles>splunkuniversalforwarder>etc>system>local` 

⚠️Splunk forwarder sends info with the index `endpoint` based on the `inputs.conf` file⚠️
⚠️Any updates done to this file will require a restart of the Splunk forwarder service⚠️

```bash
[WinEventLog://Application]
index = endpoint
disabled = false

[WinEventLog://Security]
index = endpoint
disabled = false

[WinEventLog://System]
index = endpoint
disabled = false

[WinEventLog://Microsoft-Windows-Sysmon/Operational]
index = endpoint
disabled = false
renderXml = true
source = XmlWinEventLog:Microsoft-Windows-Sysmon/Operational
```
**Local system account for Splunk forwarder service** <br>
Open up services as admin. Locate SplunkFowarder service. Double-click it, go to the log on tab, and change it to the local system account. Restart the SplunkForwarder service for the changes to apply.

<img src="https://i.imgur.com/OPxRE9H.png" width="500" > <br> <sup>Ref 7: Local system account & Restart service</sup>

**Log in to Splunk server** <br>
⚠️Splunk listens on 8000⚠️ <br>
Open a web browser and type in "192.168.10.10:8000" <br>
log in, go to the settings tab, and click indexes. Create an index called `endpoint` as all the information is being forwarded under `endpoint` as the `inputs.conf` has stated.

<img src="https://i.imgur.com/E0ZUXEi.png" width="400" > <img src="https://i.imgur.com/ZC52OWL.png" width="400" > <br> <sup>Ref 8: Log in Splunk & Create an index called endpoint </sup>

**Enable Splunk to receive data** <br>
Go to the settings tab, and click on forwarding and receiving. Under receive data click on configure receiving and add port `9997`

<img src="https://i.imgur.com/9FBSDYH.png" width="400" > <br> <sup>Ref 9: Listening on port 9997 </sup>

**Search events in Splunk with the index of endpoint** <br>
To see if data is being forwarded, click on apps at the top and go to searching & reporting in the search bar type in `index=endpoint`

<img src="https://i.imgur.com/xLVgjay.png" width="500" > <br> <sup>Ref 10: Events are coming in from both hosts Windows 10 & Windows Server </sup>

**Update Kali Machine & Install Crowbar** <br>
On Kali Linux, update all repositories and install crowbar. 
```bash
$ sudo apt-get update && sudo apt-get upgrade -y
$ sudo apt-get install -y crowbar
```
<img src="https://i.imgur.com/SrhiHma.png" width="500" > <br> <sup>Ref 11: Install crowbar </sup>

**Unzip RockYou file & Make a smaller controlled small copy** <br>
> RockYou is a wordlist that contains over a million passwords that have been leaked in a data breach; It is commonly used to crack an account.

Create a directory called `ad-project`. By default, RockYou is already installed on the Kali machine. Unzip the file and make a copy into the directory `ad-project`. Output the first 20 lines of `RockYou` file to a file called `passwords.txt`
```bash
$ mkdir ad-project
$ cd /usr/share/wordlists/
$ sudo gunzip rockyou.txt.gz
$ cp rockyou.txt ~/Desktop/ad-project
$ cd ~/Desktop/ad-project
$ head -n 20 rockyou.txt > passwords.txt
```
<img src="https://i.imgur.com/K4dPSD2.png" width="500" > <br> <sup>Ref 12: Unzip rockyou.txt.gz </sup>

**Add known password & Perform brute force attack** <br>
Edit the `passwords.txt` file to add a known password to the list. Run Crowbar, Crowbar using the list of passwords to attempt an RDP login.
```bash
$ nano passwords.txt
$ crowbar -b rdp -u jdoe -C passwords.txt -s 192.168.10.100/32
```
> -b = target service
> -u = username
> -C = file
> -s = source IP 

⚠️*Target PC* needs to have RDP enabled⚠️<br>
<img src="https://i.imgur.com/nUJE1ti.png" width="500" > <br> <sup>Ref 13: Crowbar uses passwords.txt to RDP user jdoe on 192.168.10.100/32 </sup>

**View brute force attack on Splunk** <br>
click on apps at the top and go to searching & reporting in the search bar type in `index=endpoint jdoe EventCode=4624 kali`. This will search for events that have `jdoe`& `kali` along with the event code of 4624. 
> Event code 4624 = Account successfully logged on <br>
> Event code 4625 = Account failed to log on 

<img src="https://i.imgur.com/7LeC5nM.png" width="500" > <br> <sup>Ref 14: Performing data query in Splunk </sup>
<br>
<img src="https://i.imgur.com/NyvmwNb.png" width="500" > <br> <sup>Ref 15: This shows that a logon from user jdoe was successful from the machine Kali </sup>

-------------------------------------------------------------------

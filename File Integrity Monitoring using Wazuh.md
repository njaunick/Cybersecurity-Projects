# File Integrity Monitoring home lab using Wazuh.

- This is my first cybersecurity project. I decided to monitor the integrity of my files using an open source tool called Wazuh.

## Requirements

- Oracle virtual box
- A windows PC( You can install a virtual machine or you can use your current one if you are using Windows)
- Wazuh (You can download the ova file and install it in virtual box using this link https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html

## Procedure

1. Install Wazuh
2. Install the agent on my Windows PC(I decided to use it for Monitoring)
3. Configure Wazuh to monitor the files

### 1. Install Wazuh

- Drag the .ova file into Oracle Virtual box and modify the RAM allocation to 4096MB and 2 processors

![Wazuh Installation](https://i.imgur.com/AIlWYkz.png)

- Make sure the bridged adapter is turned on. This enables Wazuh to communicate with your Windows PC

![Imgur](https://imgur.com/WjDUVeP.png)

- After installing Wazuh, start it and log in using the default username and password <br>
Username: admin <br>
Password: admin

- Run the ip addr command to get the ip address of your server and plug that in your browser. This leads you to the Wazuh GUI.

![Imgur](https://imgur.com/NN3oBrb.png)

## 2. Install the agent on my own computer since I decided to use it for Monitoring

- Under agents tab > deploy new agent and choose windows and enter the ip address of your wazuh server

![Imgur](https://imgur.com/0HFXbzK.png)

- You should get a command to use in your POWERSHELL

- Run Powershell as Administrator(Required)

![Imgur](https://imgur.com/gWvMOTB.png)

-Run the command from Wazuh in powershell to install the agent on your PC

![Imgur](https://imgur.com/mkL9490.png)

-Your Wazuh dashboard should display your added agent

![Imgur](https://imgur.com/YbToWx9.png)

-Now Wazuh is up and ready to be configured to monitor our files.

## 3. Configure Wazuh to monitor your files

- Next, enable the File Integrity Monitoring on your computer

- Go to C:\ Program Files (x86)\ ossec-agent\ ossec.conf and check <br>

![Imgur](https://imgur.com/nJWKNX2.png)

- If disabled reads no then it means it's enabled

### Windows FIM use cases

#### Use Case: Monitoring System32 folder to track malicious executable

- The System32 area holds important system files, dynamic link libraries (DLLs), and executables that the Windows operating system needs to work properly.

-Let's configure Wazuh to monitor our System32 folder

-Open the configuration file (C:\ Program Files (x86)\ ossec-agent\ ossec.conf) and scroll down to Default directories to be monitored and add the following command

![Imgur](https://imgur.com/XTf9jof.png)

- Open up the WindowsUI app in the same folder and under manage hit restart to restart wazuh

![Imgur](https://imgur.com/NRCABga.png)

- Create a file(firstprojectever.txt) in the the System32 folder and go to wazuh

- You should see the following under FIM events(Integrity Monitoring)

- This was after adding the file

![Imgur](https://imgur.com/PY8HM6u.png)

- This was after deleting the file

![Imgur](https://imgur.com/OTPENau.png)

## Hurdles I faced

- The default value set for refresh time is 12 hours which prevented me form getting logs. I changed it to 15 and after refreshing Wazuh, it started registering logs.

- While creating my agent, I included a default group which prevented my Wazuh from getting logs. After changing it to none, Wazuh started registering logs.

## Conclusion

- We have successfully learned to set up Wazuh, onboarded the Windows agent, and created the FIM rule.
- Massive thank you to Rajneesh G for his walkthrough and my mentor Reggie Taylor for helping me troubleshoot.

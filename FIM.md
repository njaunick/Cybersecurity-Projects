# File Integrity Monitoring home lab using Wazuh.

- This is my first cybersecurity project where I decided to monitor the integrity of my files using an open source platform called Wazuh.

## Requirements

- Oracle virtual box
- A windows PC( You can install a virtual machine or you can use your current one if you are using Windows)
- Wazuh (You can download the ova file and install it in virtual box using this link https://documentation.wazuh.com/current/deployment-options/virtual-machine/virtual-machine.html

## Procedure

1. Installed Wazuh
2. Installing the agent on my own computer since I decided to use it for Monitoring
3. Configure Wazuh to monitor your files

### 1. Installing Wazuh

- I dragged the .ova file into Oracle Virtual box and modified the RAM allocation to 4096MB and 2 processors

![Wazuh Installation](https://i.imgur.com/AIlWYkz.png)

- Make sure the bridged adapter is turned on. This helps Wazuh communicate with your Windows PC

![Imgur](https://imgur.com/WjDUVeP.png)

- After installing Wazuh, I started it and logged in using the default username and password <br>
Username: wazuh-login <br>
Password: wazuh

- I run the ip addr command to get the ip address and used it on my browser to access the Wazuh's UI

![Imgur](https://imgur.com/NN3oBrb.png)

## 2. Installing the agent on my own computer since I decided to use it for Monitoring

- Under agents tab > deploy new agent and choose windows and the ip address of your wazuh server

![Imgur](https://imgur.com/0HFXbzK.png)

- You should get a command to use in your POWERSHELL

![Imgur](https://imgur.com/gWvMOTB.png)

-Run the commands in powershell and you should connect to your server

![Imgur](https://imgur.com/mkL9490.png)

-Your Wazuh dashboard should display your added agent

![Imgur](https://imgur.com/YbToWx9.png)

-Now Wazuh is up and ready to monitor our files.

## 3. Configure Wazuh to monitor your files

- Next, I enabled the File Integrity Monitoring on my computer

- Go to C:\ Program Files (x86)\ ossec-agent\ ossec.conf and check <br>

![Imgur](https://imgur.com/nJWKNX2.png)

- If disabled reads no then it means it's enabled

### Windows FIM use cases

#### Use Case: Monitoring System32 folder to track malicious executable

-Let's configure Wazuh to monitor our System32 folder

-Open the configuration file (C:\ Program Files (x86)\ ossec-agent\ ossec.conf) and scroll down to Default directories to be monitored and add the following command

![Imgur](https://imgur.com/XTf9jof.png)

- Then open up the WindowsUI app in the same folder and under manage hit restart to restart wazuh

![Imgur](https://imgur.com/NRCABga.png)

- Create a file in the the System32 folder and go to wazuh

- You should see the following under FIM events(Integrity Monitoring)

- This was after adding the file

![Imgur](https://imgur.com/PY8HM6u.png)

- This was after deleting the file

![Imgur](https://imgur.com/OTPENau.png)

## Conclusion

- We have successfully learned to set up Wazuh, onboarded the Windows agent, and created the FIM rule.
- Thank you to Rajneesh G for his walkthrough helped me do it and my mentor Reggie Taylor for helping me troubleshoot

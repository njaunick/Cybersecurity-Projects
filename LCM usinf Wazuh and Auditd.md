# Linux Command Monitoring using Wazuh and Auditd

- For this project, I decided to monitor malicious Linux commands on an Ubuntu PC using Wazuh and Auditd module of Linux.
- Auditd is an auditing utility native to Linux systems. It’s used for accounting actions and changes in a Linux endpoint.

## Key Steps of the project

1. Configure Auditd on an Ubuntu endpoint to account for all commands executed by a given user
2. Utilize CDB list to prepare a whitelisted command execution on Wazuh Manager

## What's Linux Auditd?

- Auditd in Linux refers to the userspace component of the Linux Auditing System.

- It's responsible for:
1. Monitors and logs security-related events
2. Tracks system authentication, authorization, and other activities
3. Configured via rules in /etc/audit/audit.rules
4. Monitors file access, system calls, configurations, and user activities
5. Logs stored in /var/log/audit/
6. Analyzed using tools like ausearch or auditctl

## Requirements

- Ubuntu Server(Wazuh agent)

- Ubuntu Server(Wazuh server)

## Setting up Wazuh Server

- I installed an Ubuntu Live Server on Proxmox and then installed the Wazuh server on it using the following command.

*```curl -sO https://packages.wazuh.com/4.7/wazuh-install.sh && sudo bash ./wazuh-install.sh -a```

- Once the assistant finishes the installation, the output shows the access credentials and a message that confirms that the installation was successful.

*```INFO: --- Summary ---
INFO: You can access the web interface https://<wazuh-dashboard-ip>
    User: admin
    Password: <ADMIN_PASSWORD>
INFO: Installation finished.```

- Use a browser and input the IP address of the server and then login using the username and password given

## Deploying the Wazuh Agent

- On a different Ubuntu Machine, I installed the Wazuh Agent for Wazuh to monitor this endpoint suing the following commands:

1. Install the GPG key:

*```curl -s https://packages.wazuh.com/key/GPG-KEY-WAZUH | gpg --no-default-keyring --keyring gnupg-ring:/usr/share/keyrings/wazuh.gpg --import && chmod 644 /usr/share/keyrings/wazuh.gpg```

2. Add the repository:

*```echo "deb [signed-by=/usr/share/keyrings/wazuh.gpg] https://packages.wazuh.com/4.x/apt/ stable main" | tee -a /etc/apt/sources.list.d/wazuh.list```

3. Update the package information:

*```apt-get update```

4. To deploy the Wazuh agent on your endpoint, select your package manager and edit the WAZUH_MANAGER variable to contain your Wazuh manager IP address or hostname:

*```WAZUH_MANAGER="IP-Address-of-your-server" apt-get install wazuh-agent```

5. Enable and start the Wazuh agent service:

*```systemctl daemon-reload``` <br>
*```systemctl enable wazuh-agent``` <br>
*```systemctl start wazuh-agent```

## Installing auditd on Wazuh agent

- I performed the following steps to install Auditd and create the necessary audit rules to query all commands run by a privileged user.

#### Install, start and enable Auditd

*```sudo apt -y install auditd```<br>

*```sudo systemctl start auditd```<br>

*```sudo systemctl enable auditd```

#### Create Auditd rules

*```echo "-a exit,always -F auid=1000 -F egid!=994 -F auid!=-1 -F arch=b32 -S execve -k audit-wazuh-c" >> /etc/audit/audit.rules```<br>

*```echo "-a exit,always -F auid=1000 -F egid!=994 -F auid!=-1 -F arch=b64 -S execve -k audit-wazuh-c" >> /etc/audit/audit.rules```

#### Reload the rules

*```sudo auditctl -R /etc/audit/audit.rules```<br>

*```sudo auditctl -l```

#### Integrate auditd log file

*```<localfile>```<br>
  *```<log_format>audit</log_format>```<br>
  *```<location>/var/log/audit/audit.log</location>```<br>
*```</localfile>```

#### Restart the Wazuh agent

*```sudo systemctl restart wazuh-agent```

## Setting up CDB List on Wazuh Manager

- I performed the following steps to create a CDB list of malicious programs and rules to detect the execution of the programs in the list.

#### Create a new CDB List

- I wrote a new CDB list suspicious-execution and filled its content with following:

*```ncat:yellow```<br>
*```nc:red```<br>
*```tcpdump:orange```

#### Add the CDB list under ruleset

- I added the list to the <ruleset> section of the Wazuh server /var/ossec/etc/ossec.conf file:

*```<list>etc/lists/suspicious-programs</list>```

#### Create Wazuh rule

- I created a high severity rule to fire when a "red" program is executed and add this new rule to the /var/ossec/etc/rules/local_rules.xml file on the Wazuh server.

*```<group name="audit">```<br>
*```<rule id="100210" level="12">```<br>
*```<if_sid>80792</if_sid>```<br>
*```<list field="audit.command" lookup="match_key_value" check_value="red">etc/lists/suspicious-programs</list>```<br>
*```<description>Audit: Highly Suspicious Command executed: $(audit.exe)</description>```<br>
*```<group>audit_command,</group>```<br>
*```</rule>```<br>
*```</group>```
#### Restart the Wazuh manager

*```sudo systemctl restart wazuh-manager```

## Proof of Concept

- On the Ubuntu endpoint, I installed and ran a "red" program netcat

*```sudo apt -y install netcat```<br>
*```nc -v```

## Visual

- I visualized the alert data in the Wazuh dashboard. To do this, I went to the Security events module and added the filters in the search bar to query the alerts.

![Imgur](https://imgur.com/7UY2Ytq.png)

- Thank you for reading.
- See you during the next one.

# Honeypot Project on Microsoft Azure

- For this project, I decided to:

1. Create a  vulnerable Windows VM on Azure
2. Collect the failed RDP login attempts
3. Geolocate the attackers using Ipgeolocation.io
4. Display/Visualize the results on a global map using Microsoft Sentinel

- The security concepts applied were:

1. Honey Pot Virtual Machine.
2. Log Collection and Analysis.
3. Geolocation Tracking.
4. Security Information and Event Management (SIEM).
5. Threat Visualization.
6. Least Privilege.
7. Secure Logging and Monitoring.
8. Ethical Hacking and Threat Research.

-The Tools and Technologies leveraged were:

1. Azure Account (free) - leveraged the $200 credit.
2. Microsoft Sentinel (SIEM) - advanced threat detection and response.
3. Log Analytics Workspace - to collect and analyze the data collected by our resources in the cloud.
4. Microsoft Defender for Cloud - to protect our cloud environment from threats.
5. Powershell - for task automation and configuration management.
6. Geolocation.io - API to provide locations of the IP addresses of the attackers.

- The Expected Outcomes were:

1. Familiarity with Microsoft Azure’s Common Tools and User Interface.
2. Understanding of Tool Integration in Azure Environments.
3. Appreciation for Cloud and On-Premise Infrastructure Security.
4. Proficiency in Azure SIEM (Security Information and Event Management).
5. Geographically locating our biggest attack sector’s locations.

- Key Steps of the Project:

1. Deploy a Vulnerable Windows VM: Spin up a Windows VM (Honeypot) with weakened security settings, exposing it to cyber threats.
2. Generation of Logs: Allowed Remote desktop protocol connection attempts by attackers, generating logs in the Event Viewer.
3. Analysis of the logs: Import the logs into a Logs Analytics Workspace to analyze the data.
4. Azure Sentinel Integration: Transfer the parsed data to Sentinel for real time monitoring and response.
5. Live SIEM Map: Visualize the live cyber-attacks on a workspace within Sentinel.

## NOTE:

### After completing this lab, it was of the utmost importance that we go back and  delete our resources promptly. Failure to do so will result in a crazy bill

### Create a virtual Machine

- I created a VM with the following configurations.

![Imgur](https://imgur.com/EVEPdLm.png)
![Imgur](https://imgur.com/qlsUBlC.png)

- Under “NIC network security group”, I selected “Advanced”, and then under “Configure network security group” selected “create new” for a new Allow all in Firewall.

- I made the following configurations to it:

![Imgur](https://imgur.com/9udpoO1.png)

- I deployed the VM

### Set up Log Analytics Workspace

- While VM is deploying, I navigated to Log analytics Workspace
- Its purpose is to ingest logs from the VM, create a log that will parse the data and feed that into Microsoft Sentinel.
- I created a Honeypot Log Analytics Workspace under the resource group I had created for the lab. I also used the same region I had used for my VM.

![Imgur](https://imgur.com/KQnMF0X.png)

### Enable gathering VM logs in Microsoft Defender for cloud

- I navigated to “Microsoft Defender for Cloud” and under environmental settings and selected the Honey Pot I had created initially.

![Imgur](https://imgur.com/YhfMoKG.png)

- Next, I clicked "Azure Defender Plans." Here, I ensured that "SQL servers" were turned OFF. Then, toggled “Servers” ON. Then selected "Save." Once completed, on the left panel, I selected "Data Collection" and chose "All Events," then saved.

![Imgur](https://imgur.com/JvmZBbZ.png)

### Connect Log Analytics to VM

- Next, I headed back to our “Log Analytics workspace” and connected it. To do this, once on the “Log Analytics” page, I selected the log that I created.

- Then, in the left panel, scrolled down and found "Virtual Machines (deprecated)," and selected the VM we had created in the earlier steps "honeypot-vm."

- I selected the VM and then clicked "Connect."

![Imgur](https://imgur.com/IhYDLV8.png)

### Setup Microsoft Sentinel

- Once on Sentinel, I clicked on "Create," , chose the workspace "HoneyPot2" and clicked "Add”.

![Imgur](https://imgur.com/m0vVnng.png)

- I then selected the correct workspace then clicked “Add”.

![Imgur](https://imgur.com/I64VtD8.png)

### Obtaining our Public IP Address

- Once I setup Microsoft Sentinel (SIEM), the VM was ready! I navigated back to the "Virtual Machines" page on Azure and copied the public IP address of the VM.

![Imgur](https://imgur.com/Bl0OHWo.png)

### Log into VM with Remote Desktop

- I was using a Windows machine, so I used "Remote Desktop Connection."

- I pasted the Public IP into the text box and connected.

![Imgur](https://imgur.com/avbkpby.png)

- I selected "More choices" and then "Use a different account".

- Here in the Windows Security pop-up, we entered credentials for the machine we created previously.

![Imgur](https://imgur.com/oHKVaVZ.png)

- BUT WAIT!!!
I intentionally used incorrect credentials this time around to provoke a “failed login attempt”. This will be used for a later step.

### Viewing logs in Event Viewer

- Event Viewer is typically used to view all the logs pertaining to the Windows system.

- Once opened, I saw categories such as "Custom Views," "Windows Logs," "Applications and Services Logs," and "Subscriptions."

- My primary focus was under "Windows Logs" and "Security." This populated all the security events that had been triggered on the VM since its creation.

- I took some time to view each tab to gain a basic understanding of what was being logged.

- I noticed a column tab titled "Keywords" and "EventID." These were areas of interest.

- Remember the failed login attempt I did earlier? That should have triggered an event log with the keyword "Audit Failure" and "EventID" "4625."

![Imgur](https://imgur.com/2zVTIsX.png)

### Turn off Windows Firewall on VM

- Next, I needed to disable the Windows firewall to allow attackers to discover the machine.
In the VM,I looked up "wf.msc" then selected "Windows Defender Firewall Properties."

- At the top, we saw tabs such as "Domain Profile," "Private Profile," and "Public Profile." For each of these, I set the firewall state to “OFF”. Then selected "Apply" and clicked “OK.”

![Imgur](https://imgur.com/auxTL77.png)

### Download PowerShell Script

- I downloaded the PowerShell script that I used to pull the data from Event Viewer.
https://github.com/joshmadakor1/Sentinel-Lab/blob/main/Custom_Security_Log_Exporter.ps1

![Imgur](https://imgur.com/V97d5OI.png)

- I then saved the script in Powershell ISE on the VM

![Imgur](https://imgur.com/DzUkkRO.png)

### Get Geolocation.io API Key

- I navigated to a browser and searched for https://ipgeolocation.io/.

- This is the tool that will allow us to view the general geolocation (longitude and latitude) from the source IPs attempting to RDP into our VM. To do that, we need an API Key.

- I selected "Get Free API Key", created an account and copied our API key into our script.

##### *Note: The free API key is only usable for up to 1000 calls per day.

### Modify the Script

- Once the API Key was added, I clicked the play button to run the script.

##### *NOTE: We had to create a file called Programlogs and modify our script for it to create the directory and store the log file.

![Imgur](https://imgur.com/2w3GuPe.png)

### Run Script To get Geo Data from Attackers

- The script ran in an infinite loop. It constantly checks the Event Viewer for new logs and writes them to a file to parse later.

- Sc incoming

### Create Custom Log in Log Analytics Workspace

- Once in Log Analytics, I navigated to Tables > Create > New custom log (MMA-Based).

- The Log Analytics Workspace needed a sample log file, so it could parse through and display entries in the sample.

- I minimized the VM, created a new "failed_rdp.txt" using Notepad, and pasted the logs into that file. Then browsed the file and added the sample.                                                       

- Once  at the collection path, I specified where the log lives on the VM.
For the type, I chose "Windows," and for the path,  copied the path from the VM "C:\Programlogs\failed_rdp.log."

- We clicked next. It appears on screen under details as:
 Custom log name: "FAILED_RDP_WITH_GEO_CL,"

- Select “Review + Create”.

![Imgur](https://imgur.com/6A7TWmC.png)

- I then got a cup of coffee and waited a little while for the Log Analytics Workspace and virtual machine to get synced up.

- After about 5 minutes, I navigated to the logs tab and attempted to run "FAILED_RDP_WITH_GEO_CL".
I had to try this a couple times before it populated any results.

![Imgur](https://imgur.com/GH8jmsx.png)

### Create Custom Fields and Extract Fields from raw Custom Log Data

- Once the custom logs begin to generate I noticed a few columns. In particular the “Raw Data” column.

- This has all of the information pertaining to the geolocation of the IP addresses attempting to connect to the VM.

- I needed to clean this up a bit. I need to be able to extract and parse this data to prepare it for my SIEM.

- To do this I used this script:

*```FAILED_RDP_WITH_GEO_CL | extend username = extract(@"username:([^,]+)", 1, RawData), timestamp = extract(@"timestamp:([^,]+)", 1, RawData), latitude = extract(@"latitude:([^,]+)", 1, RawData), longitude = extract(@"longitude:([^,]+)", 1, RawData), sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), state = extract(@"state:([^,]+)", 1, RawData), label = extract(@"label:([^,]+)", 1, RawData), destination = extract(@"destinationhost:([^,]+)", 1, RawData), country = extract(@"country:([^,]+)", 1, RawData) | where destination != "samplehost" | where sourcehost != "" | summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude```

- This will create the fields needed to extract and structure our raw data and make it easier to parse.

### Setup Map in Microsoft Sentinel with Latitude and Longitude (or country)

- Now we start setting up our Microsoft Sentinel map!

- That opened "Azure Sentinel Default Directory." Here I found the custom log group "HoneyPot2" and my resource group "honeypot".

- I navigated to "Workbooks" located in the left pane. This is where I will set up the map.

- I selected Add Workbook>Edit>Add Query and pasted the following Query

*```FAILED_RDP_WITH_GEO_CL | extend username = extract(@"username:([^,]+)", 1, RawData), timestamp = extract(@"timestamp:([^,]+)", 1, RawData), latitude = extract(@"latitude:([^,]+)", 1, RawData), longitude = extract(@"longitude:([^,]+)", 1, RawData), sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData), state = extract(@"state:([^,]+)", 1, RawData), label = extract(@"label:([^,]+)", 1, RawData), destination = extract(@"destinationhost:([^,]+)", 1, RawData), country = extract(@"country:([^,]+)", 1, RawData) | where destination != "samplehost" | where sourcehost != "" | summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude```

At the top panel, under "Visualization” I selected "Map" and then "Run Query."

##### *Note: this will take some time as attackers begin to identify this VM.

- Microsoft Sentinel then produced a rough draft of our map.

- Under “Size” I selected “Full”. Then we moved over to adjust the map settings.

- This broke down the individual fields in the query. I set our focus towards the bottom of the page on “Metric Settings” and set it to “Label”, then apply,  save and close.

- Now we see a label for each IP that attempts to login to our VM via RDP. As well as the geodata associated with it.

- I also set  the "Auto Refresh" to every 5 minutes so that it would automatically display the data on the map without manual refreshing.

##### *Note: Make sure that the script on the VM is still running, as we need this to extract the geodata.

- At this point, we let it sit for a few days.

![Imgur](https://imgur.com/uaU5QU9.png)

### Final Thoughts

- This project really helped me understand different cyber threats and how to leverage logs and SIEMS. It also gave me the chance to mess with Azure which is always a pleasure.

- Massive thank you to Josh Madakor for coming up with this project. It has given me a ton of experience.

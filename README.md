# Introduction
When I began building my CyberSecurity portfolio, one of the biggest things I wanted to have on it was a Security Information and Event Management(SIEM)-focused project, however, there were setbacks in finding one that would work. Initially, I tried doing Mad Hat's Microsoft Sentinel project but due to my subscription blocking certain Virtual Machine configurations for the Threat Intelligence feeds and the odd cuts in the vidoes that leave out specfic details, I was unable to properly complete the project and ended up scrapping it. Then I tried doing a project involving ChatGPT integration with Microsoft Sentinel but because the models used have been depreciated, the workflow wouldn't work. After doing some research, I found 2 projects that could be of some use and after using ChatGPT in the role of a hiring manager, I settled on one project. 

# Objectives
- Master Microsoft Sentinel fundamentals: understand its core feature, architecture, and integration  
- Deploy and Configure Infrastructure: spin up components such as Log Analytics workspaces, Virtual Machines, and Sentinel workspaces
- Enable Secure Networking and Host hardening: apply security best practices to both network and endpoints
- Ingest rich telemetry: use data connectors to centralize Windows Security Event logs into Sentinel
- Leverage KQL and Analytics rules: write Kusto queries and custom detection rules for threat identification
- Implement MITRE ATT&CK mapping: align detections with adversary tactics and techniques
- Deploy SOAR capabilites: use Sentinel to automate response workflow via playbooks and orchestration
- Hands-on threat simulation: test the setup with real-world attack scenarios and lab exercises

# Tools Used 
- Microsoft Sentinel
- Log Analytics Workspaces
- Windows 10 Virtual Machine
- Data connectors
- Kusto Query Language(KQL)


# Part 1: Setting up the Lab
## Step 1- Creating an Azure account
The first thing you'll want to do is head to the [Azure Portal](http://portal.azure.com/) and create an account. Once done, you'll get a $200 credit to use for your projects over 30 days or until the credit runs out.
## Step 2- Create a resource group
Search up Resource Groups in the Azure portal and create a Resource Group. You can call it whatever you like, just make it easy to remember for later on. For mine, I called it Sentinel_Lab and tagged it as Sentinel Lab

![image](https://github.com/user-attachments/assets/6da39466-8578-47d9-a6e0-1b61758bb3a6)

Once you click Review+Create, the Resource Group will be automatically deployed and you'll be able to see it in the Resource groups menu as shown below
![image](https://github.com/user-attachments/assets/eea052d9-0265-430a-badb-c9a3651411dd)

## Step 3- Creating the Virtual Machine
Now search up Virtual Machines in the Azure Portal, select the create button, use the Resource group and fill out the required fields to create the virtual machine. The following images detail my Virtual Machine configuration. Do note that your password should be easy to remember or stored in a password manager for easy access. 
![image](https://github.com/user-attachments/assets/5b511337-e80b-4ddd-a969-b5e6e19995f6)
![image](https://github.com/user-attachments/assets/8c54a984-e392-4e42-8ce5-a06b14c49e2e)
![image](https://github.com/user-attachments/assets/defc4258-7a77-4c77-8bc0-e63237b0486b)


For this lab, we won't have to make any changes to the storage, networking, management, advanced or tags as the default settings will suffice. 

## Step 4- Enabling Just-In-Time-Access
Once the VM is up and running, search for and select Microsoft Defender for Cloud, scroll down to environment settings, and select your Azure Subscription. You're going to see a page that looks like the following
![image](https://github.com/user-attachments/assets/8fb1bb64-141b-4a8c-a30a-b84a534f3b32)

Select Enable All Plans and after a few minutes, you'll end up with the following
![image](https://github.com/user-attachments/assets/6b0cdac2-9c77-41f9-9664-ed85562115e2)
![image](https://github.com/user-attachments/assets/ae84a839-cb05-48f7-8cd1-8ddee08c7ceb)

Click the Save icon to save all plans. Navigate back to your virtual machine and configure the just-in-time policy for the Virtual Machine. For mine, I had to set the IP range to the public IP of my VM along with port 3389. If you've done this correctly, then you should see the VM appear in the Just-In-Time access section of workload protections as shown below
![image](https://github.com/user-attachments/assets/5e56c350-3a6f-4773-b121-6b0619ce232c)

## Step 5- Creating Log Analytics Workspace and Sentinel Deployment
Search for Microsoft Sentinel in the Azure Portal and select Create. This will prompt you to create a Log Analytics workspace. 
![image](https://github.com/user-attachments/assets/a9eeffee-6a0b-4e95-81e0-6e22f597083f)

After creating the Log Analytics Workspace, navigate back to Microsoft Sentinel and click create. You'll be prompted to add your newly created workspace to Sentinel. Add the workspace and you'll be brought to the Microsoft Sentinel homepage.


# Part 2: Ingesting Data into Sentinel
## Step 1- Setting up and Connecting the Data Connectors
Now that we have our lab components created, it's time to start getting our Data Connectors configured. If you look at the the Incident page of Sentinel, you'll find that there are no incidents to be found because there's no data being fed into Sentinel. Navigate to the Data Connectors section of Sentinel and go to the Content Hub where you'll search for Windows Security Events via AMA. Select it and open the Connector page. At the bottom of the page, click on 'Create data connector rule' and you'll be brought to this page
![image](https://github.com/user-attachments/assets/844768d3-fafa-4c23-a93f-91a5bb8e5497)
Fill out the Basic, Resource, and Collect pages
![image](https://github.com/user-attachments/assets/269b707e-eb5d-4e56-a74b-7fd17894e6e3)
![image](https://github.com/user-attachments/assets/077c2739-04c3-4e3a-bf18-01688d99a07b)
![image](https://github.com/user-attachments/assets/d045a183-f41d-4432-8545-861778b19939)

# Part 3: Generating Security Events 
## Step 1- Configuring our VM
Remote into your VM and open Event Viewer. Expand the Windows Log, and observe the Security Logs. Search for EventID: 4624. ID:4624 is indicative of a successful logon and we can also examine more detailed information about the logon if we need to.
![image](https://github.com/user-attachments/assets/f7289bcd-05e8-47b3-96c1-f8ac90d9505a)

## Step 2- Creating Queries and Generating Logs. 
Now head back to Microsoft Sentinel and select Logs. From there, on the query page, swtich to KQL mode. Our first query is going to be the following: 
SecurityEvent
| where EventID == 4624
| project TimeGenerated, Computer, AccountName
![image](https://github.com/user-attachments/assets/ec716e64-0c98-420d-a947-e8a638e57a87)

## Step 3- Creating a Custom Rule for detecting potentially malicious tasks
Go back into the VM, open the Local Security Policy, expand the Advanced Audit Policy Configuration section to select the Object Access policy and right-click on the Audit Other Object Access subcategory. You're going to want to enable Enable Success and Failure options as shown in the image below
![image](https://github.com/user-attachments/assets/690fed32-2d96-4199-bebc-fa6d475e73da)

Open Windows Task Scheduler and navigate to Create Task. Add a name and change the 'Configure For Operating system' to Windows 10. Make sure to select the 'Run whether the user is logged in or not' option and 'Run with highest privileges'. 
![image](https://github.com/user-attachments/assets/047d5f8d-ed43-4e5c-8a7e-3762fd8e7352)

In the 'Triggers' tab, use the Image below as a guide. 
![image](https://github.com/user-attachments/assets/f17e68e2-7662-432a-92b6-7622c53a8dfd)

In the 'Actions' tab, use the image below
![image](https://github.com/user-attachments/assets/a4925149-14b5-4a07-858a-24176eb37c2a)

Return to the Logs page and create a new query using the following:
SecurityEvent                             
 | where EventID == 4698
 | parse EventData with * '<Data Name="SubjectUserName">' User '</Data>' *
 | parse EventData with * '<Data Name="TaskName">' NameofScheduledTask '</Data>' *
 | parse EventData with * '<Data Name="ClientProcessId">' ClientProcessID '</Data>' *
 | project Computer, TimeGenerated, ClientProcessID, NameofScheduledTask, User
 ![image](https://github.com/user-attachments/assets/715122a1-26d6-46a8-95ba-6267ed0d0c16)


In Microsoft Sentinel, head to Analytics and create a new Scheduled Query Rule. 
![image](https://github.com/user-attachments/assets/25e21dc7-a7b0-4a31-bcaf-b77d9f314de1)

Under Set Rule Logic, use the new query created in the Logs. The second image is the other part that you'll need to configure as well
![image](https://github.com/user-attachments/assets/8552d28b-7a36-41db-96cd-5c81ff0b1014)
![image](https://github.com/user-attachments/assets/f927b9f5-2c28-4795-be85-b529d6526606)
![image](https://github.com/user-attachments/assets/ccbd9d2c-d0d8-4ec5-93ea-083aa58f67b8)

After 10 minutes(or by luck) you should see an incident appear in the Incident section as shown below. Now you can act like a SOC analyst and close that ticket down or investigate it further.
![image](https://github.com/user-attachments/assets/ad182292-9f50-4cc8-a81e-78a9d1e32d4c)
![image](https://github.com/user-attachments/assets/0986aeba-b167-4f7c-baa9-c422fcefd8a8)
![image](https://github.com/user-attachments/assets/5890ed9a-5ced-4115-aaf3-8f8c6df709a3)


# Acknowledgement
I want to thank Enyel Salas for the inspiration and the guide for this project. His Medium article [Empowering SIEM with Microsoft Sentinel Detection Lab and SOAR Automation](https://medium.com/@enyel.salas84/project-microsoft-sentinel-lab-f2dc3477371b) and youtube videos were a big help in getting this project working. 

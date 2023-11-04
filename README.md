# Azure Sentinel SIEM Honeypot Setup

This guide outlines the process of setting up Microsoft Azure Sentinel as a Security Information and Event Management (SIEM) system to monitor a Virtual Machine (VM) acting as a honeypot. The honeypot captures live threat intelligence from RDP brute force attempts using a custom PowerShell script. This script parses Windows Log event information and uses a third-party API to retrieve geolocation information.

## Step 1: VM Configuration

1. Navigate to the Virtual Machines page in Azure.
2. Click "Create Virtual Machine" to configure the virtual machine with the following settings:
   - Resource group: HoneypotLab
   - VM name: honeypot
   - Region: US West
   - Security type: Standard
   - Image: Windows 10 Pro
   - Username and Password: Choose your preferred credentials
   - Check the Windows license box.

   ![Azure VM Configuration](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/a8036010-e642-4c9b-a336-5729e6e2161c)

   Leave the default configuration for the disks page.

3. In the Networking section, update the VM firewall rules:
   - Under NIC network security group, select "Advanced" and create a new network security group.
   - Delete the default inbound rule on the Create network security group page.
   - Add a new inbound rule with the following properties:
     - Destination port ranges: *
     - Priority: 100

   ![VM Firewall Configuration](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/a6d2f62d-13ca-455c-91b0-04c8e1c221f0)

   This configuration allows all traffic from the internet into the virtual machine, allowing you to log RDP brute-force attempts.

4. Click "OK" on the Network security group page, then "Review and create" and finally "Create."

## Step 2: Set Up Log Analytics

After configuring the VM, create a Log Analytics workspace in Azure. Follow these steps:

1. Go to the Log Analytics workspace page and click "Create Log Analytics workspace."
2. Configure the workspace with the following settings:
   - Resource group: HoneypotLab (created earlier)
   - Name: HoneypotLabLAW

   ![Log Analytics Configuration](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/834777c9-1f7e-42ed-88c9-be72666709ca)

   Keep the default settings on the Tags page and click "Review and Create," then "Create."

3. Go to the Microsoft Defender for Cloud page in Azure. Find the Environment settings page and enable Foundational CSPM and Server plans.

   ![Defender for Cloud Configuration](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/03af95bd-54a5-440a-8433-dcde01a3ddea)

   Click the save icon to save this configuration.

4. Enable logging for all events in the Log Analytics workspace.

   ![Enable Logging](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/4c53c451-d6fe-4484-84f9-8e6e49efff43)

5. Return to the Log Analytics workspace (HoneypotLabLAW), go to virtual machines on the left pane, and select the virtual machine created earlier.
6. Click "Connect."

7. Next, go to the Microsoft Sentinel page in Azure and click "Create." Select the Log Analytics workspace (LAW) created earlier and click "Add" to link this resource to your SIEM (Sentinel).

   ![Connect to Sentinel](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/5767902f-e09e-44d5-8b0e-f8759b0d323e)

## Step 3: Connect to and Configure the Honeypot VM

1. Connect to the VM via the Remote Desktop Connection app on Windows. Obtain the VM's IP address from the VM overview page.

   ![VM IP Address](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/48fb8936-1cb7-43ef-a184-e9e83beba4da)

2. Open the Remote Desktop Connection app on your PC and enter the IP address.

3. Use the login credentials created in step 1 to log in.

4. To make the VM accessible from the internet, change the firewall settings on the VM. Open the Windows Firewall with Advanced Security application by typing "wf.msc" in the search bar.

   ![Firewall Settings](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/7cf2f817-9546-4120-994d-dc84300dc053)

5. On the properties screen, turn off the firewall state for domain, private, and public profiles.

   ![Turn Off Firewall](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/bc44d857-b31a-4b53-bc0d-bdd63fcbeb73)

6. The VM is now open to the internet, and you can download a custom PowerShell script to observe live attacks (RDP logon attempts) from around the world.

7. Make an account on ipgeolocation.io to obtain an API key, which you'll need for the PowerShell script.

   ![API Key](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/e714a5a9-8eb6-42d3-a068-a5d52099cbce)

8. Open PowerShell ISE in the VM, create a new file, and paste the PowerShell script. Replace the value of $API_KEY with the API key you obtained earlier.

   ![Replace API Key](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/3c63aa38-7723-4c87-8f2f-d210a63e8925)

9. Run the script to start capturing geolocation data of attackers.

10. Set up custom logs in Azure so that Sentinel (SIEM) can access the files created by your PowerShell script. On your host machine, go to Log Analytics workspaces > HoneypotLabLAW > tables > create (mma based).

11. Since we don't have a sample log on the host machine, connect to the VM and go to C:\ProgramData\failed_rdp (created by the script). Copy the contents of the file and paste them into a notepad instance on your host machine.

12. Save the file as "failed_rdp.log" and upload it to Azure on the Sample page of "Create a custom log (tables)."

   ![Custom Log Configuration](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/9e6a267d-ae80-4936-ae5b-943f9d15a3e5)

   Click "Next," "Next" again, select Windows for the collection paths, and enter "C:\ProgramData\failed_rdp.log" for the path.

13. Name the log "FAILED_RDP_WITH_GEO."

   ![Log Name](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/c871f61d-836e-49bc-8518-81bbfbecb335)

14. Click "Next," "Next," "Create."

15. Go to the Log Analytics workspace page in Azure > HoneypotLabLAW > logs and query the "FAILED_RDP_WITH_GEO_CL" that you just created.

   ![Query Logs](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/993b7285-f405-4a50-bcbc-a280919d6000)

16. Once the logs transfer to Azure, run the query, and your screen should display the captured data.

## Step 4: Viewing Live Attacks on the Map  
To view live attacks on the map in your Azure Sentinel setup, follow these steps:  

1.  Go to the Microsoft Sentinel page, select "honeypotLabLAW," and create a new workbook that will show a live map of failed RDP attempts from around the world.

   ![Create Workbook](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/afd985b8-0aab-4661-876f-f13a38fd659f)

2. Edit the workbook by removing the default widgets.

   ![Edit Workbook](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/12ec6a8a-4755-42ca-b50a-bc07427b9129)

3. Add a query to the workbook to display live attack data on a map.

   ```
   FAILED_RDP_WITH_GEO_CL 
   | extend username = extract(@"username:([^,]+)", 1, RawData),
     timestamp = extract(@"timestamp:([^,]+)", 1, RawData),
     latitude = extract(@"latitude:([^,]+)", 1, RawData),
     longitude = extract(@"longitude:([^,]+)", 1, RawData),
     sourcehost = extract(@"sourcehost:([^,]+)", 1, RawData),
     state = extract(@"state:([^,]+)", 1, RawData),
     label = extract(@"label:([^,]+)", 1, RawData),
     destination = extract(@"destinationhost:([^,]+)", 1, RawData),
     country = extract(@"country:([^,]+)", 1, RawData)
   | where destination != "samplehost"
   | where sourcehost != ""
   | summarize event_count=count() by timestamp, label, country, state, sourcehost, username, destination, longitude, latitude
    ```   


4. After this change the visualization to map and size to full. Go to map settings and change the value of **Metric Label** to **Label** :  
    ![azurelab22](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/7c95d300-c73e-4827-9416-88954dfe38ac)

5. Click "Apply," then "Save and Close."

6. Hit "Run Query," and you'll be able to see live attacks plotted on a map.

7. Save the workbook. In the "Properties" section at the top of the workbook page, add the following details:

* Title: Failed RDP World Map
* Resource Group: Honeypot Lab
* Location: West US 3

![azurelab22](https://github.com/ClaytoneJ/Azure-Sentinel-SIEM-Honeypot/assets/119146054/7c95d300-c73e-4827-9416-88954dfe38ac)

8. Click "Apply" and change the auto-refresh to 10 minutes, then click "Apply" and "Save" again.

This will update the map with failed RDP attempts every 10 minutes, allowing you to monitor and analyze incoming attacks in real-time.

![azurelab25](https://github.com/ClaytoneJ/Azure-Sentinel-Honeypot-Setup-Guide/assets/119146054/208d0bc3-26f6-492e-9d35-d68cb8529798)

After returning to the Sentinel workspace several hours later, you can see where attacks are coming from around the world.

---
title: "Azure File Sync"
layout: post
date: 2020-12-31 22:48
image: /assets/images/markdown.jpg
headerImage: false
tag:
- Azure
- Storage
- Cloud
category: blog
author: Andrew Haratine
description: Markdown summary with different options
---


<b>INTRODUCTION</b>

Azure provides a how-to guide for deploying Azure File Sync in their technical documentation here: 
<a href="https://docs.microsoft.com/en-us/documentation/">Product Directory</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/?product=featured">Azure</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/?product=storage">Storage</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/storage/files/">Deploy Azure File Sync</a> 

I was curious to see what this would look like, so I decided to set this up in my test lab and run through the process my self. I documented the steps below. I hope you find this helpful!

Let’s start at the very beginning…

<h2><b>WHAT IS AZURE FILE SYNC?</b></h2>

Simply put, Azure File Sync allows you to extend your on-premise file share into Azure. By choosing Azure File Sync, you expand your storage capacity and provide redundancy in the cloud.

<b>REQUIREMENTS</b>

1. OS: Windows Server 2012 R2 or later.
2. Memory: 2GB of Ram or More
3. Patches: Latest Windows patches applied
4. Storage: Locally attached volume formatted in the NTFS file format. Remote storage connected by USB isn't supported.

<b>SUPPORTED PROTOCOLS</b>

1. SMB, NFS, or FTPS (any supported file sharing protocol that Windows Server supports)

<b>IMPORTANT TERMINOLOGY</b>

Azure outline’s the following terms you will need to understand to use Azure File Sync:

1. <b>Storage Sync Service</b> is the high-level Azure resource for Azure File Sync. The service is a peer of the storage account, and it can also be deployed to Azure resource groups.
2. A <b>sync group</b> outlines the replication topology for a set of files or folders. All endpoints located in the same sync group are kept in sync with each other. If you have different sets of files that must be in sync and managed with Azure File Sync, you would create two sync groups and different endpoints.
3. A <b>registered server</b> represents the trust relationship between the on-premises server and the Storage Sync Service. You can register multiple servers to the Storage Sync Service. But a server can be registered with only one Storage Sync Service at a time.
4. <b>Azure File Sync agent</b> is a downloadable package that enables Windows Server to be synced with an Azure file share. The agent has three components:
   <b>FileSyncSvc.exe Service</b> that monitors changes on endpoints. The <b>StorageSync.sys.</b> Azure file system filter driver, and the <b> PowerShell management cmdlets.</b>
5. A <b>server endpoint</b> represents a specific location on a registered server, like a folder on a local disk. Multiple server endpoints can exist on the same volume if their paths don't overlap.
6. The <b>cloud endpoint</b> is the Azure file share that's part of a sync group. The whole file share syncs and can be a member of only one cloud endpoint. An Azure file share can be a member of only one sync group at a time.
7. <b>Cloud tiering</b> is an optional feature of Azure File Sync that allows frequently accessed files to be cached locally on the server. Files are cached or tiered according to the cloud tiering policy you create.


<h2><b>HOW DOES AZURE FILE SYNC WORK?</b></h2>

Now that we know the requirements, the supported protocols, and understand the important terminology, we are fully prepared to dive into the Azure File Sync deployment process.

<b>DEPLOYMENT PROCESS OVERVIEW</b>

1.	<b>Evaluation of your on-premise system.</b>
2.	<b>Create File Sync Resources.</b> You will need a storage account to contain a file share, Storage Sync Service, and sync group. Create the resources in that order.
3.	<b>Install the Azure File Sync Agent.</b> Install this on each file server taking part in replication to the Storage Sync Service.
4.	<b>Register the Windows Server computer</b> with the Storage Sync Service.
5.	<b>Create the server endpoint.</b>
6.	<b>Verify files are syncing.</b>

<b>TIME TO IMPLEMENT:</b> 2 HOURS




<h2><b>SETUP WINDOWS SERVER VM IN AZURE</b></h2>

Normally, Azure File Sync Service is installed on an on-premise server. Since I tested this completely in my lab environment, I decided to set up my own Virtual Network in Azure and spin up a Windows Server VM that sits inside this network. <b>Skip this part if you already have an on-prem windows server.</b>

Setting up a Windows Server VM in Azure is easiest to do with PowerShell (tip: install w/ Homebrew  <a href="https://docs.microsoft.com/en-us/powershell/scripting/install/installing-powershell-core-on-macos?view=powershell-7.1">here</a> if you are on macOS).

Complete the following steps to setup a VNET/Windows Server VM in Azure:
<ol start="1">
<li>Open PowerShell and type <b>az login</b></li>
<br><p><img src="/assets/images/azureFileSync/01azLogin.jpg" alt="azLogin"></p>
 
<br><li>After you authenticate to Azure Cloud, create a resource group by running the following command:</li>
<pre>
$resourceGroup = 'operations-file-sync-rg'
$location = 'EastUS'
New-AzResourceGroup -Name $resourceGroup -Location $location</pre>

<br><li>PowerShell will confirm the resource group has been created by providing "ProvisioningState : Succeeded" output as noted in the screenshot below. After confirming this proceed to the next step.</li>

<br><p><img src="/assets/images/azureFileSync/02createResourceGroupConfirmation.jpg" alt="createResourceGroupConfirmation"></p>

<br><li>Create a Virtual Network (operations-sync-vnet) along with a subnet (operation-sync-subnet) with the following command (no PowerShell output for confirmation):</li>

<pre>
$subnetConfig = New-AzVirtualNetworkSubnetConfig `
-Name operation-sync-subnet `
-AddressPrefix 10.0.0.0/24

$virtualNetwork = New-AzVirtualNetwork `
-Name operations-sync-vnet `
-AddressPrefix 10.0.0.0/16 `
-Location $location `
-ResourceGroupName $resourceGroup `
-Subnet $subnetConfig
</pre>
<p><img src="/assets/images/azureFileSync/03createVNET.jpg" alt="createVNET"></p>

<br><li>Enter the following command to create user credentials:</li>
<pre>
$cred = Get-Credential
</pre>
<p><img src="/assets/images/azureFileSync/04credentials.jpg" alt="credentials"></p>

<br><li>Choose a username and password for the Windows Server VM.</li>
<li>Create the Windows Server VM by running the following command:</li>
</ol>
<pre>
New-Azvm `
 -Name FileServerLocal `
 -Credential $cred `
 -ResourceGroupName $resourceGroup `
 -Size Standard_DS1_v2 `
 -VirtualNetworkName operations-sync-vnet `
 -SubnetName operation-sync-subnet `
 -Image "MicrosoftWindowsServer:WindowsServer:2019-Datacenter-with-Containers:latest"</pre>
 <p><img src="/assets/images/azureFileSync/05createWindowsServer.jpg" alt="createWindowsServer"></p>

<p><img src="/assets/images/azureFileSync/06windowsServerProvisioningSuccess.jpg" alt="windowsServerProvisioningSuccess"></p>

<b>NOTE:</b> Before closing this out I made sure to tag the resource group – I always tag my resources in order to keep everything organized. I recommend getting in the habit of tagging resources whenever you create them. This will help tremendously when it comes time for reporting. I ran the following command from PowerShell to tag my new operations-file-sync-rg with 2 name-value pairs, as noted in the command and screenshot below.  

<pre>$tags = @{"Department"="Operations"; "Environment"="Test"}
$resourceGroup = Get-AzResourceGroup -Name operations-file-sync-rg
New-AzTag -ResourceId $resourceGroup.ResourceId -tag $tags</pre>

<p><img src="/assets/images/azureFileSync/07resourceTag.jpeg" alt="resourceTag"></p>

After this, I double checked a couple things in Azure Portal. I verified my LocalServerServer was added to the appropriate VNET/Subnet, and also added some tags to the resource while in the portal

<p><img src="/assets/images/azureFileSync/08verifyingFileServer.jpeg" alt="verifyingFileServer"></p>

<h2><b>EVALUATION OF YOUR ON PREMISE SYSTEM</b></h2>


<h2>Install the Azure PowerShell Modules</h2>
<ol start="1">
<li>From the Windows Server VM, I right clicked <b>Start,</b> and selected <b>Windows PowerShell (Admin). </b></li>
<li>In the console, download the latest Azure PowerShell modules.</li>

<pre>Install-Module -Name Az</pre>

<li>Enter Y when prompted to accept any untrusted repositories.</li>
</ol>

<h2>Complete an Assessment</h2>
<ol start="1">
<li>Complete a system and data file check by running the following command:</li>

<pre>Invoke-AzStorageSyncCompatibilityCheck -Path D:\CADFolder</pre>

<li>Confirm the validation is successful:</li>
<pre>
Environment validation results:

Computer name: localhost
OS version check: Passed.
File system check: Passed.

Namespace validation results:

Path: C:\CADFolder
Number of files scanned: 4
Number of directories scanned:

There were no compatibility issues found with your files.</pre>
<li>Save the results to a .csv file:</li>
</ol>

<pre>$results=Invoke-AzStorageSyncCompatibilityCheck -Path D:\CADFolder
$results | Select-Object -Property Type, Path, Level, Description | Export-Csv -Path D:\assessment-results.csv</pre>

<h2><b>CREATE FILE SYNC RESOURCES</b></h2>
Now that we have verified our Windows Server supports Azure File Sync, we are prepared to create our File Sync resources in the Azure Portal. It is important to create the resources in the following order:
<ol start="1">
<li><b>Storage Account</b> (the storage account is used to store the file share).</li>
<li><b>File Share.</b> The file share is the cloud version of the normal on-premise file share, storing all files and folders.</li>
<li><b>Storage Sync Service.</b>The Storage Sync Service is responsible for establishing trust between your company’s server and Azure.</li>
<li><b>Sync Group.</b> The sync group must contain one cloud endpoint that represents an Azure file share and one or more server endpoints that map to a path on a registered Windows file server.</li></ol>

<h2>Create the Storage Account</h2>
To create the Storage Account, complete the following steps:
<ol start="1">
<li>Login to the azure portal.</li>
<li>Search for <b>Storage accounts.</b></li>
<li>Select <b>add.</b></li>
<li>Choose your resource group and storage account name.</li>
<li>Accept defaults for the rest of the values.</li>
<li>Select <b>Review + create</b> and then select <b>Create.</b></li>
</ol>

<br><img src="/assets/images/azureFileSync/09storageAccount.jpg" alt="storageAccount">

<h2>Create the File Share</h2>
To create the <b>File Share,</b> complete the following steps:
<ol start="1">
<li>Wait for the storage account to be created.</li>
<li>Once the resource is created, select <b>Go to resource.</b></li>
<li>Go to the Overview page.</li>
<li>Select File Shares (bottom left hand corner of screenshot).</li>
<li>Click add <b>File Share.</b></li>
<li>Choose the File Server name</li>
<li>Select your quota</li>
<li>For <b>Tiers,</b> choose either <b>Transaction optimized,</b> <b>Hot</b>, or <b>Cold</b></li>   
<li>Select <b>Create.</b></li>  
</ol>

<br><img src="/assets/images/azureFileSync/010fileShares.jpeg" alt="fileShares">

<h2>Create the Storage Sync Service</h2>
To create the <b>Storage Sync Service,</b> complete the following steps:
<ol start="1">
<li>In the top left-hand corner of the Azure portal, click Create a resource.</li>
<li>Search for Azure File sync in the search box and select it.</li>
<li>Select <b>Create.</b></li>
<li>Enter the same resource group and choose an appropriate name.</li>
<li>Select <b>Review + create</b> and then select <b>Create.</b></li>
</ol>
<br><img src="/assets/images/azureFileSync/012azureSyncService.jpeg" alt="azureSyncService">

<h2>Create the Sync Group</h2>
Complete the following steps to create the <b>Sync Group.</b>
<ol start="1">
<li>Once the Storage Sync Service is complete, select <b>Go to resource.</b></li>
<li>In the Overview pain, select <b>Sync groups.</b></li>
<li>Select <b>+ Sync group.</b></li>
<li>Enter an appropriate Sync group name.</li>
<li>Choose the Storage account you created.</li>
<li>Choose the Azure File Share you created.</li>
<li>Select <b>Create.</b></li>
</ol>
 
<br><img src="/assets/images/azureFileSync/013syncGroup.jpeg" alt="013syncGroup.jpeg">

<h2><b>SETUP AZURE FILE SYNC ON WINDOWS SERVER(S):</b><i> Install the Azure File Sync Agent.</i></h2>
Now that Azure File Sync is set up in the Azure portal, we will begin preparing Azure File Sync on our on-prem Windows Server. See below for an overview of steps needed to be completed for setting up Azure File Sync on Windows Server:
<ol start="1">
<li>Disable IE Enhance Security Configuration</li>
<li>Install the Azure File Sync Agent</li>
<li>Register the Azure File Sync Agent</li>
<li>Add the server endpoint</li>
</ol>

<br>Below I cover each of these steps in detail, from start to end.

<h2>Disable IE Enhanced Security Configuration</h2>

<br>Complete the following steps to disable IE Enhanced Security Configuration from your Windows Server:
<ol start="1">
<li>In Windows Security, select More Choices -> Use a different account.</li>
<li>Enter your username and password for the Windows Server.</li>
<li>select <b>Server Manager -> Local Server.</b></li>
<li>In the <b>Properties -> IE Enhanced Security Configuration,</b> select <b>On.</b></li>

<br><p><img src="/assets/images/azureFileSync/014enhanceSecurity.jpeg" alt="enhanceSecurity"></p>

<li>Select <b>Off</b> for <b>Administrators</b> and <b>Users.</b></li>
</ol>

<h2>Install the Azure File Sync Agent</h2>
Complete the following steps to Install the Azure File Sync Agent from your Windows Server:
<ol start="1">
<li>Open any web browser.</li>
<li>Download the <b>Azure File Sync</b> agent by going to Microsoft Download Center page <a href="https://www.microsoft.com/en-us/download/details.aspx?id=57159">here</a></li>
<li>Select <b>Download.</b></li>
<li>Choose <b>StorageSyncAgent_WS2019.msi</b> and select Next.</li>
<li>Allow the pop-up and then select <b>Run.</b></li>
<li>Accept all defaults for the <b>Storage Sync Agent Setup.</b></li>
<li>Select the check box for Automatically update when the new version becomes available.</li>
<li>Run any updates that are necessary and click <b>Finish.</b></li>
</ol>

<br><img src="/assets/images/azureFileSync/015azureFileSyncAgentDownload.jpeg" alt="azureFileSyncAgentDownload">

<h2>Register the Azure File Sync Agent</h2>
Complete the following steps to <b>Register</b> the <b>Azure File Sync Agent</b> from your Windows Server:
<ol start="1">
<li>Select <b>Sign in</b> on Azure File Sync - Server Registration and sign in using your Azure credentials.</li>
<li>Enter the appropriate values for <b>Subscription, Resource Group,</b> and <b>Storage Sync Service.</b></li>
<li>Select <b>Register.</b></li>

<br><p><img src="/assets/images/azureFileSync/016azureFileSyncSignIn.jpeg" alt="016azureFileSyncSignIn.jpeg"></p>

<br><li>You should see <b>Registration successful!</b> Ensure the Network connectivity shows a status of <b>Passed.</b></li>
</ol>


<h2>Add the Server Endpoint.</h2>
Complete the following steps to add the <b>Server Endpoint</b> in the <b>Azure Portal:</b>
<ol start="1">
<li>Go to the sync group you created.</li>
<li>Select <b>Add server end point.</b></li>
<li>Select the <b>Registered Server.</b></li>
<li>Select the <b>path</b> you want to sync.</li>
<li>Select <b>Enabled</b> for <b>Cloud Tiering.</b></li>
<li>Click <b>Create.</b></li></ol>
   
<br>Now that Azure File Sync is configured on our on-premise Windows Server and in our Azure Console, we can verify file transfer is occurring.

<h2><b>VERIFY AZURE FILE SYNC</b></h2>
Complete the following steps to verify <b>Azure File Sync:</b>
<ol start="1">
<li>Copy the Storage Account <b>access key</b> from under Settings in the <b>Azure Portal.</b></li>
<li>From the on-premise server, open <b>File Explorer.</b></li>
<li>Click <b>This PC.</b></li>
<li>Select <b>Map Network Drive.</b></li>
<li>In the Folder box, enter \\storageaccount.file.core.windows.net\fileshare.</li> 
<li>Select Connect using different credentials.</li>


<br><p><img src="/assets/images/azureFileSync/018mapNetworkDrive.jpeg" alt="NetworkDrive"></p>


<br><li>Select Finish.</li>
<li>For the username, enter AZURE\thenameofyourstorageaccount.</li>
<li>For the password, paste your Access key.</li>
<li>Select <b>OK.</b></li>
<li>Go to the <b>Mapped drive</b> and find a specific path you want to test.</li>
<li>Create a folder in Documents called Confidential and to be sure, create a test document in this folder called daily-media-upload.</li>


<br><img src="/assets/images/azureFileSync/019testFolderDoc.jpeg" alt="testFolderDoc">


<br><li>Back in the <b>Azure Portal,</b> go to the same <b>\Documents\Confidential</b> path from the on-premise Windows Server.</li>
<li>Click <b>Refresh.</b></li>
<li>Verify you see the <b>daily-media-upload</b> file.</li>
</ol>

<br><img src="/assets/images/azureFileSync/020azureConfirmation.jpeg" alt="azureConfirmation">


<h2><b>SUMMARY</b></h2>

<ol start="1">
<li>The setup is straight forward.</li>
<li>The local cache option provides a quick way for users to access files.</li>
<li>You can choose between redundancy levels (i.e., locally, zonally, or geographically redundant).</li>
<li>You can integrate Azure File Sync with snapshots and backup.</li>
<li>You can restore individual items or entire shares.</li>
</ol>

<br>If you have one or multiple on-premise servers, Azure File Sync could help you centralize your operations. It's a simple as going through these steps and installing the <b>Azure File Sync Agent</b> on each on-premise Windows Server you wish to connect to Azure. Once your data is in Azure, you'll also have the potential to benefit from Azure's other features, like: security, redundancy, scheduled backups, and snapshots.

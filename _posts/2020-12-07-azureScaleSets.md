---
title: "Azure VM Scale Sets"
layout: post
date: 2020-12-14 22:48
image: /assets/images/azure.jpg
headerImage: true
tag:
- Azure
- Computing
- Cloud
category: blog
author: Andrew Haratine
description: Azure VM Scale Sets
---

<b>INTRODUCTION</b>



<h2>WHAT IS A VIRTUAL MACHINE SCALE SET?</h2>
Azure virtual machine scale sets automatically increase or decrease the number of load-balanced, identical VM instances in your environment in response to demand or a defined schedule. Virtual machine scale sets are ideal for scenarios that include compute workloads, big-data workload, and container workloads.

<b>IMPORTANT TERMINOLOGY</b>
<ol start="1">
<li><b>Horizontal Scaling</b> is the proceess of adding or removing several virtual machines in a scale set.</li>
<li><b>Vertical Scaling</b> is the process of adding resources such as memory, CPU power, or disk space to VMs.</li>
<li><b>Scheduled Calling,</b> where you proactively schedule the scale set to deploy one or <i>N</i> number of additional instances to accomodate a spike in trrafic and the spike back down when eveything is back to normal.</li>
<li><b>Autoscaling:</b> Of the workload is variable and can't always be scheduled, you can use metric-based threshold scaling.</li>
</ol>


<b>GOOD TO KNOW</b>
<ol start="1">
<li><b>Low Priority Scaling Sets</b> give you up to 80% off on price, but come at a trade off: no availability gaurantees. See <a href="https://azure.microsoft.com/en-in/resources/videos/ignite-2018-save-costs-with-low-priority-vm-scale-sets/">here</a> for more details.</li>
</ol>


<h2>HOW TO SET UP A VIRTUAL MACHINE SCALE SET</h2>

<ol start="1">
<li>Sign in to the <a href="https://portal.azure.com/">Azure portal</a> and open Azure Cloud Shell.</li>
<li>In Cloud Shell, start the code editor and create a file named test-init.yaml.</li>
<pre>code cloud-init.yaml</pre>
<li>Add the following text to the file. This file contains configuration information to install nginx on the VMs in the scale set.</li>
<pre>#cloud-config
package_upgrade: true
packages:
  - nginx
write_files:
  - owner: www-data:www-data
  - path: /var/www/html/index.html
    content: |
        Hello world - Virtual Machine Scale Set
runcmd:
  - service nginx restart</pre>

<li>Press Ctrl+S to save the file. Then press Ctrl+Q to close the code editor.</li>

<br><img src="/assets/images/01scaleSets" alt="01scaleSets">

<li>Run the following command to create a new resource group named <br>scaleset-rg<br> for your scale set:</li>

<pre>az group create \
  --location eastus \
  --name scaleset-rg</pre>

<br><img src="/assets/images/02scaleSets" alt="02scaleSets">

<li>Run the following command to create the virtual machine scale set:</li>
</ol>
<pre>az vmss create \
  --resource-group scaleset-rg \
  --name webServerScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.yaml \
  --admin-username azureuser \
  --generate-ssh-keys</pre>








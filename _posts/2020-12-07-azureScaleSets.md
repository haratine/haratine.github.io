---
title: "Azure VM Scale Sets"
layout: post
date: 2020-12-07 22:48
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
Azure provides a how-to guide for create a Virtual Machine Scale Set in their technical documentation here: 
<a href="https://docs.microsoft.com/en-us/documentation/">Product Directory</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/?product=featured">Azure</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-manage-vm">Create / Manage a VM</a><b> -> </b><a href="https://docs.microsoft.com/en-us/azure/virtual-machines/windows/tutorial-create-vmss">Create a scale set.</a> 

I was curious to see what this would look like, so I decided to set this up in my test lab and run through the process my self. I documented the steps below.


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

<h2>HOW DOES A VIRTUAL MACHINE SCALE SET WORK?</h2>

Now that we know the key benefits, the important terminology, we are better prepared to dive into the Azure virtual machine scale set deployment process.

<b>DEPLOYMENT PROCESS OVERVIEW</b>
<ol start="1">
<li><b>Deploy</b> the virtual machine scale set.</li>
<li><b>Configure</b> the virtual machine scale set.</li>
<li><b>Test</b> the virtual machine scale set.</li>
<li><b>Configure</b> the virtual machine scale set rules</li>
</ol>

<b>TIME TO IMPLEMENT:</b> 1 HOUR

<h2>DEPLOY THE VIRTUAL MACHINE SCALE SET</h2>

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

<br><img src="/assets/images/01scaleSets.jpeg" alt="01scaleSets">

<li>Run the following command to create a new resource group named <br>scaleset-rg<br> for your scale set:</li>

<pre>az group create \
  --location eastus \
  --name scaleset-rg</pre>

<br><img src="/assets/images/02scaleSets.jpeg" alt="02scaleSets">

<li>Run the following command to create the virtual machine scale set:</li>

<pre>az vmss create \
  --resource-group scaleset-rg \
  --name webServerScaleSet \
  --image UbuntuLTS \
  --upgrade-policy-mode automatic \
  --custom-data cloud-init.yaml \
  --admin-username azureuser \
  --generate-ssh-keys</pre>
</ol>

<h2>CONFIGURE THE VIRTUAL MACHINE SCALE SET</h2>
Since we are working with a web server, it is important to add a health probe at port 80, this way if the server doesn't respond it is considered unavailable. As a result, the load balancer will not route traffic to this particular server.

<ol start="1">
<li>Run the following command to add a health probe to the load balancer:</li>
<pre>az network lb probe create \
  --lb-name webServerScaleSetLB \
  --resource-group scaleset-rg \
  --name webServerHealth \
  --port 80 \
  --protocol Http \
  --path /</pre>

<li>Run the following command to configure the load balancer to route HTTP traffic to intsances in the scale set:</li>
<pre>az network lb rule create \
  --resource-group scaleset-rg \
  --name webServerLoadBalancerRuleWeb \
  --lb-name webServerScaleSetLB \
  --probe-name webServerHealth \
  --backend-pool-name webServerScaleSetLBBEPool \
  --backend-port 80 \
  --frontend-ip-name loadBalancerFrontEnd \
  --frontend-port 80 \
  --protocol tcp</pre>
</ol>

<h2>TEST THE VIRTUAL MACHINE SCALE SET</h2>
<ol start="1">
<li>In the <a href="https://portal.azure.com/">Azure portal</a>, go to Resources Groups -> scaleset-rg.</li>
<br><img src="/assets/images/03scaleSets.jpeg" alt="03scaleSets">
<li>Select the webServerScaleSet virtual machine scale set.</li>
<li>Find the public IP of the the virtual machine scale set and browse to it.</li>
<br><img src="/assets/images/04scaleSets.jpeg" alt="04scaleSets">
<li>Ensure you see the configurations made beforehand in the <b>cloud-init.yaml</b> file.</li>
<br><img src="/assets/images/05scaleSets.jpg" alt="05scaleSets">
</ol>


<h2>CONFIGURE VIRTUAL MACHINE SCALE SET RULES</h2>

<b>CREATE A SCALE-OUT RULE</b>
<ol start="1">
<li>In the <a href="https://portal.azure.com/">Azure portal</a>, go to the page for the virtual machine scale set.</li>
<li>On the virtual machine scale set page, under <b>Settings</b>, select <b>Scaling.</b></li>
<li>Select Custom <b>autoscale.</b></li>
<li>In the <b>Default</b> scale rule, ensure that the <b>Scale</b> mode is set to <b>Scale based on a metric.</b> Then select <b> + Add a rule.</b></li>
<li>On the <b>Scale rule</b> page, specify the following properties and values: <b>Metric source</b>, <b>Time aggregation</b>, <b>Metric name</b>, <b>Time grain statistic</b>, <b>Operator</b>, <b>Threshold</b>, <b>Duration</b>, <b>Operation</b>, <b>Instance count</b>, <b>Cool down (minutes)</b>, then select <b>Add.</b></li>
<br><img src="/assets/images/06scaleSets.jpeg" alt="06scaleSets">
</ol>
<b>CREATE A SCALE-IN RULE</b>
<ol start="1">
<li>In the <b>Default</b> scale rule, select <b>+ Add a rule.</b></li>
<li>On the <b>Scale rule</b> page, specify the following properties and values: <b>Metric source</b>, <b>Time aggregation</b>, <b>Metric name</b>, <b>Time grain statistic</b>, <b>Operator</b>, <b>Threshold</b>, <b>Duration</b>, <b>Operation</b>, <b>Instance count</b>, <b>Cool down (minutes)</b>, then select <b>Add.</b></li>
<br><img src="/assets/images/07scaleSets.jpeg" alt="07scaleSets">
<li>Verify the rules have saved:</li>
<br><img src="/assets/images/08scaleSets.jpeg" alt="08scaleSets">
</ol>











---
title: "Linux: Ubuntu 18.04 DHCP Server"
layout: post
date: 2021-03-07 22:48
image: /assets/images/01linux.jpeg
headerImage: true
tag:
- Linux
category: blog
author: Andrew Haratine
description: Linux DHCP Server
---


<b>INTRODUCTION</b>

Ever wonder how to setup a DHCP server on a Linux? It’s quite easy. Most people tend to do this on a Windows. However, if you want to make Linux your one-stop-shop for IP address management, continue reading. 

In this example, I go through the process of setting up DHCP on Ubuntu Server 18.04 in my VMWare Fusion home lab. 


<h2><b>WHAT IS DHCP?</b></h2>

Dynamic Host Configuration Protocol (DHCP) is a network management protocol used to automate the process of configuring devices on IP networks, thus allowing them to use network services such as DNS, NTP, and any communication protocol based on UDP or TCP. A DHCP server dynamically assigns an IP address and other network configuration parameters to each device on a network so they can communicate with other IP networks. 

<b>REQUIREMENTS</b>

1. Lab environronment & a running instance of Ubuntu 18.04.


<b>DEPLOYMENT PROCESS OVERVIEW</b>

1.	<b>Install DHCP Server</b>
2.	<b>Configure DHCP Server</b>
3.	<b>Restart the DHCP server.</b> 
3.	<b>Verify configurations</b> 


<b>TIME TO IMPLEMENT:</b> 10 minutes


<h2><b>INSTALL THE DHCP SERVER</b></h2>
<ol start="1">

<li>First, run the following command to make sure everything is up to date.</li>
<pre>sudo apt-get update</pre>
<li>Now, run the following command to install the DHCP server</li>
<pre>sudo apt-get install isc-dhcp-server -y</pre>

</ol>

<h2><b>CONFIGURE THE DHCP SERVER</b></h2>
<ol start="1">

<li>Now, configuring the DHCP server by issuing the following command:</li> 
<pre>sudo nano /etc/dhcp/dhcpd.conf</pre>
<li>In dhcpd configuration file, you need to specify a couple of things to get up and running. See screenshot below</li> 

<p><img src="https://haratine.net/assets/images/01dhcp.jpeg" alt="createVNET"></p>

<li>Uncomment out (remove the # character) the following line:</li> 
<pre>#authoritative;</pre>

<p><img src="https://haratine.net/assets/images/02dhcp.jpeg" alt="credentials"></p>
<li>Scroll to the end of the dhcpd configuration file and be sure to adjust accordingly to fit your networking needs:</li> 
<p><img src="https://haratine.net/assets/images/03dhcp.jpeg" alt="credentials"></p>

<li>Save and close dhcpd.conf file.</li> 

</ol>


<h2><b>RESTART THE DHCP SERVER</b></h2>
<ol start="1">
<li>Restart the DHCP server with the following command:</li> 
<pre>sudo systemctl restart isc-dhcp-server.service</pre>

</ol>

<h2><b>VERIFY CONFIGURATIONS</b></h2>
<ol start="1">

<li>To list out any address that have been handed out by you Linux DHCP server, issue the following command:</li> 
<pre>dhcp-lease-list</pre>

</ol>

<h2><b>SUMMARY</b></h2>
<ol start="1">
<li>The setup is straight forward and isn't a time drain.</li>
<li>The linux CLI is swift and can be a lot easier to manage than navigating the Windows gui.</li>
<li>Linux is extremely leightweight when compared to Windows (for instance, Linux doesn't require a lot of RAM) so having this option is very convenient. </li>
</ol>



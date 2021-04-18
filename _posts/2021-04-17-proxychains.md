---
title: "Configuring Proxychains on Kali Linux"
layout: post
date: 2021-04-17 22:48
image: /assets/images/kali.jpg
headerImage: true
tag:
- Kali Linux
- Security
category: blog
author: Andrew Haratine
description: Proxychains
---




<b>INTRODUCTION</b>

A proxy server is a dedicated computer or software system running on a computer which can act as an intermediary between an end device, such as a computer and another server which a client is requesting any services from. If you use Proxchains and connect to the Internet, your client IP address will not be shown but rather the IP of the proxy server. 


<h2><b>WHAT IS PROXYCHAINS</b></h2>

Proxychains is a UNIX program that forces any TCP connection made by any given application to go through proxies like TOR or any other SOCKS4, SOCKS5 or HTTP proxies.

<b>IMPORTANT TERMINOLOGY</b>

<ol start="1">
<li><b>Dynamic Chain</b> — dynamically runs through all available proxies. All proxies must be online to play and dead proxies are automatically skipped.</li>
<li><b>Strict Chain</b> — follows a connection of proxies in sequences. All proxies must be online to play and a single dead proxy in the chain will result will interrupt the connection.</li>
<li><b>Random Chain</b> — bounces connection back and fourth between random proxies. Best for testing your IDS o not for production.</li>
</ol>

<b>IMPORTANT CONCEPTS</b>
<ol start="1">
<li><b>DNS Leak</b> - DNS queries must also be proxied, if not your privacy is at risk since all DNS queries are sent using an unencrypted DNS request over the network.</li>
</ol>

<br>In this example, I go through the process of configuring <b>Proxychains</b> on Kali Linux.

<br><b>REQUIREMENTS</b>

1. Unix OS

<br><b>DEPLOYMENT PROCESS OVERVIEW</b>


1.	<b>Verify Current Environment</b>
2.	<b>Configure proxychains.conf File</b>
3.  <b>Verify Configuration</b> 
	

<b>TIME TO IMPLEMENT:</b> 15 minutes


<h2><b>VERIFY CURRENT ENVIRONMENT</b></h2>

<br>It is important to verify your current environment before begin using proxychains. After we congifure Proxychains, we can use the following information to verify Proxychains is functioning appropriately.

<ol start="1">

<li>Go to <a href="https://www.google.com">google.com</a> and search the following. Take note of your public IP address:</li>
<pre>What's my ip?</pre>

<br><img src="/assets/images/proxychains02.jpeg" alt="proxychains">

<li>Go to the following URL and take note of the IP your DNS Server is reporting for you. It should be same IP from the first step.</li>
<pre>https://dnsleaktest.com/</pre>

<br><img src="/assets/images/proxychains03.jpeg" alt="proxychains">

<li>Now, go to the following URL and take note of your DNS Server's IP address:</li>
<pre>http://www.whatsmydnsserver.com/</pre>

<br><img src="/assets/images/proxychains04.jpg" alt="proxychains">


<li>Change directories to Documents. Here, we will make a file to keep trake of the IP addresses we just took note of.</li>
<pre>cd ~/Documents</pre>
<li>Create a file and call it environment by running the following command:</li>
<pre>touch environment</pre>
<li>Open the file with the following command then paste IP addresses along with their counterpart.</li>
<pre>vim environment</pre>
</ol>


<br><img src="/assets/images/proxychains20.jpg" alt="proxychains">



<h2><b>CONFIGURE PROXYCHAINS.CONF FILE</b></h2>
<ol start="1">
<li>Open the proxychains.conf file in vim on Kali Linux by issuing the following command:</li> 
<pre>sudo vim /etc/proxchains.conf</pre>
<li>In the /etc/proxychains.conf configuration file, enable the option for <i>dynamic chain:</i></li> 

<br><img src="/assets/images/proxychains05.jpeg" alt="proxychains">

<li>Scroll down and enable the option for <i>Proxy DNS Requests:</i></li> 

<br><img src="/assets/images/proxychains06.jpg" alt="proxychains">

<li>At the botom of the configuration file add the following to enable SOCKS5 and Tor:</li> 
<pre>socks5   127.0.0.1 9050</pre>

<br><img src="/assets/images/proxychains07.jpeg" alt="proxychains">

<li>If you haven't already, install Tor by running the following command:</li> 
<pre>sudo apt-get install tor</pre>
<li>Start the Tor service and check it's status after by running the following commands:</li> 
<pre>
service tor start
service tor status
</pre>
</ol>



<h2><b>VERIFY CONFIGURATIONS</b></h2>
<ol start="1">
<li>Run the following command to test using Firefox with the new Proxychains configuration. Firefox will open.</li> 
<pre>proxychains firefox</pre>

<br>Note the dynammic chain configuration in Terminal and how the browser opens up in German:

<br><img src="/assets/images/proxychains08.jpeg" alt="proxychains">

<li>Go to <a href="https://www.google.com">google.com</a> and search the following. Take note of your public IP address:</li>
<pre>What's my ip?</pre>

<br><img src="/assets/images/proxychains09.jpeg" alt="proxychains">

<li>Go to the following URL and take note of the IP your DNS Server is reporting for you:</li>
<pre>https://dnsleak.com/</pre>

<br><img src="/assets/images/proxychains10.jpeg" alt="proxychains">

<li>Now, go to the following URL and take note of your DNS Server's IP address:</li>
<pre>http://www.whatsmydnsserver.com/</pre>

<br><img src="/assets/images/proxychains22.jpg" alt="proxychains">


<li>Change directories again to Documents. Here, we will make note of the IP addresses after our Proxchains configuration so we can compare the differences.</li>
<pre>
cd ~/Documents
vim environment
</pre>
<li>Paste each new IP address in and compare.</li>
</ol>

<br><img src="/assets/images/proxychains21.jpg" alt="proxychains">

<br>By looking at our <i>environment</i> file, we are able to verify that Proxychains is (1) forwarding our IP address through Tor proxy, (2) our DNS quieres our proxied through Tor (and mitigating DNS Leak), and (3) our DNS server is reporting a different IP address. Proxychains is configured properly.


<h2><b>USE CASES</b></h2>

<br>See below for some exampes use cases for Proxychains:
<ol start="1">
<li>Proxychains with Nmap scans:</li> 
<pre>
proxychains [nmap command]
proxychains nmap -sC -sV -oA nmap/outputfile 10.10.10.10
</pre>
<li>Proxychains with SSH:</li> 
<pre>
ssh -D 127.0.0.1:8080 target.com 
</pre>
</ol>
<br>This will make SSH forward all traffic sent to port 8080 to target.com. You will need to add 127.0.0.1:8080 to the Proxychains proxy list.


<h2><b>SUMMARY</b></h2>
<ol start="1">
<li>The concept of proxies and knowing how to use proxies is important to learn for anyone in infosec.</li>
<li>Having the Proxychains tool configured and ready for use can prove useful, even if you don't use the tool daily.</li>
<li>Be sure to double check your configurations - a DNS leak can easily expose your identity.</li>
</ol>
    



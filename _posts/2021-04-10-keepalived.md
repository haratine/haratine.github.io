---
title: "High Availability with keepalived"
layout: post
date: 2021-04-10 22:48
image: /assets/images/redhat.jpg
headerImage: true
tag:
- Linux
- High Availability
- Infrastructure
category: blog
author: Andrew Haratine
description: Keepalived
---




<b>INTRODUCTION</b>

keepalived can be used to monitor services or systems and to automatically failover to a standby node if problems occur. High availability is achieved by the Virtual Redundancy Routing Protocol (VRRP). As a result, high availabilty can be maintained across your environment.

<h2><b>WHAT IS KEEPALIVED</b></h2>

keepalived is a system daemon in Linux systems that provides frameworks for both high availability and load balancing. 

<b>IMPORTANT TERMINOLOGY</b>

<ol start="1">
  <li><b>VIP</b> — Virtual IP, a virtual IP address able to automatically switch between the servers in case of a failure.</li>
  <li><b>Master</b> — a server the VIP is currently active on.</li>
<li><b>Backup</b> — a server the VIP will switch to in case of a Master failure.</li>
<li><b>VRID</b> — Virtual Router ID.</li>
</ol>

<b>IMPORTANT CONCEPTS</b>
<ol start="1">
<li><b>Basic operation algorithm:</b> at fixed intervals, the Master server sends VRRP packets (heartbeats) to the specific multicasting address 224.0.0.18, and all slave server listen to this address. </li>
<li>If a <b>Backup Server</b> does not receive any heartbeat packets, it starts the Master selection procedure.</li>
</ol>
 

<br>In this example, I go through the process of configuring <b>keepalived</b> on 2 Red Hat CentOS boxes.

<br><b>REQUIREMENTS</b>

1. Lab environronment & 2 running instances of CentOS 7 LVS.


<b>DEPLOYMENT PROCESS OVERVIEW</b>


1.	<b>Install keepalived</b>
2.	<b>Configure Master Server keepalived.conf File</b>
3.	<b>Configure Backup Server keepalived.conf File</b>
4.  <b>Verify Configuration</b> 
	



<b>TIME TO IMPLEMENT:</b> 15 minutes


<h2><b>INSTALL KEEPALIVED</b></h2>
<ol start="1">

<li>First, run the following commands to make sure everything is up to date/upgraded.</li>
<pre>sudo yum -y update</pre>
<pre>sudo yum -y upgrade</pre>
<li>Run the following command to install <b>keepalived</b></li>
<pre>sudo yum -y install keepalived</pre>
</ol>



<h2><b>CONFIGURATION OVERVIEW</b></h2>

Note: in this example we are configuring 2 Linux Virtual Servers with <i>keepalived</i>. <i>keepalived</i> uses <i>/etc/keepalived/keepalive.conf</i> as its primary configuration file. In this file, we will need specifiy the following on each server:

1. <b>Router ID</b> - unique identifier for the server.
2. <b>State</b> - state of server (Master of Slave).
3. <b>Priority</b> - what server has priority (highest is selected as Master)
4. <b>Authentication</b> - type of authentication (PASS) and what password.
5. <b>VIP</b> - Virtual IP Address.

<h2><b>CONFIGURE THE KEEPALIVED.CONF FILE [MASTER SERVER]</b></h2>
<ol start="1">
<li>Open the keepalive.conf file in vim on LVS1 by issuing the following command:</li> 
<pre>sudo vim /etc/keepalived/keepalive.conf</pre>
<li>In /etc/keepalived/keepalive.conf configuration file, you need to specify the following:</li> 
<pre>
#MASTER SERVER
global_defs {
    notification_email {
        andrew@haratine.net
    }
    notification_email_from andrew@haratine.net
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id main_server
}
vrrp_instance VI_1 {
    state MASTER
    interface eth0
    virtual_router_id 51
    priority 100
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass passwordhere
    }

    virtual_ipaddress {
        # virtual IP address
        10.10.10.10
    }
}
</pre>
<li>Feel free to add additional configurations, as noted under <i>global_defs</i>. Save and close keepalive.conf file.</li> 
<li>Run the following commmands to start the daemon:</li>
<pre>
systemctl enable keepalived
systemctl start keepalived
</pre>
</ol>

<h2><b>CONFIGURE THE KEEPALIVED.CONF FILE [BACKUP SERVER]</b></h2>
<ol start="1">
<li>Open the keepalive.conf file in vim on LVS1 by issuing the following command:</li> 
<pre>sudo vim /etc/keepalived/keepalive.conf</pre>
<li>In /etc/keepalived/keepalive.conf configuration file, you need to specify the following:</li> 
<pre>
#BACKUP SERVER
global_defs {
    notification_email {
        andrew@haratine.net
    }
    notification_email_from andrew@haratine.net
    smtp_server 127.0.0.1
    smtp_connect_timeout 30
    router_id backup_server
}
vrrp_instance VI_1 {
    state BACKUP
    interface eth0
    virtual_router_id 51
    priority 99
    nopreempt
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass passwordhere
    }

    virtual_ipaddress {
        # virtual IP address
        10.10.10.10
    }
}
</pre>
<li>Save and close keepalive.conf file.</li>
<li>Run the following commmands to start the daemon:</li>
<pre>
systemctl enable keepalived
systemctl start keepalived 
</pre>
</ol>

<br><img src="/assets/images/01keepalived.jpg" alt="01keepalived">

<h2><b>VERIFY CONFIGURATIONS</b></h2>
<ol start="1">
<li>To test the configuration, run the following command on both machines:</li> 
<pre>sudo systemctl status keepalived</pre>
</ol>

<br><img src="/assets/images/02keepalived.jpg" alt="02keepalived">



<h2><b>SUMMARY</b></h2>
<ol start="1">
<li>Minimal congiruation was needed to conigure high availbiilty.</li>
<li>Attention to detail is key: ensure the <b>Backup</b> server has a lower priority than the <b>Master</b>, the <b>STATE</b> is set appropriately, and the <b>VIP</b> is not being used by another DHCP host in your environment.</li>
<li>To be sure, <i>keepalived</i> is a great option if you are considering high availability for your critical infrastructure.</li>
</ol>
    



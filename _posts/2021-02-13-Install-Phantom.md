---
layout: post
title: Install Splunk Phantom
published: true
date: 2021-02-13 23:49:00
category: Phantom
tags: Splunk Phantom admin
---
*Personal notes on installing an **unprivileged** Splunk Phantom instance using the tarball (.tgz) file.  



https://docs.splunk.com/Documentation/Phantom/4.10.1/Install/InstallUnprivileged  

Version 4.10.1 of the Install guide was updated and feels more streamlined.  Just follow those steps accordingly and use the following notes as reference.  

1. Enable proxy  
    `echo "export http_proxy=http://proxy.example.com:8080/" &gt;&gt; /etc/profile`  
    `echo "export https_proxy=http://proxy.example.com:8080/" &gt;&gt; /etc/profile`  
    `echo "export HTTP_PROXY=http://proxy.example.com:8080/" &gt;&gt; /etc/profile`  
    `echo "export HTTPS_PROXY=http://proxy.example.com:8080/" &gt;&gt; /etc/profile`  
    `echo "export http_proxy=http://proxy.example.com:8080/" &gt;&gt; /etc/profile.d/http_proxy.sh`  
    `echo “export https_proxy=http://proxy.example.com:8080/“ &gt;&gt; /etc/profile.d/http_proxy.sh`  
    `echo “export http_proxy=http://proxy.example.com:8080/“ &gt;&gt; /etc/profile.d/http_proxy.csh`  
    `echo “export https_proxy=http://proxy.example.com:8080/“ &gt;&gt; /etc/profile.d/http_proxy.csh`  

2. Disable SELinux  
    Edit `/etc/selinux/config` to disable SELinux. Change the SELINUX= entry to:  
    `SELINUX=disabled`

3. Yum update then reboot

    `yum clean all`  
    `yum update`  
    `shutdown -r now`  

4. Install dependencies  
    `yum install -y libevent libicu c-ares bind-utils java-1.8.0-openjdk-headless mailcap fontconfig ntpdate perl rsync xmlsec1 xmlsec1-openssl libxslt ntp zip net-tools policycoreutils-python libxml2 libcurl gnutls`

5. DO NOT install GlusterFS (Skip step 6 in the document for installing GlusterFS)  

6. Set firewall rule to allow Phantom ports

    Standalone  
    `firewall-cmd —add-port=22/tcp —permanent`  
    `firewall-cmd —add-port=80/tcp —permanent`  
    `firewall-cmd —add-port=443/tcp —permanent`  
    `firewall-cmd —add-port=8000/tcp —permanent`  

    PostgreSQL  
	`firewall-cmd —add-port=5432/tcp —permanent`  
    `firewall-cmd —add-port=6432/tcp —permanent`  

    Embedded Splunk Enterprise  
    `firewall-cmd —add-port=5121/tcp —permanent`  
    `firewall-cmd —add-port=5122/tcp —permanent`  

    Cluster Node  
    `firewall-cmd —add-port=4369/tcp —permanent`  
    `firewall-cmd —add-port=5671/tcp —permanent`  
    `firewall-cmd —add-port=8300/tcp —permanent`  
    `firewall-cmd —add-port=8301/tcp —permanent`  
    `firewall-cmd —add-port=8302/tcp —permanent`  
    `firewall-cmd —add-port=8888/tcp —permanent`  
    `firewall-cmd —add-port=15672/tcp —permanent`  
    `firewall-cmd —add-port=25672/tcp —permanent`  
    `firewall-cmd —add-port=27100-27200/tcp —permanent`  

    Save and reload FW Policy  
    `firewall-cmd —reload`  
    `firewall-cmd —list-all`.

7. NTP (Enable if it’s not already)

8. Create a file called /etc/sysctl.d/50-phantom.conf.  See step 11 of the unprivileged install doc to paste in the necessary configuration  

9. Apply the new kernel settings  

    `sysctl -system`  

10. Create the user account that will be used to run Splunk Phantom.

    `adduser -c “Phantom User” phantom`  
    `passwd phantom`

11. Create a file called /etc/security/limits.d/25-phantom-limits.conf.  This file sets resource limits for the user that will run Splunk Phantom.

    `touch /etc/security/limits.d/25-phantom-limits.conf`

12. Edit the file /etc/security/limits.d/25-phantom-limits.conf to add these settings:

    `phantom          hard    nofile          64000`  
    `phantom          soft    nofile          64000`  
    `phantom          hard    nproc           64000`  
    `phantom          soft    nproc           64000`             

13. Apply the new security settings.

    `sysctl —system`

14. Set /opt/phantom permission

    `chown phantom:phantom /opt/phantom`  
    `chmod 775 /opt/phantom`

15. Exit the root user

    `exit`

16. Use sudo to switch to the phantom user

    `sudo su - phantom`

17. Following steps must be done as the phantom user
    - Install can take a long time.  
    - If you are on a VPN connection, confirm that you have plenty of hours remaining VPN session.  
    - If in doubt, reestablish VPN, or switch to another connection method to avoid getting disconnected in the middle of the install.

18. Copy or upload then uncompress phantom-x.x.yyyyy-1.tgz to `/opt/phantom` (**NOTE:**  tar will overwrite permission of “.” to 0750.  Leave it as is.  No change)

    `cd /opt/phantom`  
    `tar xvzf phantom-x.x.yyyyy.tgz`  

19. Install

    run whoami to be sure you are using the “phantom” user  
    `whoami`  

    Install and configure Phantom with the WebUI on port 8000  
    `./phantom_tar_install.sh install —https-port=8000`

For more installation command line options, see phantom_tar_install.sh options.  

**Optional:** During install, open a second terminal, SSH in then switch to the phantom user. Then `tail -f /opt/phantom/var/log/phantom/phantom_install_log` to see the installation progress.

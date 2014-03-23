 ![IT Buff](http://webfiles.co/images/itbuff/logo.png)

![IT Buff](http://webfiles.co/images/laravel.png)<br>
  <a href="https://packagist.org/packages/laravel/framework"><img src="https://github-camo.global.ssl.fastly.net/68e11e213bb9de27c39735d71d895e565e4a0171/68747470733a2f2f706f7365722e707567782e6f72672f6c61726176656c2f6672616d65776f726b2f76657273696f6e2e706e67" alt="Latest Stable Version" data-canonical-src="https://poser.pugx.org/laravel/framework/version.png" style="max-width:100%;"></a> <a href="https://packagist.org/packages/laravel/framework"><img src="https://github-camo.global.ssl.fastly.net/adb18155bdfcb2c721d65b9966d88e00ef0aa078/68747470733a2f2f706f7365722e707567782e6f72672f6c61726176656c2f6672616d65776f726b2f642f746f74616c2e706e67" alt="Total Downloads" data-canonical-src="https://poser.pugx.org/laravel/framework/d/total.png" style="max-width:100%;"></a> <a href="https://travis-ci.org/laravel/framework"><img src="https://github-camo.global.ssl.fastly.net/ab9e32f6bed9a1d50326614b1ef7753eae090836/68747470733a2f2f7472617669732d63692e6f72672f6c61726176656c2f6672616d65776f726b2e706e67" alt="Build Status" data-canonical-src="https://travis-ci.org/laravel/framework.png" style="max-width:100%;"></a> <a href="https://packagist.org/packages/laravel/framework"><img src="https://github-camo.global.ssl.fastly.net/05c22e45aebd8bc5c6197c99ae3aff2fa3c21dd9/68747470733a2f2f706f7365722e707567782e6f72672f6c61726176656c2f6672616d65776f726b2f6c6963656e73652e706e67" alt="License" data-canonical-src="https://poser.pugx.org/laravel/framework/license.png" style="max-width:100%;"></a>


# ![IT Buff](http://webfiles.co/images/ubuntu.gif) ![IT Buff](http://webfiles.co/images/sambalogo.png) Installation guide
___

> #### Date: 23 March 2014
> 
> This may not be a Laravel specify tutorial, but it shows you how to set up SAMBA 3.6 on a ubuntu server version 12.04 LTS which authenticates all users to a Windows Active Directory Server, including group based permissions. 
>
> We'll be installing Samba, Winbind, Kerberos and nsswitch. This allows you to have a Linux machine sharing and authenticating files and folders from Windows Active Directory.
>
> We bind a Linux machine to the Active Directory (disabling shell access for the users), and we then configure Samba to accept these users to the shares we set up.
>
> ###Overview
>
> * Install Packages
> * Configure NTP & DNS
> * Configure Kerberos
> * Configure nsswitch
> * Configure Samba
> * Join the Domain
> * Configure Samba shares
> * Check the setup
>
> ### Data Used in this tutorial.<br>(simply replace the data with your prefered names or IP)
>  
> | Item                              | Data                            |
> |-----------------------------------|---------------------------------|
> | Active Directory Domain:          | ITBUFF.LOCAL                    |
> | Realm/workgroup:                  | ITBUFF                          |
> | Windows Server (DHCP/DNS/AD) IP:  | 192.168.1.3                     |
> | Windows Server netbios name:      | WS1                             |
> | Windows Server FQDN:              | WS1.ITBUFF.LOCAL                |
> | this server ubuntu 12.04 LTS IP:  | 192.168.1.5                     |
> | this server netbios name:         | webserver                       |
> | this server FQDN:                 | webserver.ITBUFF.LOCAL          |
> | Network gateway (ADSL Router)     | 192.168.1.254                   |
> | DNS Server                        | 192.168.1.3                     ||
>  
> ### Shares
>
> | Share Name                        | Server   | AD     |
> |-----------------------------------|----------|--------|
> | Domain Users                      | WS1      | yes    |
> | Dev Admins   (create now)         | WS1      | yes    |
> | Dev Team 1   (create now)         | WS1      | yes    |
> | Dev Team 2   (create now)         | WS1      | yes    |
> | Dev Team 3   (create now)         | WS1      | yes    |
> | public       (create now)         | WS1      | yes    |
> | marketing    (create now)         | WS1      | yes    ||
> 
> ###This tutorial was tested with the following software:
> 
> | Software or OS                    | Version                         |
> |-----------------------------------|---------------------------------|
> | ubuntu Server                     | 12.04 LTS                       |
> | Samba                             | 3.6.3                           |
> | Windows Small Business Server     | 2011                 AD         |
> | Windows Small Business Server     | 2008 and 2008 R2     AD         |
> | Windows Server Standard           | 2003 and 2008                   |
>
> ### Additional notes: 
> This tutorial assumes that your Windows server is the Primary DHCP and DNS server. As such ubuntu should be setup as **MANUAL NETWORK SETTING**. If you are going to install ubuntu from scratch., which you should, then just after the installer discovers your hardware configuration, ***press on the back button*** which will allow you to choose manual network settings. Refer to Data Used above for IP settings. make sure to use the right IP address for your network.
>
> ### This tutorial's procedures was tested with the following software:
> 
> | Option                            | Data                            |
> |-----------------------------------|---------------------------------|
> | ubuntu Server                     | 12.04 LTS                       |
> | Samba                             | 3.6.3                           |
> | Windows Small Business Server     | 2011                 AD         |
> | Windows Small Business Server     | 2008 and 2008 R2     AD         |
>
> So before starting this Samba configuration make sure that the Windows server is on the LAN and operational with all the groups you need to use created. don't forget to also **add members to the groups**.
> 
> Install ubuntu with no packages. It's best not to select Samba during setup as we will do this manually in step 1.
>
> ntp is for Time synchronization between PDC and this server

 * ### **Step 1** - Update and Getting the necessary packages

	To make it easier stay logged in as root:
	
	~~~
		sudo bash
	~~~

    
	If you did not select samba during server install:
    
	~~~bash
		apt-get update && sudo apt-get -y upgrade
		sudo apt-get -y dist-upgrade
        
		sudo apt-get install python-software-properties
        
		apt-get install ntp krb5-user samba smbfs smbclient winbind
        
		sudo apt-get update
		sudo apt-get -y upgrade
		sudo apt-get -y dist-upgrade
		sudo apt-get -y install -f
		sudo apt-get -y autoclean
	~~~
	
	Otherwise: 
    
	~~~bash
		apt-get install ntp krb5-user smbfs smbclient winbind
 	~~~
    
    If your having trouble try disaling the firewall. **ONLY IF PROBLEMS OCCUR LATER**
    
    ~~~
        sudo ufw disable
    ~~~
    
    Display Samba version
    
    ~~~
    	smbd --version
    ~~~
___

 * ### **Step 2** - Configure NTP
 
	Sync time with windows AD
 	
	~~~
		vim /etc/ntp.conf

		make sure there's only one line with server . . .  and make it:
		
		server 192.168.1.3
	~~~
___

 * ### **Step 3** - Configuring Kerberos
 
	Edit /etc/krb5.conf First make a backup.
	
    ~~~
	    cp /etc/krb5.conf /etc/krb5.conf.original
	~~~
    ~~~
	    vim /etc/krb5.conf
	~~~
    ~~~
		[logging]
			default = FILE10000:/var/log/krb5lib.log

		[libdefaults]
			ticket_lifetime = 24000
			default_realm = SIACOM.LOCAL

		[realms]
		    SIACOM.LOCAL = {
		    kdc = windows_server_name
		    admin_server = windows_server_name
		    default_domain = SIACOM.LOCAL

		    }

		[domain_realm]
		    .domain.local = SIACOM.LOCAL
		    domain.local = SIACOM.LOCAL
     ~~~

	If everything goes ok so far, you should be able to check whether Kerberos is working by issuing the following command. You will need an account with administrative privileges on the Windows side to make this work.
	~~~
	    kinit Admin@SIACOM.LOCAL
    ~~~

	If you did not get any errors or messages, it probably worked. In order to know exactly what happened, run the command:
	~~~
	   klist
	~~~

___


 * ### **Step 4** - Configuring Samba
	
    Edit the /etc/samba/smb.conf file. You can do it by running
	~~~
		vim /etc/samba/smb.conf
    ~~~
	Replace and/or add the following lines to the samba configuration file:
	~~~
	    [global]
		workgroup = DOMAIN_NAME
		realm = DOMAIN.LOCAL
		netbios name = ubuntu_server_name
		server string = %h server (Samba %v, Ubuntu)
		dns proxy = no
		log file = /var/log/samba/log.%m
		max log size = 1000
		syslog = 0
		panic action = /usr/share/samba/panic-action %d
		security = ADS
		domain master = no
		idmap uid = 10000-20000
		idmap gid = 10000-20000
		template shell = /bin/bash
		template homedir = /home/%D/%U
		winbind enum groups = yes
		winbind enum users = yes
		winbind use default domain = yes
		winbind separator = +
		usershare allow guests = yes
	~~~
	
    Now, check whether the samba settings are correct by running
    
	~~~
		testparm
    ~~~

	Restart the Winbind and Samba services by running

	~~~
		/etc/init.d/winbind stop
		/etc/init.d/smbd restart
		/etc/init.d/winbind start
    ~~~

___

 * ### **Step 5** - Join the Windows Domain

	You should now be able to join the Windows domain by running the following command:

		net ads join -U Administrator@DOMAIN.LOCAL

___

 * ### **Step 6** - Configure Winbind

	Edit the /etc/nsswitch.conf file and make it look like this:
	~~~
	    passwd: compat winbind
	    group:  compat winbind
	    shadow: compat winbind

		hosts:     files dns wins
		networks:  files dns

		protocols:   db files
		services:    db files
		ethers:      db files
		rpc:         db files

		netgroup:     nis
	~~~
	
    Restart Samba and Winbind
    
	~~~
		/etc/init.d/winbind stop
		/etc/init.d/smbd restart
		/etc/init.d/winbind start
	~~~

	if everything is alright you should be able to retrieve information from your Windows Domain.

	To get a list of users run
	
    ~~~
		wbinfo -u
	~~~

	To get a list of groups run
	
    ~~~
		wbinfo -g
	~~~

	To check for DOMAIN via RPC
	
    ~~~
		wbinfo -t
        
        Expected output:    checking the trust secret for domain DOMAINNAME via RPC calls succeeded
	~~~

    To get details about the domain run
	
    ~~~	
		net ads info
	~~~

___

 * ### **Step 7** - Enabling login from domain accounts

	Edit the file /etc/pam.d/common-auth and add the following line at the start of the file:
	~~~
		vim /etc/pam.d/common-auth

			auth    sufficient                      pam_winbind.so
	~~~
    
	This basically means that if a user successfully logins using a domain account that is enough to login to the system.

	Edit the /etc/pam.d/common-session file and add the following line to enable automatic creation of home folder when a new user logins to the linux box:

		vim /etc/pam.d/common-session

			session required pam_mkhomedir.so

	The folder will be created according to the parameter in the smb.conf file

		template homedir = /home/%D/%U

	> If you keep getting asked for the sudo the **password twice** then:
    >   
	> 	sudo vim /etc/pam.d/common-auth
    >    
	> 	auth    sufficient                      pam_ldap.so use_first_pass
	>
    >  if not change auth    sufficient                      pam_winbind.so to:
    
	~~~
		auth    sufficient                      pam_winbind.so try_first_pass
	~~~





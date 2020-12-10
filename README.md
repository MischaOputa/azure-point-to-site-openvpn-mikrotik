# azure-point-to-site-openvpn-mikrotik
create openvpn server on mikrotik create free azure account on azure create two linux ubuntu VMs in different virtual networks and create routing between them connect one of the VMs to vpn  
Assuming you have a routeros/Mikrotik connection setup already:
Download winbox  https://mikrotik.com/download select winbox for your os, download and run and login with your credentials e.g
IP: 20.xx.xx5.xxx
username: admin
password: xxxxxxxxxx
10.0.0.0/16 (default address range)
10.1.0.0/1610.0.0.0/16 (default subnet)
In this network, MikroTik Router (RouterOS v6.46) is connected to internet through ether1 interface with your(Mikrotik router) IP address. In your network, this IP address should be replaced with public IP address. MikroTik Router’s ether2 interface is connected to local network having IP network 10.10.11.0/24. We will configure OpenVPN server in this router and OpenVPN client in a Windows Operating System. After OpenVPN Server and Client configuration, the router will create a virtual interface (OpenVPN Tunnel) across public network where VPN Gateway IP address will be 192.168.2.1 and Client machine will get an IP Address within 192.168.2.0/24 IP Block. We will also declare route in OpenVPN Client so that connected VPN user can access resources of OpenVPN server’s network.
Enable and configure OpenVPN Server in MikroTik Router. It is assumed that your WAN and LAN networks are working without any issue.

Complete MikroTik OpenVPN Server configuration can be divided into the following three steps.

Step 1: Creating TLS Certificate for OpenVPN Server and Client
Step 2: Enabling and Configuring OpenVPN Server
Step 3: Creating OpenVPN Users
Please refer to https://wiki.mikrotik.com/wiki/Main_Page for details 
We will create required OpenVPN certificate from our RouterOS. OpenVPN Server and Client require three types of certificates:

CA (Certification Authority) Certificate
Server Certificate and
Client Certificate
Creating CA certificate

The following steps will show how to create CA certificate in MikroTik RouterOS.

From Winbox, go to System > Certificates menu item and click on Certificates tab and then click on PLUS SIGN (+). New Certificate window will appear.
Put your CA certificate name (for example: CA) in Name input field. Also put a certificate common name (for example: CA) in Common Name input field.

Click on Key Usage tab and uncheck all checkboxes except crl sign and key cert. sign checkboxes.
Click on Apply button and then click on Sign button. Sign window will appear now.
Your created CA certificate template will appear in Certificate dropdown menu. Select your newly created certificate template if it is not selected.
Put MikroTik Router’s WAN IP address (example: 117.58.247.198) in CA CRL Host input field.
Click on Sign button. Your Signed certificate will be created within few seconds.
Click on OK button to close New Certificate window.
If newly created CA certificate does not show T flag or Trusted property shows no, double click on your CA certificate and click on Trusted checkbox located at the bottom of General tab and then click on Apply and OK button.

Creating Server Certificate

Next--> Server certificate .

Click on PLUS SIGN (+) again. New Certificate window will appear.
Put your server certificate name (for example: Server) in Name input field. Also put a certificate common name (for example: Server) in Common Name input field.
If you have put any optional field in CA certificate, put them here also.
Click on Key Usage tab and uncheck all checkboxes except digital signature, key encipherment and tls server checkboxes.
Click on Apply button and then click on Sign button. Sign window will appear now.
Your newly created Server certificate template will appear in certificate dropdown menu. Select newly created certificate template if it is not selected.
Also select CA certificate from CA dropdown menu.
Click on Sign button. Your Signed certificate will be created within few seconds.
Click on OK button to close New Certificate window.
Double click on your server certificate and click on Trusted checkbox located at the bottom of General tab and then click on Apply and OK button.

Creating Client Certificate

The following steps will show how to create client certificate in MikroTik RouterOS.

Click on PLUS SIGN (+) again. New Certificate window will appear.
Put your client certificate name (for example: Client) in Name input field. Also put a certificate common name (for example: Client) in Common Name input field.
If you put any optional field in CA certificate, put them here also.
Click on Key Usage tab and uncheck all checkboxes except tls client checkbox.
Click on Apply button and then click on Sign button. Sign window will appear now.
Your newly created Client certificate template will appear in certificate dropdown menu. Select your newly created certificate template if it is not selected.
Also select CA certificate from CA dropdown menu.
Click on Sign button. Your Signed certificate will be created within few seconds.
Click on OK button to close New Certificate window.

NOW...Export all certs by going to files select and copy the certs,createa folder locally and cpy/paste them in there.
NEXT STEP--> OpenVPN Server Configuration in MikroTik Router
Click on PPP menu item from Winbox and then click on Interface tab.
Click on OVPN Server button. OVPN Server window will appear.
Click on Enabled checkbox to enable OpenVPN Server.
Put your desired TCP Port (example: 443) on which you want to run OpenVPN Server in Port input field.
Make sure ip option is selected in Mode dropdown menu.
From Certificate dropdown menu, choose server certificate that we created before. Also click on Require Client Certificate checkbox.
From Auth. Panel, uncheck all checkboxes except sha1.
From Cipher panel, uncheck all checkboxes except aes 256.
Now click on Apply and OK button.(OpenVPN Server is now running in MikroTik Router)
 Creating OpenVPN Users
MikroTik OpenVPN uses username and password to validate legal connection. So, we have to create username and password to allow any user. The complete user configuration for OpenVPN Server can be divided into three parts.

IP Pool Configuration
User Profile Configuration and
User Configuration
IP Pool Configuration

Usually multiple users can connect to OpenVPN Server. So, it is always better to create an IP Pool from where connected user will get IP address. The following steps will show how to create IP Pool in MikroTik Router.

From Winbox, go to IP > Pool menu item. IP Pool Window will appear.
Click on PLUS SIGN (+). New IP Pool window will appear.
Put a meaningful name (vpn_pool) in Name input field.
Put desired IP Ranges (192.168.2.2-192.168.2.250 Use yours these are just dummies for the example) in Addresses input filed. Make sure not to use VPN Gateway IP (192.168.2.1) and the last IP (192.168.2.154) because last IP will be used as DHCP Server IP.
Click Apply and OK button.

User Profile Configuration

After creating IP Pool, we will now configure profile so that all users can have similar characteristics. The following steps will show how to configure user profile for OpenVPN User.

From Winbox, go to PPP menu item and click on Profile tab and then click on PLUS SIGN (+). New PPP Profile window will appear.
Put a meaningful name (vpn_profile) in Name input field.
Put VPN Gateway address (192.168.2.1) in Local Address input field.
Choose the created IP Pool (vpn_pool) from Remote Address dropdown menu.
Click Apply and OK button.

OpenVPN Users Configuration

After creating user profile, we will now create users who will be connected to OpenVPN Server. The following steps will show how to create OpenVPN users in MikroTik RouterOS.

From PPP window, click on Secrets tab and then click on PLUS SIGN (+). New PPP Secret window will appear.
Put username (For example: mischa123) in Name input field and put password in Password input field.
Choose ovpn from Service dropdown menu.
Choose the created profile from Profile dropdown menu.
Click on Apply and OK button.(Openvpn server user created)
NOW..timefor the fun part install opevpn OpenVPN Users Configuration
Go to https://openvpn.net/community-downloads/ pick the one suited for your os
Note: when installing select the following TAP Virtual Adapter, EasyRSA Mgmt scripts,GUI
After install open netwrok connections and enable TAP Windows adapter-V9 
Now take the certs you copied earlier and copy them to the config file(windows C:Programfiles/openvpn/config)
Rename the certs as follows
ca:     CA.crt
cert:   Client.crt
key:   Client.key 
Open the sample-config will be found where a sample OpenVPN Client configuration file named client.ovpn is provided. Copy this sample configuration file into config folder and then open the client configuration file with a text editor such as WordPad, NotePad ++ and add the required parameters to the file and save

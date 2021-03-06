# Why use VPN

 A Virtual Private Network (VPN) is a significant step in securing communication. It's a requirement in compliance with many standards such as HIPAA, NIST, ... etc.
VPN achieves the encryption of data in-transit or in-motion by creating a tunnel through which, data is transfered back and forth between the two parties of the tunnel.
For an attacker from the outside, the data will look Gubberish, nothing logical. VPN uses mutual authentication which means that each party in the communication is able to authenticate the other party to make sure the other party is who they claim to be. This is implemented by certificate authentication.


# Azure VPN
VPN has different protocols to be built upon. It also can be built on different layers of the OSI model. For example, IPSec works in layer 4 of the OSI model (Transport), whereas L2TP works in layer 2 (Data link) and OpenVPN works in layer 3 (Network). The steps needed to create a VPN in Azure follows:

# VPN Steps:

### i. create a virtual gateway 

### ii. insert the root certificate

### iii. download the client application

### iv. install a leaf certificate


# In-details

#### i. create a virtual gateway

A Virtual gateway is a service in Azure that is connected to a certain virtual network. Each virtual network can have only one virtual gateway which in the logical explanation acts as a router. This virtual gateway is the service responsible for encryption and decryption of the data. To create a VPN to connect to an existing virtual network we need first to add a gateway subnet, which this virtual gateway will take an IP address of that range.
We can add a gateway subnet to the network using Azure cli:

      az network vnet subnet create \
      --vnet-name VNet1 \
      -n GatewaySubnet \
      -g TestRG1 \
      --address-prefix 10.1.255.0/27
  
Note: without a gateway subnet, we'll not be able to create a virtual network.


We'll need to allocate a public IP address to the virtual gateway. We can do that through the following code:

      az network public-ip create \
      -n rg-vpn-pip \
      -g TestRG1  \
      --allocation-method Dynamic
 
 Note: We can either allocate that public IP address statically or dynamically. The dynamic choice will cost less than statically.
 
      az network vnet-gateway create \
      -n vgw-vpn \
      -l eastus \
      -g TestRG1 \
      --public-ip-address rg-vpn-pip \
      --vnet VNet1 \
      --gateway-type Vpn \
      --sku basic \
      --vpn-type RouteBased \
      --no-wait
 
 Note: Now the virtual gateway is created for the virtual network (VNet1). This means that all the resources that exist in that virtual network will be accessible to VPN clients, after the gateway is configured.
  

#### ii. insert a root certificate

This article assumes you have generated a self-signed cerficate that will be used as the root cerfiicate in the virtual gateway. This cerificate can be created using PowerShell in Windows or using OpenSSL in Linux. The root cerfiicate will be inserted in the configuration of the virtual gateway and clients will have a leaf certificate generated from this root certificate. 


![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/VPN/VPN%20gateway.PNG)



#### iii. Download and Install the VPN client

After inserting the root certificate in the virtual gateway, we need to download the VPN client and install on each client. 


![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/VPN/VPN-connect.PNG)


In the image below, you can find the VPN setting after the installtion of the VPN client file:


![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/VPN/VPN-settings.PNG)


#### iv. Install a leaf certificate

A plethora of certificates can be generated from one root certificate that so each client can have a certificate. To have a certificate for each client is best practice because each client certificate's can be revoked for any reason like staff leaving, access revocation without the need to regenerate and re-install a new certificate for each employee leaving the organization.


The image below shows how you are connected to the virtual gateway. Finally, your connection to the virtual network and the resources inside is encrypted and is immune to man-in-the-middle attack, congratulations.

![This is an image](https://github.com/Hazemwaddah/Azure_Security/blob/main/VPN/VPN-connected.PNG)



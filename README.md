![How to deploy a standalone DNS on a Windows Server 2022](https://github.com/jonathansantacruz3/How-to-Deploy-Standalone-DNS-on-Windows-Server-2022/assets/151465848/75a45bf5-1c0d-4ca5-8a6c-2cefaed1250f)

<h2>Environments and Technologies Used</h2>

- Microsoft Hyper-V (Virtual Machines/Compute)

<h2>Operating Systems Used </h2>

- Windows 11 Home Edition</b> (23H2)[HOST]
- Windows Server 2022 (21H2)
- Windows 11 Pro (23h2)

<h2>List of Prerequisites</h2>

- Hyper-V Enabled 
- Windows Server 2022 ISO file
- Windows 11 Pro ISO file

<h1>How to Deploy a Standalone DNS on a Windows Server 2022</h1>
In this project, I will detail the process of how to set up a standalone DNS (Domain Name Server) on a Windows Server 2022 operating System and use a Windows 11 Pro client to validate the DNS records configured. 

The first thing to point out is the topology in this diagram. I utilized a private virtual switch within Hyper-V. It is worth mentioning, that I also used an external virtual switch on both virtual machines to test their connectivity and succeeded. I named my server, Thanos, and it is the authoritative DNS for my domain galaxy.com. I also connected a client machine named StarLord running Windows 11 Professional. 

<h3>Why configure the DNS server’s IP address to static?</h3>

You must set your DNS server’s IP (internet protocol) address to static to ensure that your client PC can resolve domain names. If it can’t resolve domain names, then it can’t visit any of the organization’s websites based on the names you provide. Also, Active Directory Domain Services requires a DNS to be available in the network to function. Usually, a DHCP server (dynamic host configuration protocol) takes care of assigning IPs, but in enterprise environments where the network is isolated from external networks, and clients still need to access resources, a DNS server in the network fills this role and caches all the necessary domains and IPs. 
<br />



<h2>Installation Steps</h2>

<h3></h3>

<h3>How to set the DNS server IP to static?</h3>

Navigate to “Local Server” in the left panel of the Server Manager console. Then click on the Ethernet option. 
 
In the new window right-click on the Ethernet icon and choose “Properties”. Scroll down to the “Internet Protocol Version 4” option and double-click. By default, it will be set to automatically assign IPs using DHCP. If you want to discover what the current IP configurations of are of your network adapters, open the command line by typing CMD in the search bar. Then type “ipconfig” in the terminal and it should display the current IPv4 address, subnet mask, and the default gateway. 
     
Enter the following configuration and enter the new static IP address. For the alternate DNS server IP address, you can input 8.8.8.8 for Google’s server. Then press “OK”.
 

<h3>Install the DNS role in Server Manager:</h3>
While in the Server Manager console head over to the “Dashboard” panel and click on “Add roles and features”. This will lead you into the Wizard to configure this feature. In the first window it will suggest what should be done to optimize security and performance. Then choose the “role-based” installation type, after hitting next. The next window will display the IP address configuration set for the server and the name of the machine. If there are any discrepancies such as the APIPA (automatic IP addressing) address, then you need to go back and configure properly.
 
  
In this window, add the DNS role by checking the box and its management features in the box that pops up by clicking the “Add feature" button. Click “Next” when it asks if you want to add any additional features and “Next” again. 
  
In this window, it gives you the option to check the box if you want the machine to restart automatically if required. By default, it will be unchecked and should remain this way, especially if the server is a production machine that needs to be available around the clock. You have control of when to perform maintenance on the computer. Click on “Install” and it will begin the process. After the installation, you should the DNS feature on the left panel of the Server Manager console.
 
 
 
NOTE: This link at the bottom gives you the option to export the configuration settings of this installation and deploy it through PowerShell on another machine. 
 

The next step is to head over to the DNS panel and open its tools in the top right section of the console. Click on DNS to open its snap-in manager. This is where we can add lookup zones (IP addresses and domain names) and make this DNS the authoritative server for the domain galaxy.com. This DNS server will store and cache any records in the network for quick use. 
  

<h3>Create the primary forward zone:</h3>
In the DNS snap-in console, create a new forward lookup zone by right-clicking it and choosing “New Zone”. This will bring up the Wizard, and then press “Next”. You want to choose the “Primary zone” option to establish the server as an authoritative source. It will then ask you to name the zone for the environment, which in this case, is asking you to name the domain this DNS resides in. I decided to name my name space galaxy.com. After hitting “Next”, it will create a file to store the records. In the next window, it gives you options on how the records will be stored in the zone. You will choose to manually update records since this is a standalone network. The next screen is a summary of the new zone. After clicking on “Finish”, the new folder in the forward lookup zone is populated with the new records. 
      
<h3>Name server (NS) and start of authority (SOA) validation configuration:</h3>
Go into the first SOA record by right clicking and selecting “Properties”. Navigate to the NS tab to set it and validate by choosing the “Edit” button to create a connection to a name server. Click on “Resolve” and it should find that it is a proper name server for the DNS zone. These addresses will provide resolutions for any future queries.  Click on “OK”, then “Apply” and “OK”. 
   


<h3>Create an A record to help identify the DNS server in the network as a host:</h3>

In the same forward lookup zone, right-click on any open space to choose “New Host (A or AAAA)” to create an A record. Enter the name of the machine name, which in my case, is DNS-THANOS. At the same time, it will generate the fully qualified domain name (FQDN) with the domain added after the computer name. The IP address will be the static address we configured earlier. Click on “Add Host”, and then “Done”. 
  

<h3>Create the primary reverse lookup zone:</h3>

A reverse lookup zone will enable an IP query to resolve to a domain name. In other words, if a user wants to look up a server with its IP address, it will return the name of the server as well. To create this, go to the DNS tools in the DNS panel. Click on DNS-THANOS to display the “Reverse Lookup Zone” folder. Right click on it and choose “New zone”. It will guide you through the Wizard again and since I am only working with IPv4 in this project and it is the authoritative DNS, I will select the “Primary zone” and “IPv4 Reverse lookup” option. The network ID will be a /24 so the first 3 octets are inputted in the field. The FQDN will be displayed backwards because of the way the machine interprets it. For the same reason as the forward lookup zone, we will set it to manually update any records in the future. 
            
Once finished, it will create new SOA and NS records. You will then resolve the records to validate them. Click on the NS record and then the “Resolve button within the “Edit” button. Click “OK”, “Apply, then “OK”. 
  
   
<h3>Creating Pointer (PTR) Records (Domain Name to IP address):</h3>
In the reverse lookup zone, right-click an open space and choose “New Pointer (PTR)” from the list of options that populate. It will ask for the IP address, which you will type the address of the DNS server. Then it will ask for the name of the record that you can browse and appoint from the forward lookup zone. 		
   

Now that the pointer records are created the server can be resolved by its domain name or its IP address. Now we can link a client to this server to access resources. 

<h3>Validate DNS record using a client PC:</h3>
If you open the command line and type “nslookup dns-thanos.galaxy.com 192.168.0.51” it should find the DNS server, however, there is no record for StarLord. This means that if you query the DNS for the client computer, nothing will appear. To update this, let’s go back to DNS-Thanos under the DNS panel and choose to create a new A record. In my case, my client's PC is named StarLord with an IP address 192.168.0.100. The FQDN will be created and by checking the box to create a PTR record you will also have a reverse lookup zone record. Now, if you open the command line again and type in “nslookup Starlord.galaxy.com  192.168.0.51”, it should respond with the name of the server, client, and their respective addresses. The reason you want to use nslookup with the server’s IP address is because that computer is where all the records are stored. 
 



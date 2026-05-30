# Active Directory Home Lab (Oracle VirtualBox)

## Objective
Built an on-premises Active Directory environment from scratch using two
virtual machines in Oracle VirtualBox. Configured a Windows Server 2022
Domain Controller providing AD DS, DNS, DHCP, and NAT/routing, then joined
a Windows 10 client to the domain and bulk-created users via PowerShell.

## Environment & Tools
- Oracle VirtualBox (hypervisor)
- Windows Server 2019 (Domain Controller)
- Windows 10 (client)
- PowerShell

## Network Topology
Internet → [Server: NIC1 (NAT)] 
                [Server: NIC2 (Internal Network)] → [Windows 10 Client]
The Server routes the client's traffic to the internet and hands out
IP addresses via DHCP.

## Steps

### 1. Install VirtualBox and Create the Server VM
Created a VM for the Domain Controller and gave it two network adapters: Adapter 1
set to NAT (for internet access) and Adapter 2 set to Internal Network (the private
LAN shared with the client). Two adapters are required because the server needs to
live on both the internet-facing side and the isolated internal side at the same
time — this is what makes it the gateway between the client and the internet.

### 2. Install Windows Server 2022
Mounted the Windows Server 2022 ISO and installed the OS, choosing the "Desktop Experience" edition so the server has a graphical interface. This makes it possible to manage Active Directory, DHCP and DNS visually through Server Manager rather than entirely from the command line.

### 3. Set a Static IP on the Internal Adapter
Assigned a fixed IP address to the internal adapter and pointed the server's DNS at itself. A Domain Controller must have a static (unchanging) IP because every client depends on it for DNS and DHCP — if its address changed, clients would lose track of it. It points to itself for DNS because, once Active Directory is installed, this server is the DNS server for the entire domain.


### 4. Install Active Directory Domain Services & Promote to DC
Added the AD DS role through Server Manager, then ran the promotion wizard to create a new forest and domain (e.g. mydomain.com). Installing the role only copies the AD software onto the machine; promoting it is what actually turns the server into a live Domain Controller — it builds the directory database and installs DNS automatically. After this step, the server is the central authority for the domain.

### 5. Create a Domain Admin Account
Created a dedicated administrator account inside an Organizational Unit (OU), separate from the built-in Administrator. Using a named admin account is good security practice: it keeps actions traceable to a specific person and mirrors how real organizations manage privileged access rather than relying on the default account.

### 6. Install RAS / NAT (Routing and Remote Access)
Installed the Remote Access role and configured NAT so the isolated client can reach the internet through the server. The client has no internet path of its own — it sits only on the internal network. RAS/NAT turns the server into a router that takes the client's internet-bound traffic from the internal adapter and forwards it out through the NAT adapter, then relays the responses back.

### 7. Install and Configure DHCP
Added the DHCP role and created a scope — a pool of IP addresses the server is allowed to hand out, along with the gateway and DNS settings clients should use. This means any client that joins the internal network is automatically given a valid IP address and the information it needs to find the domain, with no manual configuration required.


### 8. Bulk-Create Users with PowerShell
Ran a PowerShell script to generate roughly 1,000 user accounts in Active Directory. Creating that many users by hand would take days; the script reads a list of names and creates them all in seconds. This demonstrates automation — a core sysadmin skill used for the kind of repetitive provisioning real IT teams handle constantly. (The script's
execution policy had to be allowed first, since Windows blocks unsigned scripts by default.) Script credited below and stored in /scripts.

### 9. Create the Windows 10 Client VM and Join the Domain
Built a Windows 10 Pro client VM on the same Internal Network, confirmed it
automatically received an IP address from the server's DHCP (verified with
ipconfig), then joined it to the domain. Windows 10 Pro is required because only Pro and higher can join a domain — Home edition cannot. The ipconfig check is important: seeing a valid IP from the DHCP scope proves the networking, DHCP, and DNS are all working before attempting the join.

### 10. Log In as a Domain User
Logged into the Windows 10 client using one of the accounts created by the PowerShell script. A successful login confirms the entire chain works end to end: the client found the Domain Controller over the internal network, DNS resolved the domain, the scripted account is valid and authentication succeeded.

## What I Learned
- How AD DS, DNS and DHCP work together in a domain environment
- Configuring NAT/routing so an isolated internal network reaches the internet
- Bulk user provisioning with PowerShell
- (Add any troubleshooting you actually hit — this is the most valuable part)

## Credits
PowerShell user-generation script by Josh Madakor (github.com/joshmadakor1/AD_PS).

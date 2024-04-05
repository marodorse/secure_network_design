# secure_network_design

Client Requirements:

Active Directory and DNS server
DHCP server
DMZ concept implemented through VLANs and access control lists (ACLs) (firewall alternative)
iSCSI storage server
Four network sectors:
Management/Secretariat (5 workstations)
Study (8 workstations)
Production (10 workstations)
Support (2 sectors, 10 workstations each)

# 
### NETWORK DESIGN FOR SKM company. 

_Architecture implemented_ : hierarchical 3 tier with star topology.  

- Core layer : network backbone, high speed data transport, fast packet forwarding) 

- Distribution layer : here routing filtering and policy enforcment services are implemented, it combines the serparated traffic from the access layer. Vlan segmentation and ACL are configured here 

- Access layer : entry point for end devices, brings together the traffic from all the access points. 

**Why ?**  

Scalability, simplified management, enhanced security ( granular control )  

network diagram with annotations :   
![diagram](https://github.com/marodorse/secure_network_design/assets/34199422/2eede354-9590-4563-a9c1-44054927b27d)

IP Addressing Per sector and vlan :   
![image](https://github.com/marodorse/secure_network_design/assets/34199422/27d91246-e8c9-4adc-921d-8af44080e4c3)

# key devices configurations : 

mainswitchML#show running-config
Building configuration...
Current configuration : 5363 bytes
!
version 16.3.2
no service timestamps log datetime msec
no service timestamps debug datetime msec
service password-encryption
!
hostname mainswitchML
!
!
enable password 7 0836495C061A0E
no ip cef
ip routing
!
no ipv6 cef
username skm password 7 0836495C061A0E
ip ssh version 2
no ip domain-lookup
ip domain-name ourcreation.org
spanning-tree mode pvst
interface GigabitEthernet1/0/1
no switchport
ip address 172.16.3.145 255.255.255.252
duplex auto
speed auto
!
interface GigabitEthernet1/0/2
switchport mode trunk
ip access-group 101 in
ip access-group 101 out
!
interface GigabitEthernet1/0/3
switchport mode trunk
!
interface GigabitEthernet1/0/4
switchport mode trunk
!
interface GigabitEthernet1/0/5
switchport mode trunk
!
interface GigabitEthernet1/0/6
switchport mode trunk
!
interface GigabitEthernet1/0/7
switchport trunk allowed vlan 10-14,20
switchport mode trunk
interface Vlan1
!
router ospf 10
router-id 1.1.1.1
log-adjacency-changes
network 172.16.1.0 0.0.0.127 area 0
network 172.16.1.128 0.0.0.127 area 0
network 172.16.2.0 0.0.0.127 area 0
network 172.16.2.128 0.0.0.127 area 0
network 172.16.3.0 0.0.0.127 area 0
network 172.16.3.128 0.0.0.15 area 0
network 172.16.3.144 0.0.0.3 area 0
!
ip classless
ip route 0.0.0.0 0.0.0.0 GigabitEthernet1/0/1 
!
ip flow-export version 9
!
!
access-list 101 deny ip 172.16.1.0 0.0.0.127 
172.16.1.128 0.0.0.127
access-list 101 deny ip 172.16.1.0 0.0.0.127 172.16.2.0 
0.0.0.127
access-list 101 deny ip 172.16.1.0 0.0.0.127 
172.16.2.128 0.0.0.127
access-list 101 deny ip 172.16.1.0 0.0.0.127 172.16.3.0 
0.0.0.127
access-list 101 deny ip 172.16.1.0 0.0.0.127 
172.16.3.128 0.0.0.127
access-list 101 deny ip 172.16.1.0 0.0.0.127 
172.16.3.128 0.0.0.15
access-list 102 deny ip 172.16.1.128 0.0.0.127 
172.16.1.0 0.0.0.127
access-list 102 deny ip 172.16.1.128 0.0.0.127 
172.16.2.0 0.0.0.127
access-list 102 deny ip 172.16.1.128 0.0.0.127 
172.16.2.128 0.0.0.127
access-list 102 deny ip 172.16.1.128 0.0.0.127 
172.16.3.128 0.0.0.127
access-list 102 deny ip 172.16.1.128 0.0.0.127 
172.16.3.0 0.0.0.127
access-list 103 deny ip 172.16.2.0 0.0.0.127 172.16.1.0 
0.0.0.127
access-list 103 deny ip 172.16.2.0 0.0.0.127 
172.16.1.128 0.0.0.127
access-list 103 deny ip 172.16.2.0 0.0.0.127 
172.16.2.128 0.0.0.127
access-list 103 deny ip 172.16.2.0 0.0.0.127 172.16.3.0 
0.0.0.127
access-list 103 deny ip 172.16.2.0 0.0.0.127 
172.16.3.128 0.0.0.15
access-list 104 deny ip 172.16.2.128 0.0.0.127 
172.16.1.0 0.0.0.127

# Security features :

implementation of :    

- no ip domain look-up command ->
- 
  prevents information leakage where wrong commands could be interpretated as DNS queries, revealing internal network details to external DNS servers. This command allows mistyped stuff to be treated as      errors with no DNS resolution attempt. Reduces also attack surface (no dns spoofing or dns responses manipulation are a potential attack vector. This is a good network security hygiene.

- line console 0 ->

secures and manages a cisco device, so when you want to physically connect to a device, and adding a password means the person is prompted to enter a password before being granted access to the devices command line interface. Here security is implemented by reducing risks of unauthorized access ( but password can be guessed through brute-force attack to an extend)

- RADIUS service ->
  
RADIUS (Remote Authentication Dial-In User Service) is a networking protocol for centralized authentication, authorization, and accounting (AAA). It verifies user identities, controls access to network resources, and logs user activity. RADIUS is widely used in enterprise and ISP networks, providing a scalable and secure solution. It centralizes user management, enforcing access policies and tracking network usage. RADIUS operates at the application layer, typically using UDP for communication. It offers encryption and message integrity to protect sensitive data. RADIUS servers authenticate users against a centralized database and determine access rights based on predefined policies. It facilitates seamless authentication for various network devices, including routers, switches, RADIUS enhances network security and simplifies user management across distributed environments. BUT it is also OUT DATED as it works with telnet. 

- SSH version 2 ->
  
with 4096 bits RSA key for encryption, SSH (Secure Shell) is a network protocol used for secure remote access to devices over a network. It provides encrypted communication between client and server, ensuring confidentiality and integrity of data. SSH facilitates secure login and command execution on remote systems. It operates on port 22 by default and offers authentication using passwords or cryptographic keys. SSH is widely adopted for managing network devices, servers, and cloud instances. It supports various authentication methods and allows secure file transfer (SFTP) and port forwarding. SSH enhances network security by protecting against eavesdropping and unauthorized access. It enables secure administration and communication in distributed environments, improving overall system reliability and integrity.

- NAT ->
  
provides a level of security by hiding the internal IP addresses of devices on the local network from external sources, making it more difficult for attackers to directly target individual devices, it translates private addresses into public ones.
 
- VLANs and ACLs implemented in DMZ concept ->

A DMZ  is a network design strategy that involves creating a buffer zone between an organization's internal network, a “safe” network, and external “unsafe” networks, such as the internet. 
The DMZ serves as a segregated area where our  public-facing services and resources are hosted,here the DNS server with its DNS and HTTP services.
This provides an additional layer of security to protect sensitive internal data from external threats.The segmentation provided by VLANs isolates critical data from potential threats, ACLs provide granular control over traffic flow and access permissions, reducing the attack surface and protecting against unauthorized access. 
ACLs are applied to the interfaces connecting the VLANs to enforce access control policies and regulate traffic flow between network segments. ACLs specify which types of traffic are allowed or denied based on predefined criteria, such as source and destination IP addresses, ports, or protocols. 
 ACLs are used to filter traffic entering and exiting the DMZ, allowing only authorized traffic to reach public-facing services while blocking or restricting access to sensitive resources in the internal network. For example, ACLs may permit inbound web traffic to reach web servers in the DMZ while blocking direct access to internal databases or file servers. NAT -> NAT provides a level of security by hiding the internal IP addresses of devices on the local network from external sources, making it more difficult for attackers to directly target individual devices.




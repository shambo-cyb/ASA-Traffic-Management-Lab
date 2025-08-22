# ASA-Traffic-Management-Lab
Configured Cisco ASA firewall in GNS3 with inside, outside, and DMZ networks, implementing NAT, DHCP, and ACLs to manage and monitor traffic.

## Tools used 
- GNS3 with Cisco ASA Device IOs configured
- VMWare for virtual machines 

## Topology Image 
<img width="600" height="680" alt="1" src="https://github.com/user-attachments/assets/f7f286a9-01c9-4ad6-a0e1-25999fab9be6" />

#### Description of the environment 
This project sets up a simulated enterprise network in GNS3 using a Cisco ASA firewall.
It demonstrates how inside, outside, and DMZ networks interact securely.
The environment mimics a real-world perimeter defense lab.
- Inside Network: A Windows 10 client connected via SW1, receiving IPs dynamically from ASA’s DHCP.
- Firewall (ASA-1): Configured with inside, outside, and DMZ interfaces, NAT rules, ACLs, and DHCP services.
- DMZ: A webserver hosted behind R2, reachable from outside via static NAT.
- Perimeter Router (R1): Connects ASA to the external network and routes traffic.
- Outside Network: Simulated external server accessed through the ASA’s outside interface

## Config for the Lab 
### R2 Configuration (DMZ webserver) 
```bash
ip route 0.0.0.0 0.0.0.0 172.16.4.1
ip http server
 ```
### ASA Configuration 
```bash
interface GigabitEthernet0
nameif outside
ip address 192.139.14.1 255.255.255.0
no shutdown
interface GigabitEthernet1
nameif inside
security-level 100
ip address 192.168.1.1 255.255.255.0
no shutdown
interface GigabitEthernet2
nameif DMZ
security-level 50
ip address 172.16.4.1 255.255.255.0
no shutdown
```
### Assigning a deafault route
```bash
route outside 0.0.0.0 0.0.0.0 192.139.14.9
```
### Configure Network Address Translation
```bash
object network inside-net1
subnet 192.168.1.0 255.255.255.0
nat (inside,outside) dynamic interface
exit
object network inside-net2
subnet 192.168.1.0 255.255.255.0
nat (inside, dmz) dynamic 172.16.4.3
exit
object network DMZ-Server
host 172.16.4.8
nat (dmz,outside) static 192.139.14.8
exit
```
### After assigning the dhcp pool preferably for my name I am able to see the PC getting the ip address by the DHCP Server
<img width="600" height="436" alt="Slide2" src="https://github.com/user-attachments/assets/a82716d6-cf14-460d-9ba0-5a33827814d7" />

### Xlate and Conn 

<img width="600" height="176" alt="Slide3" src="https://github.com/user-attachments/assets/50be849a-4555-4199-858c-dda9a2189b3b" />

<img width="600" height="387" alt="Slide4" src="https://github.com/user-attachments/assets/bbce2970-1d95-415f-b6fb-36290ccdc0c7" />

The commands show xlate and show conn reveal that two internal devices are connected to the outside internet, and the firewall is using Port Address Translation (PAT) to share a single public IP address. The show log output then confirms the connections and even shows a denied ICMP packet, which is the firewall blocking a ping attempt.


### Cisco ASA Log and ACL Analysis
<img width="600" height="749" alt="Slide8" src="https://github.com/user-attachments/assets/6bf17c8f-7010-4f52-9a2a-cfccfb7129e7" />

This image provides a detail of log and ACL analysis on a Cisco ASA firewall. The process began with clearing the logging buffer to focus on recent events, followed by a review of the current log file and the configured access lists.
- The log output confirms the successful establishment and subsequent normal teardown of a TCP connection from an outside host (200.4.4.5) to a host on the DMZ network (172.16.4.8).
- An entry with message ID ASA-4-106023 indicates a denied ICMP packet by the firewall. This is consistent with a deny-all rule often found at the end of an ACL.
- The show access-list command reveals the existence of an inbound ACL named "ACLInbound" that explicitly permits Telnet traffic to the DMZ host (172.16.4.8). However, the previous log entry shows a Telnet attempt was denied, indicating that a more specific, higher-priority deny rule or an implicit deny-all rule is taking effect before this permit rule is matched.




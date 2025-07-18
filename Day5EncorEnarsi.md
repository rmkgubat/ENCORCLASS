''TASK1: BE A NETWORK ENGINEER.''

CONFIG T
no logging console

telco1:
config t
IP ROUTE 0.0.0.0  0.0.0.0 208.8.8.2
ip name-server 8.8.8.8 1.1.1.1
ip domain lookup
int gi 1
no shut
ip add 208.8.8.11 255.255.255.0
Int Gi 2
no shut
ip add 192.168.102.11 255.255.255.0
int gi 3
no shut
ip add 192.168.103.11 255.255.255.0
line vty 0 14
 transport input all
 exec-timeout 0 0
do wr
do sh ip int brief


telco2:
config t
IP ROUTE 0.0.0.0  0.0.0.0 208.8.8.2
ip name-server 8.8.8.8 1.1.1.1
ip domain lookup
int gi 1
no shut
ip add 208.8.8.12 255.255.255.0
Int Gi 2
no shut
ip add 192.168.102.12 255.255.255.0
int gi 3
no shut
ip add 192.168.103.12 255.255.255.0
line vty 0 14
 transport input all
 exec-timeout 0 0
do wr
do sh ip int brief


telco3:
config t
IP ROUTE 0.0.0.0  0.0.0.0 208.8.8.2
ip name-server 8.8.8.8 1.1.1.1
ip domain lookup
int gi 1
no shut
ip add 208.8.8.13 255.255.255.0
Int Gi 2
no shut
ip add 192.168.102.13 255.255.255.0
int gi 3
no shut
ip add 192.168.103.13 255.255.255.0
line vty 0 14
 transport input all
 exec-timeout 0 0
do wr
do sh ip int brief

TASK1: HSRP WITH IP SLA.
GI1:108.5                        GI1:108.6
APPS1                             APPS2
GI2:102.5                        GI2:102.6
             HSRPIP:  102.7
GI3:103.5                        GI3:103.6
             HSRPIP:  103.7
          use standby 3 and track 3

telco1:PrimaryInt:
config t
Track 2 Int gi 1 line-protocol
Int gi 2
standby version 2
standby 2 ip 192.168.102.7
standby 2 preempt
standby 2 Priority 110
standby 2 track 2 decrement 70
END

telco2: Backup:
config t
Int gi 2
standby version 2
standby 2 ip 192.168.102.7
standby 2 preempt
standby 2 Priority 100
END

telco3: BackupOfBackup:

config t
Int gi 2
standby version 2
standby 2 ip 192.168.102.7
standby 2 preempt
standby 2 Priority 60
END

CONFIGURE REAL IP SLA: Service level agreement:

@Telco1:
config t
no ip sla 4
ip sla 4 
  icmp-echo 8.8.8.8
  freq 5
  exit
ip  sla  schedule 4 life forever start-time now
track  4  ip sla 4
  delay down 10 up 5
  exit
Int gi 2
NO standby 2 track 2 decrement 60
standby 2 track 4 decrement 60

@Telco2:
config t
no ip sla 4
ip sla 4 
  icmp-echo 8.8.8.8
  freq 5
  exit
ip  sla  schedule 4 life forever start-time now
track  4  ip sla 4
  delay down 10 up 5
  exit
Int gi 2
standby 2 track 4 decrement 50

GLBP with load balancing:

telco1:
config t
int gi3
glbp 10 ip 192.168.103.7
glbp 10 priority 120
glbp 10 preempt
glbp 10 load-balancing round-robin
glbp 10 timers 1 3
glbp 10 weighting 110 lower 90 upper 100
glbp 10 weighting track 4 decrement 15
!if R1 can’t reach 8.8.8.8, its weighting 
!drops below 100 and it relinquishes its Active role.

config t
! IP SLA config to monitor Internet
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet1
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

int gi3
! Link GLBP weighting to tracking
glbp 10 weighting 120 lower 100 upper 120
glbp 10 weighting track 1 decrement 20


telco2:
config t
int gi3
glbp 10 ip 192.168.103.7
glbp 10 priority 110
glbp 10 preempt
glbp 10 load-balancing round-robin
glbp 10 timers 1 3
glbp 10 weighting 110 lower 90 upper 100
glbp 10 weighting track 4 decrement 15
!if R1 can’t reach 8.8.8.8, its weighting 
!drops below 100 and it relinquishes its Active role.

config t
! IP SLA to monitor Internet
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet1
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

! Link GLBP weighting to tracking
int gi3
glbp 10 weighting 110 lower 90 upper 110
glbp 10 weighting track 1 decrement 20


telco3:
config t
int gi3
glbp 10 ip 192.168.103.7
glbp 10 priority 100
glbp 10 preempt
glbp 10 load-balancing round-robin
glbp 10 timers 1 3

config t
! IP SLA to monitor Internet
ip sla 1
 icmp-echo 8.8.8.8 source-interface GigabitEthernet1
 frequency 5
ip sla schedule 1 life forever start-time now

track 1 ip sla 1 reachability

! Link GLBP weighting to tracking
int gi 3
glbp 10 weighting 100 lower 80 upper 100
glbp 10 weighting track 1 decrement 20


show glbp
show glbp brief
show standby
show ip sla statistics

TASK3: NET FLOWS:

config t
interface GigabitEthernet3
 description NetFlow monitored interface
 ip flow monitor s3NETFLOW-MONITOR input
 ip flow monitor s3NETFLOW-MONITOR output

flow record s1NETFLOW-RECORD
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 collect counter bytes
 collect counter packets

flow exporter s2NETFLOW-EXPORTER
 destination 192.168.103.1
 transport udp 2055
 export-protocol netflow-v9

flow monitor s3NETFLOW-MONITOR
 record s1NETFLOW-RECORD
 exporter s2NETFLOW-EXPORTER

QOS: for https,ssh and telnet:
config t
! Define class-maps to match traffic types
class-map match-any PRIORITY-HTTPS-SSH
 match protocol https
 match protocol ssh

class-map match-any LOW-PRIORITY-TELNET
 match protocol telnet

! Define policy-map to assign priorities and bandwidth guarantees
policy-map QoS-PRIORITY
 class PRIORITY-HTTPS-SSH
  priority percent 30       ! Strict priority queue with guaranteed 30% bandwidth
 class LOW-PRIORITY-TELNET
  bandwidth percent 5       ! Reserve 5% bandwidth for Telnet (low priority)
 class class-default
  fair-queue                ! Default treatment for other traffic

! Apply the policy to the WAN interface (example: GigabitEthernet0/0)
interface GigabitEthernet0/0
 service-policy output QoS-PRIORITY




NAT/PAT:
sudo su
ifconfig eth0 192.168.103.100 netmask 255.255.255.0 up
route add default gw 192.168.103.11
ping 192.168.103.11

STEP1: define INSIDE AND OUTSIDE:
STEP2: create Access-list to permit IP of Inside:
STEP3: create a NAT pool with overload
@vpnPH:
config t
int gi 1
ip nat OUTSIDE
int gi 2
ip nat INSIDE
int gi 3
ip nat INSIDE
no access-list 8
access-list 8 permit 192.168.102.0 0.0.0.255
access-list 8 permit 192.168.103.0 0.0.0.255
ip nat inside source list 8 interface Gi 1 overload
ip nat inside source static 192.168.103.21 208.8.8.51
ip nat inside source static 192.168.103.22 208.8.8.52
end
FW-VPN-PH#show ip nat translations 
Pro  Inside global         Inside local          Outside local         Outside global
---  192.168.108.88        192.168.103.12        ---                   ---
---  192.168.108.69        192.168.103.11        ---                   ---
icmp 192.168.108.69:57608  192.168.103.11:57608  1.1.1.1:57608         1.1.1.1:57608
icmp 192.168.108.69:57864  192.168.103.11:57864  8.8.8.8:57864         8.8.8.8:57864
icmp 192.168.108.88:56840  192.168.103.12:56840  1.1.1.1:56840         1.1.1.1:56840
icmp 192.168.108.88:57096  192.168.103.12:57096  8.8.8.8:57096         8.8.8.8:57096
icmp 192.168.108.88:56584  192.168.103.12:56584  8.8.4.4:56584         8.8.4.4:56584
Total number of translations: 7

CREATING A WEB PROXY OR HIDING BEHIND NAT:
www.sti.edu.ph:      vs     www.dlsu.edu.ph:
@EDGE:
config t
no access-list 8
access-list 8 permit any
Int Gi 3
 ip nat Inside
Int gi 1
 ip nat Outside
IP Nat inside source static tcp 192.168.103.21 80 208.8.8.101 8080
IP Nat inside source static tcp 192.168.103.21 443 208.8.8.101 8443
IP Nat inside source static tcp 192.168.103.21 22 208.8.8.101 8069
IP Nat inside source static tcp 192.168.103.22 80 208.8.8.102 8080
IP Nat inside source static tcp 192.168.103.22 443 208.8.8.102 8443
IP nat inside source list 8 int gi 1 Overload
end
show ip nat translation

victim: nmap -v 200.0.0.100+k

ExamLab Training: Make a PortAddressTranslation:
10.m.1.9 22 --> 200.0.0.100+m 3022 =
10.m.1.10 80 --> 200.0.0.100+m 8088 = 
10.m.1.11 53 --> 200.0.0.100+m 4053 =
config t
IP Nat inside source static tcp 10.12.1.9 22 200.0.0.112 3022
IP Nat inside source static tcp 10.12.1.10 80 200.0.0.112 8088
IP Nat inside source static tcp 10.12.1.11 53 200.0.0.112 4053
do sh ip nat translation

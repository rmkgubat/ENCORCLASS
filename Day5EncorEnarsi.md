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

nterface GigabitEthernet1
 description NetFlow monitored interface
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 ip flow monitor NETFLOW-MONITOR input
 ip flow monitor NETFLOW-MONITOR output

flow record NETFLOW-RECORD
 match ipv4 source address
 match ipv4 destination address
 match transport source-port
 match transport destination-port
 collect counter bytes
 collect counter packets
 collect timestamp sys-uptime first
 collect timestamp sys-uptime last

flow exporter NETFLOW-EXPORTER
 destination 192.168.10.2
 transport udp 2055
 export-protocol netflow-v9

flow monitor NETFLOW-MONITOR
 record NETFLOW-RECORD
 exporter NETFLOW-EXPORTER

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

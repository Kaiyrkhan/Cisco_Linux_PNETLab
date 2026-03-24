# Configure NTP Server and Client on Multiple Platforms

1) Linux (Debian, Ubuntu, Rocky);
2) Cisco IOS (Router, Switch);
3) Huawei VRP (Router, Switch).

## NTP Server using Chrony on Linux

**🖧 Желі топологиясы**  
![Topology](https://raw.githubusercontent.com/Kaiyrkhan/Linux_Administration_101/main/Topology/Topology_interVLANRouting_NAT_Linux.png)  

**Chrony пакетін орнату**
```shell
Debian/Ubuntu/Rocky/Oracle
Package атауы: chrony

Debian/Ubuntu
Daemon/Service атауы: chrony немесе chronyd

RHEL/Rocky/Oracle
Daemon/Service атауы: chronyd

chronyd – the actual daemon to sync and serve via the Network Time Protocol
chronyc – command-line interface for the chrony daemon
```

```shell
Debian/Ubuntu
$ sudo apt update 
$ sudo apt install chrony

RHEL/Rocky/Oracle
$ sudo dnf install chrony

$ sudo systemctl status chronyd

$ ss -tulpn
$ netstat -tulpn
```

**Қазақстандық NTP серверлерге талдау жасау**
```shell
$ dig +short 2.kz.pool.ntp.org
185.116.194.200
82.200.233.67
38.180.36.242
$ ping -c2 185.116.194.200
64 bytes from 185.116.194.200: icmp_seq=1 ttl=54 time1.98 ms
64 bytes from 185.116.194.200: icmp_seq=1 ttl=54 time2.24 ms

$ dig +short ntp.nic.kz
80.241.0.72
$ ping -c2 80.241.0.72
64 bytes from 80.241.0.72: icmp_seq=1 ttl=53 time15.6 ms
64 bytes from 80.241.0.72: icmp_seq=1 ttl=53 time16.2 ms
```
> NTP Pool Time Servers Link: https://www.ntppool.org/zone/kz  

**NTP серверді конфигурациялау**
```shell
Уақыт белдеуін (Time Zone) өзгерту
$ sudo timedatectl set-timezone Asia/Almaty

$ timedatectl status
```
> Time Zones in Kazakhstan https://www.timeanddate.com/time/zone/kazakhstan  

```shell
RHEL/Rocky/Oracle
$ sudo vi /etc/chrony.conf
#pool 2.rocky.pool.ntp.org iburst        // артық DNS атауларды "#" Comment-ге алып, төменгі қатарға Қазақстанға ең жақын NTP сервердің DNS атауын енгіземіз!

Debian/Ubuntu
$ sudo nano /etc/chrony/chrony.conf
#pool 2.debian.pool.ntp.org iburst       // артық DNS атауларды "#" Comment-ге алып, төменгі қатарға Қазақстанға ең жақын NTP сервердің DNS атауын енгіземіз!

# Kazakhstan NTP pool
server ntp.nic.kz iburst
pool 2.kz.pool.ntp.org iburst
pool 1.kz.pool.ntp.org iburst

# Global NTP pool
pool time.google.com iburst
pool time.cloudflare.com iburst

# Listen on all interfaces
bindcmdaddress 0.0.0.0
bindcmdaddress ::

# Allow NTP client access from Local Network (жергілікті желіге рұқсат ету)
allow 172.16.11.0/24
allow 172.16.12.0/24

# NTP authentication
keyfile /etc/chrony/chrony.keys

# Log files location
logdir /var/log/chrony
log measurements statistics tracking

# Hardware clock synchronization
rtcsync

# Time adjustment settings (уақыт дәлдігін реттеу)
makestep 1 3
```

NTP authentication
```shell
$ sudo nano /etc/chrony/chrony.keys
# <key_id> <algorithm> <secret_key>
1 MD5 Hello@123

CTRL+O, ENTER, CTRL+X
```
> ЕСКЕРТУ: мұндағы, "MD5" **бас әріппен** жазылуы міндетті!  

**Firewall конфигурациялау**
```shell
nftables конфигурациясы

$ sudo nft add rule inet filter input udp dport 123 ip saddr 172.16.11.0/24 accept
$ sudo nft add rule inet filter input udp dport 123 ip saddr 172.16.12.0/24 accept

$ sudo nft list ruleset | sudo tee /etc/nftables.conf
$ sudo systemctl restart nftables

$ sudo nft list ruleset
```

```shell
iptables конфигурациясы

$ sudo iptables -A INPUT -p udp --dport 123 -s 172.16.11.0/24 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 123 -s 172.16.12.0/24 -j ACCEPT
$ sudo iptables -A INPUT -p udp --dport 123 -s 127.0.0.1 -j ACCEPT

$ sudo netfilter-persistent save
$ sudo netfilter-persistent reload
немесе
$ sudo iptables-save > /etc/iptables/rules.v4
$ sudo systemctl restart iptables

$ sudo iptables -vnL
```

```shell
Firewalld конфигурациясы (RHEL/Rocky)

$ sudo systemctl status firewalld

$ sudo firewall-cmd --permanent --add-port=123/udp
$ sudo firewall-cmd --permanent --add-service=ntp
немесе
$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="172.16.11.0/24" port protocol="udp" port="123" accept'
$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" source address="172.16.12.0/24" port protocol="udp" port="123" accept'

$ sudo firewall-cmd --permanent --add-rich-rule='rule family="ipv4" port protocol="udp" port="123" drop'

$ sudo firewall-cmd --reload

$ sudo firewall-cmd --list-rich-rules
$ sudo firewall-cmd --list-all --zone=public
```

```shell
UFW конфигурациясы (Debian/Ubuntu)

$ sudo ufw status
$ sudo ufw enable

$ sudo ufw allow 123/udp
немесе
$ sudo ufw allow from 172.16.11.0/24 to any port 123 proto udp
$ sudo ufw allow from 172.16.12.0/24 to any port 123 proto udp

$ sudo ufw deny proto udp from any to any port 123

$ sudo ufw reload
$ sudo ufw status verbose
```

Daemon-ды қайта жүктеу
```shell
$ sudo systemctl restart chronyd
немесе
$ sudo systemctl reload chronyd

$ sudo systemctl status chronyd
```

```shell
$ ss -tulpn
Netid  State    Local Address:Port    Peer Address:Port
udp    -        0.0.0.0:123           0.0.0.0:*
```
```shell
$ sudo apt install -y net-tools

$ netstat -tulpn
Proto  Local Address  Foreign Address   State
udp    0.0.0.0:123    0.0.0.0:*         -
```

**Нәтижені тексеру**
```shell
$ sudo chronyc sources -v
$ sudo chronyc tracking
$ sudo chronyc activity
```
```shell
$ sudo apt install ntpdate
$ sudo ntpdate -q 80.241.0.72
```

## NTP Client using Chrony on Linux

```shell
Debian/Ubuntu
$ sudo apt install chrony

RHEL/Rocky/Oracle
$ sudo dnf install chrony

$ sudo systemctl status chronyd
```

```shell
RHEL/Rocky/Oracle
$ sudo vi /etc/chrony.conf
server 172.16.11.1 iburst

Debian/Ubuntu
$ sudo nano /etc/chrony/chrony.conf
server 172.16.11.1 iburst

$ sudo systemctl restart chronyd
```

```shell
Нәтижені тексеру
$ sudo chronyc sources -v
$ sudo chronyc tracking
```

**Қосымша ақпарат**
```shell
Уақытты қолмен синхрондау (тексеру үшін)
$ sudo chronyc makestep
200 OK
```

```shell
Жүйелік уақытты қолмен өзгерту
$ sudo date +%T -s "16:35:55"
$ sudo date -s "2025-08-08 16:35:55"

Уақытты синхрондау (Synchronizing Time)
$ sudo apt install ntpdate

$ sudo ntpdate 172.16.11.1
немесе
$ sudo ntpdate -u 172.16.11.1        //  егер firewall кедергі жасаса қолдану 

$ date
```

```shell
RTC (Real Time Clock) – Hardware Clock, BIOS уақыты
$ sudo hwclock -r

System Clock – жүйелік уақыт
$ date

System Clock пен RTC (BIOS) уақыттың айырмасын тексеру
$ timedatectl
немесе
$ echo "RTC: $(sudo hwclock -r)"; echo "System: $(date)"

System to Hardware clock (аппараттық уақытты жүйелік уақытпен теңестіру)
$ sudo hwclock --systohc

Hardware to System clock (жүйелік уақытты аппараттық уақытпен теңестіру)
$ sudo hwclock --hctosys
```

## NTP Server on Cisco IOS (Router, Switch)

```shell
configure terminal

clock timezone KZ +5                     // Уақыт белдеуін орнату
clock set 08:00:00 7 AUG 2025            // Жергілікті уақытты қолмен орнату

ntp master 3                             // NTP сервер болу, stratum 3
ntp source Loopback50                    // NTP клиенттің сұраныстарына жауап беретін интерфейсті көрсету
```

```shell
Нәтижені тексеру
show ntp status
show ntp associations
```

```shell
access-list 10 permit 172.16.11.0 0.0.0.255     // NTP клиенттерге рұқсат беру
ntp access-group peer 10
```

## NTP Client on Cisco IOS (Router, Switch)

```shell
configure terminal
ntp server 172.16.11.1 prefer        // Жергілікті желідегі (LAN) NTP сервердің IP адресі
clock timezone KZ +5                 // Уақыт белдеуін орнату
```

```shell
ntp source Loopback50                // NTP серверге сұраныс жіберетін интерфейсті көрсету
немесе
ntp source GigabitEthernet1/0/1
```

```shell
Нәтижені тексеру
show ntp associations
show ntp status
```

```shell
access-list 10 permit 172.16.11.1
ntp access-group peer 10
```

## NTP Server on Huawei VRP (Router, Switch)

```shell
system-view

clock timezone KZ add 5
clock datetime 08:00:00 2025-08-07          // Жергілікті уақытты қолмен орнату

ntp-service enable
ntp-service refclock-master 3              // NTP сервер болу, stratum 3
```

```shell
Нәтижені тексеру
display ntp-service status
```

```shell
acl number 2000
 rule permit ip source 172.16.11.0 0.0.0.255
ntp-service acl 2000
```

## NTP Client on Huawei VRP (Router, Switch)

```shell
system-view

clock timezone KZ add 5

ntp-service enable
ntp-service unicast-server 172.16.11.1
```

```shell
Нәтижені тексеру
display ntp-service status
display ntp-service sessions
```

```shell
acl number 2000
 rule permit ip source 172.16.11.1 0.0.0.0
ntp-service acl 2000
```

## NTP Server using ntpd on Oracle Linux 7.9
```shell
$ sudo yum install ntp

$ sudo systemctl status ntpd
```

```shell
$ sudo vi /etc/ntp.conf

server ntp.nic.kz iburst
pool 2.kz.pool.ntp.org iburst

# ішкі желіге рұқсат беру
restrict 172.16.11.0 mask 255.255.255.0 nomodify notrap
restrict 172.16.12.0 mask 255.255.255.0 nomodify notrap

# Барлық басқа сұранысты шектеу
restrict default ignore

$ sudo systemctl restart ntpd
```

```shell
Нәтижені тексеру
$ ntpq -p
$ ntpstat
```

## References

1) [Setting the Date and Time on openEuler](https://docs.openeuler.org/en/docs/20.03_LTS/docs/Administration/basic-configuration.html)
2) [Configuring the Date and Time on RHEL](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/chap-configuring_the_date_and_time)
3) [Configuring NTP using the chrony on RHEL](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_the_chrony_suite)
4) [Configuring NTP using ntpd on RHEL](https://docs.redhat.com/en/documentation/red_hat_enterprise_linux/7/html/system_administrators_guide/ch-configuring_ntp_using_ntpd)
5) [Configuring Clock Synchronization](https://support.huawei.com/enterprise/en/doc/EDOC1100292513/a51c8e62/configuring-clock-synchronization)

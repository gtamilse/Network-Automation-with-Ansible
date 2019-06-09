
# Lab access

## Objective
- Understand lab topology
- SSH into your Ansible controller

## Lab excercise

### Lab topology
![topology](./images/lab-topo.png)

### Lab info
- Get the following info from your proctor
	- VPN server IP address
	- VPN server credentials
	- Your assigned Ansible controller IP address
	- Your assigned R1 IOS router IP address
	- Your assigned R2 XR rotuer IP address
	- Your lab devices credentials
- Important: **Verify** your lab device IP addresses before proceeding

### VPN connect
![anyconnect](./images/anyconnect.png)
- Use Cisco-Anyconnect client to VPN-into the lab
	- type in lab VPN server IP address
	- Click on settings-wheel and uncheck "block connections to untrusted servers"
	- Use VPN credentials provided by the proctor
	- Verify (from your laptop exec prompt):

```
ping <Ansible-Controller-IP-Address>
```

- Proceed if ping succeeds.

### SSH into your Ansible server
- SSH into your Ansible server using any SSH client
        - Putty is provided on CiscoLive lab computers
        - Find your pod's server IP from below link
	- [Lab pod assignment](./TECDEV-1500-Pod-Assignment.md)

### Verification
- Execute the below from your Ansible Controller $ prompt

```
$ ifconfig ens3

$ ping <IP-of-your-IOS-router>

$ ping <IP-of-your-XR-router>
```

- **Make sure:**
	- Ens3 IP address matches with **your assigned controller IP**
	- Ping to both IOS and XR routers succeed

### Example output
- This example output is provided for reference, to be used as needed.
- Do not read this if the steps are executed successfully.

```
GNAGANAB-M-J0A4:~ gnaganab$ ping 172.16.101.93 -c 1
PING 172.16.101.93 (172.16.101.93): 56 data bytes
64 bytes from 172.16.101.93: icmp_seq=0 ttl=63 time=34.493 ms

--- 172.16.101.93 ping statistics ---
1 packets transmitted, 1 packets received, 0.0% packet loss
round-trip min/avg/max/stddev = 34.493/34.493/34.493/0.000 ms
GNAGANAB-M-J0A4:~ gnaganab$
GNAGANAB-M-J0A4:~ gnaganab$ ssh cisco@172.16.101.93
cisco@172.16.101.93's password:
Welcome to Ubuntu 14.04.5 LTS (GNU/Linux 3.13.0-119-generic x86_64)
:
cisco@ansible-controller:~$
cisco@ansible-controller:~$ ifconfig ens3
ens3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 172.16.101.197  netmask 255.255.255.0  broadcast 172.16.101.255
        inet6 fe80::5c00:40ff:fe00:0  prefixlen 64  scopeid 0x20<link>
:
cisco@ansible-controller:~$ ping 172.16.101.91
PING 172.16.101.91 (172.16.101.91) 56(84) bytes of data.
64 bytes from 172.16.101.91: icmp_seq=1 ttl=255 time=0.914 ms
:
--- 172.16.101.91 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 0.893/0.903/0.914/0.031 ms

cisco@ansible-controller:~$ ping 172.16.101.92
PING 172.16.101.92 (172.16.101.92) 56(84) bytes of data.
64 bytes from 172.16.101.92: icmp_seq=1 ttl=255 time=1.17 ms
:
```

- Review the section and discuss if you have any questions.

---

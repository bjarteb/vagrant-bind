
# Install bind on Ubuntu 18.04 (bionic) utilizing Vagrant, VirtualBox and Ansible.

## Introduction

This project started out when I realized I needed a nameserver solution when experimenting
with Prometheus and specifically the blackbox_exporter.

In the book [Prometheus Up&Running](https://learning.oreilly.com/library/view/prometheus-up/9781492034131/) this statement cought my eye:

**Warning.**
```
The name resolution used by Prometheus and the Blackbox exporter is DNS resolution, not the gethostbyname syscall.
```

Well, looks like I need to incorporate a DNS server into my vagrant lab environment.
Two Ansible playbooks are provided to provision nameserver master/slave nodes and a client node.
Have a look in the provisioning folder for details. 
Now I am ready to continue my Prometheus exploration.

## Setup

```bash
# Fire up ns1 (nameserver1), ns2 (nameserver2) and c1 (client1)
vagrant up

# make sure your vm's are running
vagrant status

# login to the boxes
vagrant ssh ns1
vagrant ssh ns2
vagrant ssh c1

vagrant@c1:~$ hostname -f
c1.example.com

# lookup
vagrant@c1:~$ dig ns1.example.com +short
10.1.19.11
# reverse lookup
vagrant@c1:~$ dig -x 10.1.19.11 +short
ns1.example.com.

vagrant@c1:~$ ping ns1.example.com
PING ns1.example.com (10.1.19.11) 56(84) bytes of data.
64 bytes from ns1.example.com (10.1.19.11): icmp_seq=1 ttl=64 time=0.373 ms
64 bytes from ns1.example.com (10.1.19.11): icmp_seq=2 ttl=64 time=0.373 ms

vagrant@c1:~$ ping ns2.example.com
PING ns2.example.com (10.1.19.12) 56(84) bytes of data.
64 bytes from ns2.example.com (10.1.19.12): icmp_seq=1 ttl=64 time=1.00 ms
64 bytes from ns2.example.com (10.1.19.12): icmp_seq=2 ttl=64 time=0.409 ms
```

## Linux Networking


### nameserver nodes

In order to manage your own nameserver on your node you will have to turn off *systemd-resolved*.
*systemd-resolved* is a caching and security mechanism for DNS queries and it controls */etc/resolv.conf*

```bash
sudo systemctl stop systemd-resolved
sudo systemctl disable systemd-resolved
```

Now you can control */etc/resolv.conf*


### client node

Vagrant box configures a /etc/netplan/50-vagrant.yaml to be applied by the netplan utility.


### troubleshooting

*netstat* is deprecated. Use *ss* instead for socket inspection.

```bash
vagrant@ns1:~$ sudo ss -tulpn | grep :53
udp   UNCONN  0       0                 10.1.19.11:53             0.0.0.0:*      users:(("named",pid=2926,fd=514))
udp   UNCONN  0       0                  10.0.2.15:53             0.0.0.0:*      users:(("named",pid=2926,fd=513))
udp   UNCONN  0       0                  127.0.0.1:53             0.0.0.0:*      users:(("named",pid=2926,fd=512))
tcp   LISTEN  0       10                10.1.19.11:53             0.0.0.0:*      users:(("named",pid=2926,fd=23))
tcp   LISTEN  0       10                 10.0.2.15:53             0.0.0.0:*      users:(("named",pid=2926,fd=22))
tcp   LISTEN  0       10                 127.0.0.1:53             0.0.0.0:*      users:(("named",pid=2926,fd=21))
```

*nslookup* is deprecated. Use *dig* instead for name resolution.
```bash
# be aware of +search
# dig ns2 without +search does not resolve.
vagrant@ns1:~$ dig ns2 +short
vagrant@ns1:~$
vagrant@ns1:~$ dig ns2 +short +search
10.1.19.12
```

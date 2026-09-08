# Cisco Packet Tracer Routing Lab

This lab demonstrates **four different routing methods** across the same simple two-router topology. The goal is to understand how static and dynamic routing protocols (RIPv2, EIGRP, and OSPF) establish connectivity between remote LANs.

> **Prerequisite:** Complete the base interface configuration first. Then enable **only one** routing method at a time, verify connectivity, remove it, and move on to the next.

---

## Topology

```
    ┌──────────┐                   ┌──────────┐
    │   PC1    │                   │   PC2    │
    │192.168.10│                   │192.168.30│
    │   .10/24 │                   │   .10/24 │
    └────┬─────┘                   └────┬─────┘
         │                              │
    ┌────┴─────┐                   ┌────┴─────┐
    │  Switch  │                   │  Switch  │
    └────┬─────┘                   └────┬─────┘
         │ G0/1                         │ G0/1
    ┌────┴─────┐      WAN Link         ┌┴─────────┐
    │   NY     │    192.168.20.0/24    │    LA    │
    │   Router │◄─────────────►│   Router │
    │  G0/0: .1│                   │  G0/0: .2│
    └──────────┘                   └──────────┘
```

---

## IP Addressing Table

| Device | Interface | IP Address      | Subnet Mask     | Network           |
|--------|-----------|-----------------|-----------------|-------------------|
| NY     | G0/0      | 192.168.20.1    | 255.255.255.0   | 192.168.20.0/24   |
| NY     | G0/1      | 192.168.10.1    | 255.255.255.0   | 192.168.10.0/24   |
| PC1    | NIC       | 192.168.10.10   | 255.255.255.0   | —                 |
| LA     | G0/0      | 192.168.20.2    | 255.255.255.0   | 192.168.20.0/24   |
| LA     | G0/1      | 192.168.30.1    | 255.255.255.0   | 192.168.30.0/24   |
| PC2    | NIC       | 192.168.30.10   | 255.255.255.0   | —                 |

---

## Base Interface Configuration

Run these commands on each router **before** adding any routing protocol.

### NY Router

```
enable
configure terminal
hostname NY

interface GigabitEthernet0/0
 ip address 192.168.20.1 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 192.168.10.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

### LA Router

```
enable
configure terminal
hostname LA

interface GigabitEthernet0/0
 ip address 192.168.20.2 255.255.255.0
 no shutdown
 exit

interface GigabitEthernet0/1
 ip address 192.168.30.1 255.255.255.0
 no shutdown
 exit

end
write memory
```

### PC Configuration (Packet Tracer)

| PC | IP Address    | Subnet Mask     | Default Gateway |
|----|---------------|-----------------|-----------------|
| PC1 | 192.168.10.10 | 255.255.255.0 | 192.168.10.1   |
| PC2 | 192.168.30.10 | 255.255.255.0 | 192.168.30.1   |

---

## 1. Static Routing

Manually define the remote LAN network on each router.

### NY Router

```
enable
configure terminal

ip route 192.168.30.0 255.255.255.0 192.168.20.2

end
write memory
```

### LA Router

```
enable
configure terminal

ip route 192.168.10.0 255.255.255.0 192.168.20.1

end
write memory
```

### Verify

```
show ip route static
show ip route
ping 192.168.30.10 source 192.168.10.1    (from NY)
ping 192.168.10.10 source 192.168.30.1    (from LA)
traceroute 192.168.30.10                  (from NY)
```

---

## 2. RIPv2

Dynamic distance-vector protocol. Automatically advertises connected networks.

### NY Router

```
enable
configure terminal

router rip
 version 2
 no auto-summary
 network 192.168.20.0
 network 192.168.10.0
 exit

end
write memory
```

### LA Router

```
enable
configure terminal

router rip
 version 2
 no auto-summary
 network 192.168.20.0
 network 192.168.30.0
 exit

end
write memory
```

### Verify

```
show ip protocols
show ip route rip
show ip rip database
ping 192.168.30.10 source 192.168.10.1    (from NY)
ping 192.168.10.10 source 192.168.30.1    (from LA)
```

---

## 3. EIGRP (AS 10)

Advanced distance-vector protocol with fast convergence.

### NY Router

```
enable
configure terminal

router eigrp 10
 no auto-summary
 network 192.168.20.0 0.0.0.255
 network 192.168.10.0 0.0.0.255
 exit

end
write memory
```

### LA Router

```
enable
configure terminal

router eigrp 10
 no auto-summary
 network 192.168.20.0 0.0.0.255
 network 192.168.30.0 0.0.0.255
 exit

end
write memory
```

### Verify

```
show ip protocols
show ip route eigrp
show ip eigrp neighbors
show ip eigrp topology
ping 192.168.30.10 source 192.168.10.1    (from NY)
ping 192.168.10.10 source 192.168.30.1    (from LA)
```

---

## 4. OSPF (Area 0)

Link-state protocol using a single backbone area.

### NY Router

```
enable
configure terminal

router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.10.0 0.0.0.255 area 0
 exit

end
write memory
```

### LA Router

```
enable
configure terminal

router ospf 1
 network 192.168.20.0 0.0.0.255 area 0
 network 192.168.30.0 0.0.0.255 area 0
 exit

end
write memory
```

### Verify

```
show ip protocols
show ip route ospf
show ip ospf neighbor
show ip ospf database
ping 192.168.30.10 source 192.168.10.1    (from NY)
ping 192.168.10.10 source 192.168.30.1    (from LA)
traceroute 192.168.30.10                  (from NY)
```

---

## General Verification Commands

Use these commands on any router to inspect the current state:

```
show running-config
show ip interface brief
show ip route
show ip route summary
show ip protocols
ping <destination>
traceroute <destination>
```

---

## ⚠️ Important Warning

> **Only one routing method should be active at a time.** Running multiple protocols simultaneously on the same topology can cause route flapping, unexpected Administrative Distance conflicts, and unpredictable behavior in Packet Tracer.

### How to Remove Each Protocol

Before switching methods, enter the removal command for the currently active protocol:

| Protocol | Removal Command |
|----------|-----------------|
| Static Routes | `no ip route <network> <mask> <next-hop>` (remove each static line) |
| RIPv2 | `no router rip` |
| EIGRP | `no router eigrp 10` |
| OSPF | `no router ospf 1` |

After removing a protocol, always verify the routing table is clean:

```
show ip route
```

Then apply the next method and test end-to-end connectivity from PC1 to PC2.

---

## Tools

- **Cisco Packet Tracer** — Network simulation and visualization

---

## Author

**Ahcene Sahki**  
ECE Paris — Cybersecurity track

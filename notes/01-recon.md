# 01 — Host Discovery

## Objective
Identify the live IP address of the Kioptrix VM within the lab subnet
without prior knowledge of its address.

## Methodology
Target sits on the same local segment as the attacker (via `virbr0`), so
ARP-based discovery is preferred over ICMP — ARP cannot be filtered out on
a local segment the way ICMP can be, since it's required for the network
to function at all.

## Command
```bash
sudo nmap -sn 192.168.122.0/24
```
- `-sn`: host discovery only, no port scan
- `sudo`: enables raw ARP requests instead of falling back to slower
  TCP-based discovery

## Results
```
Nmap scan report for 192.168.122.171
Host is up (0.0015s latency).
MAC Address: 52:54:00:30:F7:1F (QEMU virtual NIC)
Nmap scan report for 192.168.122.1
Host is up.
Nmap done: 256 IP addresses (2 hosts up) scanned in 7.05 seconds
```

## Interpretation
- `192.168.122.1` — no MAC shown → attacker's own gateway/host
- `192.168.122.171` — MAC vendor `QEMU virtual NIC` → the only virtualized
  guest on the subnet → identified as the Kioptrix target

**Target confirmed: `192.168.122.171`**

## Notes
On a subnet with multiple QEMU/KVM guests, MAC vendor alone wouldn't be
enough to positively ID the target — would need to correlate against
`virsh net-dhcp-leases` or boot the target VM in isolation and diff the
host list.

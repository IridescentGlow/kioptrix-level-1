# Kioptrix Level 1 — Penetration Testing Notes

## Objective
Practice offensive security methodology end-to-end (recon → enumeration →
vulnerability research → exploitation → privilege escalation) against the
Kioptrix Level 1 VulnHub target, treating it as a black-box engagement.

## Scope
- Target: Kioptrix Level 1 VM, isolated lab network (libvirt NAT, `virbr0`,
  `192.168.122.0/24`)
- Attacker: Arch Linux host
- Rules of engagement: local lab only, no production/real-world targets

## Environment
| Role     | Detail                          |
|----------|----------------------------------|
| Attacker | Arch Linux, Hyprland, virt-manager/libvirt |
| Target   | Kioptrix Level 1 (Red Hat–based, old)      |
| Network  | NAT, `192.168.122.0/24`                    |

## Progress Log
- [x] Host discovery
- [x] Full TCP port scan
- [x] Service/version enumeration
- [x] Apache/mod_ssl vulnerability research (CVE-2002-0082)
- [x] Samba enumeration + version confirmation (Samba 2.2.1a)
- [ ] Samba vulnerability research (CVE-2003-0201 / trans2open)
- [ ] Exploitation
- [ ] Privilege escalation
- [ ] Final report / lessons learned

## Notes Index
- [01 - Host Discovery](notes/01-recon.md)
- [02 - Port Scanning](notes/02-port-scan.md)
- [03 - Service Enumeration](notes/03-service-enum.md)
- [04 - Vulnerability Research](notes/04-vuln-research.md)

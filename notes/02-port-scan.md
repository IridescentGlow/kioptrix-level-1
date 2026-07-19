# 02 — Port & Service Enumeration (Discovery)

## Objective
Enumerate all open TCP ports on the target — full range, not just the
top-1000 default — to avoid missing a high-port service that could be the
actual way in.

## Methodology
SYN scan (`-sS`) chosen over full connect scan (`-sT`):
- Faster (no full 3-way handshake teardown per port)
- Stealthier (many app-level logs only record fully-established connections)
- Requires root (raw packet crafting)

## Command
```bash
sudo nmap -sS -p- 192.168.122.171
```

## Results
```
PORT      STATE SERVICE
22/tcp    open  ssh
80/tcp    open  http
111/tcp   open  rpcbind
139/tcp   open  netbios-ssn
443/tcp   open  https
32768/tcp open  filenet-tms
```

## Interpretation / Prioritization
| Port  | Service        | Priority reasoning |
|-------|----------------|---------------------|
| 139   | netbios-ssn (SMB/Samba) | Historically high-risk service family on old Linux boxes (e.g. Samba RCE history) — prioritize enumeration |
| 80/443 | Apache HTTP/HTTPS | Rich attack surface, worth checking regardless of age |
| 111   | rpcbind        | Moderate — portmapper for RPC services, useful for further enumeration |
| 32768 | filenet-tms (Nmap guess) | Service name is Nmap's port-DB guess, not confirmed — needs `-sV` to verify |
| 22    | ssh            | Old version likely, but no immediate high-severity RCE lead at this stage |

Note: service *names* from a plain `-sS` scan are unconfirmed guesses —
`-sV` in the next phase is required to verify what's actually listening.

# 03 — Version Detection & Banner Grabbing

## Objective
Confirm actual service versions on the 6 open ports (replacing Nmap's
earlier port-DB guesses with real probe results), to identify realistic
vulnerability candidates.

## Command
```bash
sudo nmap -sS -p22,80,111,139,443,32768 -sV -sC 192.168.122.171
```
- `-sV`: version detection via active probing
- `-sC`: default NSE script set (banner grabs, common enum scripts)

## Results (key findings)
```
22/tcp    open  ssh         OpenSSH 2.9p2 (protocol 1.99)
80/tcp    open  http        Apache httpd 1.3.20 ((Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b)
111/tcp   open  rpcbind     2 (RPC #100000)
139/tcp   open  netbios-ssn Samba smbd (workgroup: MYGROUP)
443/tcp   open  ssl/https   Apache/1.3.20 (Unix) (Red-Hat/Linux) mod_ssl/2.8.4 OpenSSL/0.9.6b
32768/tcp open  status      1 (RPC #100024)
```
Host script results also confirmed NetBIOS name: `KIOPTRIX`.

## Interpretation
- **SSH**: OpenSSH 2.9p2 / protocol 1.99 — extremely old, SSHv1 supported.
  No strong RCE lead found in later research (see 04).
- **Apache/mod_ssl**: Apache 1.3.20, mod_ssl/2.8.4, OpenSSL/0.9.6b — all
  early-2000s versions. Flagged as top-priority research candidate.
- **Samba**: confirmed present but **no version number** returned by
  `-sV`/`-sC` — requires deeper manual enumeration (see 04).
- **RPC (111/32768)**: standard infrastructure services, low priority as
  standalone attack surface in this case.

## Next Steps
Research CVEs for confirmed versions (Apache/mod_ssl first, per priority
above); separately pursue Samba version fingerprinting via manual protocol
interaction since automated scanning didn't reveal it.

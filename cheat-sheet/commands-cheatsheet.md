# 📋 Volatility 3 — Commands Cheatsheet

> Quick reference for the most commonly used Volatility 3 plugins during a Windows memory investigation.  
> All commands assume your memory dump is named `memory.dmp` — replace it with your actual filename.

---

## 🧱 Basic Syntax

```bash
python vol.py -f "memory.dmp" [plugin]
```

To save output to a file instead of printing to screen:

```bash
python vol.py -f "memory.dmp" [plugin] > output.txt
```

---

## 🖥️ System Information

| Command | What it does |
| --- | --- |
| `windows.info` | OS version, architecture, build number |
| `windows.envars` | Environment variables for all processes |

```bash
# Always run this first — confirms the dump is Windows and gives you OS details
python vol.py -f "memory.dmp" windows.info
```

---

## ⚙️ Process Analysis

| Command | What it does |
| --- | --- |
| `windows.pslist` | Flat list of all running processes |
| `windows.pstree` | Hierarchical tree showing parent-child relationships |
| `windows.cmdline` | Full command-line arguments for every process |
| `windows.dlllist` | DLLs loaded by each process |

```bash
# See who launched what — PPID is your best friend
python vol.py -f "memory.dmp" windows.pstree

# Get the full command that launched each process
python vol.py -f "memory.dmp" windows.cmdline

# Filter for a specific process by name
python vol.py -f "memory.dmp" windows.pslist | grep -i powershell
```

> ⚠️ **Tip:** Always check the PPID (Parent PID).  
> `Word.exe` or `Excel.exe` launching `PowerShell.exe` = immediate red flag.

---

## 🌐 Network Analysis

| Command | What it does |
| --- | --- |
| `windows.netstat` | Active and recently closed connections |
| `windows.netscan` | Broader scan — catches more connections including closed ones |

```bash
python vol.py -f "memory.dmp" windows.netstat
python vol.py -f "memory.dmp" windows.netscan
```

**What to look for:**

| Suspicious Sign | Example |
| --- | --- |
| Unusual outbound port | Port `4444`, `8888`, `1337` |
| Unknown external IP | Any non-local IP from an unexpected process |
| Unexpected process with network activity | `rundll32.exe` or `notepad.exe` making connections |

---

## 🔑 User & Session Info

| Command | What it does |
| --- | --- |
| `windows.sessions` | Active user sessions and which processes belong to them |
| `windows.hashdump` | Extract NTLM password hashes for all users |
| `windows.cachedump` | Cached domain credentials |
| `windows.lsadump` | LSA secrets — service account passwords |

```bash
# Find which user ran a suspicious process
python vol.py -f "memory.dmp" windows.sessions

# Extract password hashes (can be cracked offline)
python vol.py -f "memory.dmp" windows.hashdump
```

---

## 📁 File & Handle Analysis

| Command | What it does |
| --- | --- |
| `windows.handles --pid X` | Files, registry keys, and objects used by a process |
| `windows.dumpfiles` | Extract a file from memory to disk |
| `windows.filescan` | Scan memory for file objects |

```bash
# See what files/keys a suspicious process is touching
python vol.py -f "memory.dmp" windows.handles --pid 1234

# Filter handles for a specific type
python vol.py -f "memory.dmp" windows.handles --pid 1234 | grep -i "File"

# Extract a specific file from memory (get the address from filescan first)
python vol.py -f "memory.dmp" -o "./output_folder" windows.dumpfiles --virtaddr 0xADDRESS
```

---

## 🗂️ Registry Analysis

| Command | What it does |
| --- | --- |
| `windows.registry.hivelist` | List all registry hives loaded in memory |
| `windows.registry.printkey` | Read a specific registry key |
| `windows.registry.userassist` | Recent programs opened by the user |

```bash
# List all hives
python vol.py -f "memory.dmp" windows.registry.hivelist

# Check persistence locations
python vol.py -f "memory.dmp" windows.registry.printkey --key "Software\Microsoft\Windows\CurrentVersion\Run"

# See what the user recently opened
python vol.py -f "memory.dmp" windows.registry.userassist
```

**Important Hives:**

| Hive | Contains |
| --- | --- |
| `SAM` | User account password hashes |
| `SYSTEM` | System config, device info |
| `SOFTWARE` | Installed programs |
| `NTUSER.DAT` | Per-user settings, recent files, browser history |

---

## 🧬 Malware Hunting

| Command | What it does |
| --- | --- |
| `windows.malfind` | Find processes with injected code or suspicious memory regions |
| `windows.vadinfo --pid X` | Virtual address descriptor — memory layout of a process |
| `windows.memmap --pid X` | Memory map for a specific process |

```bash
# Scan all processes for signs of code injection
python vol.py -f "memory.dmp" windows.malfind

# Investigate memory regions of a specific suspicious process
python vol.py -f "memory.dmp" windows.vadinfo --pid 1234
```

> 💡 `malfind` looks for memory regions that are executable but not backed by a file on disk — a classic sign of process injection or shellcode.

---

## 🔎 Quick Investigation Workflow

When you get a memory dump, run these in order:

```bash
# 1. Confirm OS
python vol.py -f "memory.dmp" windows.info

# 2. Look at process tree
python vol.py -f "memory.dmp" windows.pstree

# 3. Check command lines for anything suspicious
python vol.py -f "memory.dmp" windows.cmdline

# 4. Check network connections
python vol.py -f "memory.dmp" windows.netstat

# 5. Hunt for injection
python vol.py -f "memory.dmp" windows.malfind

# 6. If you found a suspicious PID, dig into it
python vol.py -f "memory.dmp" windows.handles --pid [PID]
python vol.py -f "memory.dmp" windows.vadinfo --pid [PID]
```

---

## 💡 Useful Tips

**Pipe with `more` on Windows (PowerShell):**
```powershell
python .\vol.py -f "memory.dmp" windows.pslist | more
```

**Search output for a keyword (Linux/Mac):**
```bash
python vol.py -f "memory.dmp" windows.pslist | grep -i chrome
```

**Search output for a keyword (Windows PowerShell):**
```powershell
python .\vol.py -f "memory.dmp" windows.pslist | Select-String "chrome"
```

**Always create an output folder before dumping files:**
```bash
mkdir output
python vol.py -f "memory.dmp" -o "./output" windows.dumpfiles --virtaddr 0xADDRESS
```

---

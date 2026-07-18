# 🛠️ Volatility 3 — Installation Guide

> Step-by-step guide to install Volatility 3 on Windows.  
> Estimated time: ~15 minutes

---

## 📋 Requirements

| Requirement | Purpose | Download |
| --- | --- | --- |
| **Python 3** | Volatility 3 runs on Python | [python.org](https://www.python.org/downloads/) |
| **Git** | Clone the Volatility repo from GitHub | [git-scm.com](https://git-scm.com/downloads) |
| **Visual Studio Build Tools** | Required to compile Python modules | [visualstudio.microsoft.com](https://visualstudio.microsoft.com/visual-cpp-build-tools/) |

> ⚠️ During Visual Studio installation, make sure to select **"Desktop development with C++"**

---

## 📥 Step 1 — Clone Volatility 3

Open PowerShell or Command Prompt and run:

```bash
git clone https://github.com/volatilityfoundation/volatility3.git
cd volatility3
```

---

## 📦 Step 2 — Install Dependencies

```bash
pip install -r requirements.txt
```

For handling large compressed memory dumps, also install:

```bash
pip install python-snappy
```

> 💡 `python-snappy` helps Volatility handle large `.dmp` files faster during analysis.

---

## ✅ Step 3 — Verify Installation

Run this command inside the `volatility3` folder:

```powershell
python .\vol.py
```

**Expected output:**

```
Volatility 3 Framework 2.x.x
usage: vol.py [-h] [-c CONFIG] [--parallelism ...] [-e EXTEND] [-p PLUGIN_DIRS]
              [-s SYMBOL_DIRS] [-v] [-l LOG] [-o OUTPUT_DIR] [-q]
              [-r RENDERER] [-f FILE] [--write-config] [--save-config ...]
              [--clear-cache] [--cache-path CACHE_PATH] [--offline]
              [--single-location SINGLE_LOCATION]
              [--stackers STACKERS [STACKERS ...]]
              [--single-swap-locations SINGLE_SWAP_LOCATIONS [SINGLE_SWAP_LOCATIONS ...]]
              plugin ...
vol.py: error: Please select a plugin to run
```

> ✅ If you see this error — Volatility is working correctly.  
> It just wants you to specify a plugin. That's expected behavior.

---

## 🔄 Step 4 — Keeping Volatility Updated

To get the latest plugins and bug fixes, run this inside the `volatility3` folder:

```bash
git pull
```

---

## 🧪 Step 5 — Run Your First Command

Put a memory dump file (`.dmp` or `.mem`) inside the `volatility3` folder, then run:

```bash
python vol.py -f "your_file.dmp" windows.info
```

If you see Windows version info — you're ready to go. 🎉

---

## 📁 Supported Memory Dump Formats

| Extension | Notes |
| --- | --- |
| `.dmp` | Standard Windows memory dump |
| `.mem` | Raw memory image |
| `.raw` | Raw memory image |
| `.vmem` | VMware memory snapshot |

> Both `.dmp` and `.mem` are identical in content — the difference is only in the name.

---

## ❌ Common Errors & Fixes

| Error | Cause | Fix |
| --- | --- | --- |
| `python is not recognized` | Python not in PATH | Reinstall Python and check **"Add to PATH"** during setup |
| `No module named 'snappy'` | python-snappy not installed | Run `pip install python-snappy` |
| `error: Microsoft Visual C++ required` | Build tools missing | Install Visual Studio Build Tools and select "Desktop development with C++" |
| `Please select a plugin to run` | No plugin specified | ✅ This is normal — add a plugin name to your command |

---

## 📚 Reference

- Official Docs: [https://volatility3.readthedocs.io](https://volatility3.readthedocs.io)
- GitHub Repo: [https://github.com/volatilityfoundation/volatility3](https://github.com/volatilityfoundation/volatility3)

# c0c0n 2026 CTF — Session Handoff & Context

> **Paste-ready brief for a fresh Claude session.** Read this top to bottom before doing anything. It tells you WHO the user is, WHAT we're doing, WHY, and HOW — plus the exact state of the machine so you can be useful in minutes, not after re-discovery.

---

## 1. WHO & WHAT (the mission)

- **User:** Syed (email uaetva@gmail.com). Attending **c0c0n 2026** cybersecurity conference in **Kochi, India (Grand Hyatt), Oct 9–10 2026**, and competing in its **CTF** ("City under Attack" theme — hospitals, power plants, govt offices).
- **You (Claude, Opus 4.8) are an ALLOWED AI agent in this CTF.** The official rules explicitly say "feel free to bring your AI agents" for **Day 1**. Day 2 (top-5 teams, isolated range) AI policy is **NOT stated — Syed must ask organizers on-site before Day 2.**
- **Your job:** act as Syed's force-multiplier teammate — triage the challenge board fast, solve/co-solve challenges using the local tooling, and fan out parallel subagents on independent challenges. Syed verifies and submits every flag (the rules make the human responsible for AI output).
- **Format:** Day 1 = open, Jeopardy-style flags, Beginner→Advanced, fixed points per challenge. Day 2 = closed, top-5 teams, multi-step scenarios in isolated range. Ties broken by **time-to-capture** (so speed on easy/mid flags matters for ranking). Team ≤3, onsite only, bring photo ID + laptop + own internet.
- **Flag format:** `c0c0n-ctf{...}`.

## 2. WHY it's set up this way (decisions already made — don't relitigate)

- The **harness (arbitrary local code execution) is already maximal** for CTF. We did NOT add web/RE MCP connectors — they'd add nothing over real local tool execution. Two browsers (built-in pane + Chrome) + curl + scripted HTTP already cover web. Business/Shopify/Gmail connectors in the session are IRRELEVANT to CTF — ignore them.
- Machine is **Windows 11 + MSYS/Git-Bash**, but most CTF targets are **Linux ELF** and pwn tooling is Linux-native. **WSL is broken** ("Class not registered"). So we use **Docker** as the Linux workhorse.
- RE approach: **radare2 (instant) + Ghidra GUI** are the default. Ghidra *headless Python scripting* has a slow first-run bootstrap (timed out at 300s) — **don't rely on scripted Ghidra headless under time pressure.**

## 3. HOW — machine state (ALL SET UP & VERIFIED 2026-10-08)

### Environments
| Env | How to invoke | Notes |
|---|---|---|
| Native Windows Python 3.12 | `python` (NOT `python3`) | Has: pwntools, pycryptodome, requests, z3, gmpy2, sympy, Pillow, capstone, ropper, volatility3 |
| **ctfbox Docker container** | `C:\c0c0n-ctf\ctfbox.ps1` (interactive) or `.\ctfbox.ps1 <cmd>` | **The Linux workhorse.** Mounts cwd at `/work`. Add `-e TERM=xterm` (already in wrapper). |
| Native tools | nmap, Wireshark/tshark, JDK 21 | installed via winget |
| **Ghidra 12.1.4** | `C:\c0c0n-ctf\tools\ghidra_12.1.4_PUBLIC\ghidraRun.bat` (GUI) | Headless = `support\analyzeHeadless.bat` (needs `-pyghidra` for scripts; slow first run) |
| Browsers | built-in pane (`mcp__Claude_Browser__*`) + Chrome | web recon, auth'd sessions. Medium blocks WebFetch (403) → use browser pane. |

### Driving ctfbox from the Bash tool (non-interactive):
```
MSYS_NO_PATHCONV=1 docker run --rm -v "/c/c0c0n-ctf:/work" -w /work ctfbox:latest bash -c '<commands>'
```
**ctfbox v2 — COMPREHENSIVE toolkit (~14GB, all verified).** Highlights by category:
- **pwn/RE:** gdb, gdb-multiarch, pwndbg, radare2, nasm, yasm, ROPgadget, ropper, one_gadget, seccomp-tools, patchelf, qemu-user-static (arm/mips/x86 cross-arch), gcc-multilib, cross binutils; Python: pwntools, capstone, unicorn, keystone, angr, r2pipe.
- **web:** gobuster, ffuf, httpx, subfinder, waybackurls, assetfinder, nikto, whatweb, wafw00f, wfuzz, mitmproxy, flask, bs4/lxml. Wordlists: **SecLists** (/usr/share/wordlists/SecLists), **rockyou.txt** (14.3M lines).
- **forensics/net:** tshark, tcpdump, tcpflow, nmap, masscan, arp-scan, hping3, sleuthkit (mmls/fls/icat), testdisk, scalpel, foremost, binwalk, ewf-tools, volatility3, scapy, pyshark, dpkt, impacket, exiftool, exiv2, imagemagick, ffmpeg, pngcheck, zbarimg.
- **crypto/stego/pw:** openssl, steghide, outguess, zsteg, john, hashcat, hashid, hydra, fcrackzip, pdfcrack, ophcrack; Python: pycryptodome, sympy, gmpy2, z3, cryptography, pyjwt, base58. uncompyle6, pyinstxtractor.py.
- **mobile:** apktool, jadx 1.5.6, dex2jar (d2j-* suite), frida + frida-tools, objection.
- **misc:** node/npm/js-beautify, rg/fd/bat/jq, pdftotext, 7z, unrar, tmux.
- Dockerfile source of truth: `C:\c0c0n-ctf\tools\ctfbox.Dockerfile` (rebuild: `docker build -t ctfbox:latest -f ctfbox.Dockerfile .`).
- **PREREQ: Docker Desktop must be RUNNING before the event** (it's the Linux box). Start "Docker Desktop.exe", wait ~1 min for daemon.

### Remaining tooling notes:
- apt `dirb` wordlist symlink is dangling — just use SecLists (`/opt/SecLists/Discovery/Web-Content/`), it's better.
- SageMath deliberately omitted (sympy+gmpy2+pycryptodome usually suffice; add only if a challenge truly needs Sage).
- Native Windows has no `openssl` — route cert/crypto CLI through the container.

## 4. HOW — multi-agent orchestration (VERIFIED working)

- You (main session) are the **hub**. You can spawn **parallel Opus subagents** (via the Agent tool), each a fresh instance with its own context + tools. They are **spokes** — they report to YOU, they don't chat peer-to-peer. You consolidate.
- **Game plan when the board drops:**
  1. Fast INLINE triage of the whole board → categorize, flag easy wins, priority map.
  2. Fan out 3–5 solver agents on INDEPENDENT, OFFLINE challenges (crypto/forensics/RE/OSINT/stego). Give each a self-contained brief + file paths + "use the ctfbox container for tools."
  3. Keep LIVE web + pwn on the main thread with Syed (need the human-in-the-loop shot at the live service).
  4. Agents report → you verify `c0c0n-ctf{...}` format → Syed submits.
- **Constraints:** all agents share Syed's ONE laptop (don't run multiple heavy jobs — volatility+hashcat+angr — at once; stagger). Agents can't reach the range directly, only through the laptop's network (VPN/venue net must be connected first). Spawning has cold-start overhead — solve quick ones inline. 5 parallel Opus ≈ 5× token burn (fine for the event).

## 5. What we know about past challenges (prep, NOT leaked answers)

There are TWO streams: **DomeCTF/BESEC.CTF** (onsite flagship — Syed's lineage) and the separate **online "c0c0n CTF"** (Sept 2025, "Adversary City"). **No leaked answers exist for the 2026 or 2025-onsite events** — challenges change yearly. Prep = pattern-matching + tooling speed.

Chains are **multi-step "follow-the-breadcrumb"** built from standard primitives (difficulty = noticing breadcrumbs + chaining fast across a big board, + live attack-defense pivots on Day 2). The hard tail genuinely resists even pro teams (UST swept 2024 over 2 days).

**Pattern-match the board against the technique catalog in memory** (file `cocon-technique-catalog.md`). Highlights to try early:
- **Web:** check robots.txt first; try admin:admin; gobuster dirs then re-enum inside /api with -x json,txt; look for /api/flag.txt; path traversal to .env; exposed .git / commit history; custom HTTP verb from RFC hint; try old api version (v1 vs v2).
- **OSINT:** follow social links → page-source comments; GitHub old commits with hardcoded creds → `c0c0n-ctf{user:pass}`.
- **Forensics:** log + pcap → filter attacker IP → follow HTTP stream; custom User-Agent as fingerprint; DNS TXT exfil (`dns and ip.addr==X`).
- **Crypto:** chained encodings (reverse the chain: rotate/base64/shift/reverse); MD5-with-junk-char trick; classic ciphers.
- **Mobile:** apktool + Frida bypass of client-side auth (needs tooling install).

## 6. Full detail lives in MEMORY

The session memory (`C:\Users\Syed\.claude\projects\C--c0c0n-ctf\memory\`) has the complete record — read `MEMORY.md` index, then:
- `cocon-ctf-2026.md` — event facts & rules
- `machine-ctf-tooling.md` — exact installed/missing tools
- `ctf-setup-decisions.md` — why we set up the way we did
- `cocon-ctf-history-research.md` — editions 2019–2025 deep research
- `cocon-technique-catalog.md` — the full reusable pattern library
- `cocon-2025-chain-patterns.md` — 2025 online chains in detail
- `day1-orchestration-plan.md` — multi-agent plan & constraints

## 7. Also in the project folder
- `CTF-PLAYBOOK.md` — operations one-pager (which env per category, intake flow, rules reminders)
- `ctfbox.ps1` — one-command Linux shell
- `tools/ghidra_12.1.4_PUBLIC/` — Ghidra

---

## IMMEDIATE NEXT STEPS for a fresh session
1. Read `MEMORY.md` + this file. Confirm Docker Desktop is running (`docker info`).
2. If it's event day: confirm Syed's laptop can reach the CTF range (VPN/venue net). Ask about Day-2 AI policy if relevant.
3. When Syed hands you a challenge/board: triage fast, pattern-match against the technique catalog, fan out agents on independent offline challenges, keep live targets with Syed.
4. **Verify every flag's format before Syed submits. Respect the rules: in-scope only, no DoS, no attacking other teams/scoreboard/organizers, don't exploit platform bugs (report privately), share writeups only after the event.**

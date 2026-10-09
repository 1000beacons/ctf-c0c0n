# c0c0n 2026 CTF — Operations Playbook

Our setup and how we work a challenge together, fast. Event: Oct 9–10 2026, Grand Hyatt Kochi.

## Environments available

| Environment | How to use | Best for |
|---|---|---|
| **Native Windows Python** (`python`) | I just run it | Quick crypto/scripting, HTTP replay |
| **`ctfbox` Linux container** | `.\ctfbox.ps1` or I drive via docker | Pwn, Linux ELF, forensics, everything |
| **Native tools** | nmap, Wireshark/tshark | Network scans, pcap GUI if you want |
| **Ghidra GUI** | `tools\ghidra_12.1.4_PUBLIC\ghidraRun.bat` | Eyeballing complex decompiled functions |
| **radare2** (in container) | I drive it | Fast disassembly/decompile — my RE default |
| **Two browsers** | built-in pane + your Chrome | Web recon, auth'd sessions |

### Running the Linux box
- Interactive shell (project mounted at `/work`): `.\ctfbox.ps1`
- One-off: `.\ctfbox.ps1 nmap -sV target`
- Anything you drop in `C:\c0c0n-ctf\` is instantly visible at `/work` inside the box.

## Workflow when a challenge drops

1. **Hand it to me raw.** Paste the prompt + any URL, and drop files into `C:\c0c0n-ctf\<challenge-name>\`. Tell me the category if you know it.
2. I triage: identify type, run first-pass tooling, report what I find.
3. We decide: I solve it end-to-end, or I set it up and you take the live/interactive part.
4. **You verify the flag before submitting** (the rules make the human responsible for AI output).

## Category → how I attack it

- **Crypto** → native Python (pycryptodome, sympy, gmpy2, z3). RSA/ECB/oracle patterns.
- **Web** → scripted HTTP (requests/httpx) + browser for recon. SQLi/SSTI/SSRF/IDOR/auth.
- **Reverse** → radare2 first (instant), Ghidra GUI for hard functions, gdb/pwndbg to run.
- **Pwn** → container: pwntools + pwndbg + one_gadget + ropper + patchelf. You run against live service.
- **Forensics** → container: binwalk, foremost, volatility3, tshark, exiftool, sqlite3.
- **Stego** → container: zsteg, steghide, exiftool, binwalk + my image analysis.
- **OSINT/misc/programming** → me directly + web search.

## Rules reminders (don't get DQ'd)
- In scope only. **No DoS**, no attacking other teams / venue net / scoreboard / organizers.
- Don't exploit platform bugs — report them privately.
- Share writeups/tools only AFTER the competition.
- **Day 2 AI policy is NOT published — ask organizers on-site before Day 2.**
- Team ≤3, onsite only, bring photo ID + laptop + own internet.

## Keep this machine ready
- Docker Desktop must be running before the event (daemon = the Linux box).
- If `ctfbox` image is ever lost: rebuild from the Dockerfile (I have the path in memory).

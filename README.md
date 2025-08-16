# Theory
Terraform/Ansible along with other providers need to use 1Password CLI with biometric authentication in order to communicate with 1Password correctly. Unfortunately since your interactive environment is Windows, it can't do this. While you could spin up a virtual box of Linux, that would be tricky and force you to authenticate to 1Password multiple times. In this article, I will guide you through how to pass all the 1Password CLI requests from WSL into Windows, will also transferring any necessary environment variables so that everything correctly functions, as if you were using 1Password from a full native Linux environment.

# Quick Setup
Install 1Password CLI on Windows
```powershell
powershell.exe -NoProfile -Command "winget install --id AgileBits.1Password.CLI --silent --accept-package-agreements --accept-source-agreements"
```
Remove 1Password CLI in WSL & Install Wrapper Script
```bash
sudo apt remove 1password-cli -y && sudo curl https://raw.githubusercontent.com/brendanpolin/1Password-CLI-WSL-Integration/refs/heads/main/op -o /usr/local/bin/op && sudo chmod +x /usr/local/bin/op
```

# Prerequisites
Ensure 1password-cli is removed in WSL
```bash
sudo apt remove 1password-cli -y
```
Ensure 1Password CLI is Installed on Windows
```powershell
powershell.exe -NoProfile -Command "winget install --id AgileBits.1Password.CLI --silent --accept-package-agreements --accept-source-agreements"
```

# Windows Authentication Wrapper Script
The wrapper script points all calls from wsl to 1Password CLI to execute your 1Password CLI installation in Windows, and pass all the necessary 1Password environment variables ($OP_) from wsl into Windows for interpretation by op.exe

Create the file `/usr/local/bin/op` and paste the contents of the below script in there
```bash
#!/bin/bash

# Find base folder for 1Password CLI in current user's Windows WinGet Packages
WIN_OP_BASE="/mnt/c/Users/$(cmd.exe /c echo %USERNAME% | tr -d '\r')/AppData/Local/Microsoft/WinGet/Packages"

# Find the latest folder matching the pattern (AgileBits.1Password.CLI*)
OP_DIR=$(ls -td "$WIN_OP_BASE"/AgileBits.1Password.CLI* 2>/dev/null | head -n1)

if [ -z "$OP_DIR" ]; then
    echo "[ERROR] Could not find 1Password CLI folder in $WIN_OP_BASE" >&2
    exit 1
fi

mapfile -d '' op_env_vars < <(env -0 | grep -z ^OP_ | cut -z -d= -f1)
export WSLENV="${WSLENV:-}:$(IFS=:; echo "${op_env_vars[*]}")"
exec $OP_DIR/op.exe "$@"
```

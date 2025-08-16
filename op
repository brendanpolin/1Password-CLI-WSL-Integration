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

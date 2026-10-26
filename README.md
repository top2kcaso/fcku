from pathlib import Path

script = r'''#!/bin/bash
# DEVICE + FIRMWARE WORKBENCH
# macOS Big Sur compatible baseline; inspect before changing anything.
# No credentials are requested or stored.

set -u

BLUE='\033[0;34m'
GREEN='\033[0;32m'
YELLOW='\033[0;33m'
RESET='\033[0m'
OUT="$HOME/Desktop/device_workbench_reports"
mkdir -p "$OUT"

pause() { printf "\nPress Return to continue..."; read -r _; }
heading() { printf "\n%s\n" "=================================================="; printf "%s\n" "$1"; printf "%s\n" "=================================================="; }

connection_check() {
  heading "USB CONNECTION CHECK"
  echo "Checking the Mac's USB tree and installed device utilities..."
  echo
  sw_vers
  echo
  echo "--- USB devices (may take a moment) ---"
  system_profiler SPUSBDataType
  echo
  echo "--- Apple/iPhone/recovery USB entries ---"
  ioreg -p IOUSB -l -w 0 | grep -i -B 5 -A 12 -E 'iPhone|Apple Mobile|Recovery|DFU' || echo "No matching labels in filtered output."
  echo
  echo "--- Optional libimobiledevice utility ---"
  if command -v idevice_id >/dev/null 2>&1; then
    idevice_id -l
  else
    echo "idevice_id not installed."
  fi
  echo
  echo "--- devicectl availability (may not exist on Big Sur) ---"
  xcrun --find devicectl 2>&1 || true
  echo
  echo "Interpretation: a USB entry is evidence of visibility only, not proof of pairing or activation."
}

firmware_info() {
  heading "FIRMWARE / IPSW WORKBENCH"
  echo "This tool does not guess a firmware file or flash automatically."
  echo "For iPhone 12 Pro Max, verify the exact device model and firmware compatibility before restoring."
  echo
  echo "--- Device model, if trusted pairing utilities are available ---"
  if command -v ideviceinfo >/dev/null 2>&1; then
    ideviceinfo -k ProductType 2>&1 || true
    ideviceinfo -k ProductVersion 2>&1 || true
    ideviceinfo -k BuildVersion 2>&1 || true
  else
    echo "ideviceinfo unavailable or not installed; USB visibility alone cannot provide iOS version."
  fi
  echo
  echo "Choose a local IPSW file to inspect its filename:"
  printf "Enter full path to IPSW (or press Return to skip): "
  read -r ipsw
  if [ -n "$ipsw" ]; then
    if [ -f "$ipsw" ]; then
      echo "File found:"
      ls -lh "$ipsw"
      file "$ipsw"
      echo "Filename only does not prove that Apple will sign or accept this firmware."
    else
      echo "No file found at that path."
    fi
  fi
  echo
  echo "Firmware restore through Finder can erase the device."
  echo "A restore does not change the Apple Account associated with Activation Lock."
}

make_report() {
  local report="$OUT/device_report_$(date +%Y%m%d_%H%M%S).txt"
  {
    echo "DEVICE WORKBENCH REPORT"
    date
    echo
    sw_vers
    echo
    echo "--- Developer tools ---"
    xcode-select -p 2>&1
    xcodebuild -version 2>&1 || true
    echo
    echo "--- USB report ---"
    system_profiler SPUSBDataType
    echo
    echo "--- filtered USB entries ---"
    ioreg -p IOUSB -l -w 0 | grep -i -B 5 -A 12 -E 'iPhone|Apple Mobile|Recovery|DFU' || true
    echo
    echo "--- optional tools ---"
    command -v idevice_id || true
    command -v ideviceinfo || true
    xcrun --find devicectl 2>&1 || true
  } > "$report"
  echo "Saved report: $report"
  echo "Review and redact serial numbers/device identifiers before sharing."
}

recovery_steps() {
  heading "RECOVERY MODE WORKFLOW"
  cat <<'EOF'
iPhone 12 Pro Max:
1. Disconnect the cable.
2. Quickly press and release Volume Up.
3. Quickly press and release Volume Down.
4. Hold the Side button until the recovery screen appears.
5. Connect the phone to the Mac and open Finder.
6. Select the device if Finder detects it.
7. Choose Update if appropriate; Restore erases the device.

A restore reinstalls iOS but does not itself remove Activation Lock.
EOF
}

account_steps() {
  heading "ACCOUNT / DEVICE OWNERSHIP WORKFLOW"
  cat <<'EOF'
This workbench cannot alter Apple's server-side account association.
For a device linked to a previous owner, have that owner remove it
from their account, or contact Apple with valid proof of purchase.
Do not enter account passwords or verification codes into scripts.
EOF
}

while true; do
  clear
  heading "DEVICE + FIRMWARE WORKBENCH"
  echo "1) Connect / inspect plugged-in device (USB diagnostics)"
  echo "2) Firmware / IPSW inspection"
  echo "3) Recovery-mode workflow"
  echo "4) Account / ownership workflow"
  echo "5) Save full diagnostic report to Desktop"
  echo "0) Exit"
  printf "\nChoose: "
  read -r choice
  case "$choice" in
    1) connection_check; pause ;;
    2) firmware_info; pause ;;
    3) recovery_steps; pause ;;
    4) account_steps; pause ;;
    5) make_report; pause ;;
    0) exit 0 ;;
    *) echo "Invalid choice."; pause ;;
  esac
done
'''
p = Path("/mnt/data/device_firmware_workbench.sh")
p.write_text(script, encoding="utf-8")
p.chmod(0o755)
print(f"Created terminal script: {p}")
# fcku

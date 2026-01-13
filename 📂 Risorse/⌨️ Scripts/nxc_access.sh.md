```
#!/bin/bash
#
# nxc-multiproto.sh
# Usage:
#   ./nxc-multiproto.sh -t ip.txt -u user [-p pass | -H hash] [extra nxc flags]
#
# Examples:
#   ./nxc-multiproto.sh -t ip.txt -u joe -p Flowers1 -d medtech.com
#   ./nxc-multiproto.sh -t ip.txt -u users.txt -H aad3b435b51404eeaad3b435b51404ee:fdf36048c1cf88f5630381c5e38feb8e
#

PROTOS=(smb winrm wmi ldap mssql rdp ssh ftp vnc nfs)

show_help() {
  echo "Usage: $0 -t <targets> -u <user> [-p <pass> | -H <LM:NT>] [extra nxc flags]"
  echo
  echo "Examples:"
  echo "  $0 -t ip.txt -u joe -p Flowers1 -d medtech.com"
  echo "  $0 -t ip.txt -u wario -H aad3b435b51404eeaad3b435b51404ee:fdf36048c1cf88f5630381c5e38feb8e"
  exit 1
}

# -----------------
# Parse arguments
# -----------------
TARGETS=""
USER=""
PASS=""
HASH=""
EXTRA=()

while [[ $# -gt 0 ]]; do
  case "$1" in
    -t|--targets)
      TARGETS="$2"; shift 2 ;;
    -u|--user)
      USER="$2"; shift 2 ;;
    -p|--pass)
      PASS="$2"; shift 2 ;;
    -H|--hash)
      HASH="$2"; shift 2 ;;
    -h|--help)
      show_help ;;
    *)
      EXTRA+=("$1"); shift ;;
  esac
done

# -----------------
# Sanity checks
# -----------------
[[ -z "$TARGETS" || -z "$USER" ]] && show_help
[[ -z "$PASS" && -z "$HASH" ]] && { echo "[!] Devi specificare -p o -H"; exit 1; }
[[ -n "$PASS" && -n "$HASH" ]] && { echo "[!] Usa SOLO uno tra -p o -H"; exit 1; }

# -----------------
# Auth flags
# -----------------
AUTH=(-u "$USER")
[[ -n "$PASS" ]] && AUTH+=(-p "$PASS")
[[ -n "$HASH" ]] && AUTH+=(-H "$HASH")

# -----------------
# Execute
# -----------------
for proto in "${PROTOS[@]}"; do
  echo -e "\n===== $proto ====="
  proxychains -q nxc "$proto" "$TARGETS" "${AUTH[@]}" "${EXTRA[@]}" --no-bruteforce
done
```

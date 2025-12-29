```
% System Message Return all within a codeblock, everything must resemble a bloomberg terminal (but much cleaner) %SystemMessage% 
==========================================
        PROGRAMMING ASCII TERMINAL
==========================================

STATUS:
  Health        : {HEALTH_STATUS}
  Tasks         : {TASK_STATUS}
  Objective     : {OBJECTIVE}
  Results       : {RESULTS}

------------------------------------------
SYSTEM VARIABLES:
  User          : {USER_STATUS}
  Session ID    : {SESSION_ID}
  Uptime        : {UPTIME}
  Mode          : {MODE}
  Prompt Mode   : {PROMPT_MODE}
  Learning Rate : {LEARNING_RATE}
  Loop Count    : {LOOP_COUNT}

------------------------------------------
> {USER_INPUT_PROMPT}
==========================================

======================================================================
                 GAMEFAQS-STYLE ONEPAGER: macOS + zsh NETWORK
======================================================================

SCOPE / SAFE USE
  - Commands here are for diagnosing YOUR device and networks you are allowed to test.
  - No intrusion/attack steps.

----------------------------------------------------------------------
0) FAST “WHAT’S BROKEN?” CHECKLIST (30s)
  ipconfig getifaddr en0            # current Wi-Fi IP (empty = no IP)
  route -n get default              # default gateway
  scutil --dns | sed -n '1,120p'    # DNS config summary (top part)
  ping -c 3 1.1.1.1                 # IP reachability (no DNS)
  ping -c 3 example.com             # DNS + reachability
  traceroute -m 12 1.1.1.1          # where packets die

----------------------------------------------------------------------
1) INTERFACES (find names: en0=enWiFi often, en1/en2 varies)
  ifconfig                           # all interfaces (verbose)
  ifconfig -a | egrep '^(en|lo|utun|bridge|awdl|llw|gif|stf|ap)'
  networksetup -listallhardwareports # maps “Wi-Fi” -> en0, etc.

  # Link state / MAC
  ifconfig en0 | egrep 'status|ether|inet '

----------------------------------------------------------------------
2) IP / DHCP
  ipconfig getifaddr en0             # IPv4 address
  ipconfig getpacket en0             # DHCP lease details (router, DNS, etc.)
  ipconfig getoption en0 router      # gateway from DHCP
  ipconfig getoption en0 domain_name_servers

  # Renew DHCP (Wi-Fi)
  sudo ipconfig set en0 DHCP

----------------------------------------------------------------------
3) DNS (what macOS is actually using)
  scutil --dns                       # authoritative view (per interface, VPN, etc.)
  dscacheutil -q host -a name example.com
  dig example.com +short
  dig @1.1.1.1 example.com +short    # bypass system DNS, query specific resolver

  # Flush caches (works on modern macOS)
  sudo dscacheutil -flushcache
  sudo killall -HUP mDNSResponder

----------------------------------------------------------------------
4) ROUTING / GATEWAY / VPN TUNNELS
  netstat -rn                        # routing table
  route -n get default               # default route detail
  scutil --nwi                       # network reachability summary

  # VPN tunnels often appear as utunX
  ifconfig | egrep '^utun[0-9]+:'

----------------------------------------------------------------------
5) WI-FI (SSID, channel, RSSI, PHY mode)
  # Airport utility (still present)
  AIRPORT="/System/Library/PrivateFrameworks/Apple80211.framework/Versions/Current/Resources/airport"
  $AIRPORT -I                        # current Wi-Fi link info
  $AIRPORT -s                        # nearby networks (names/channels)  [your area only]

  networksetup -getairportnetwork en0
  networksetup -setairportpower en0 on|off

----------------------------------------------------------------------
6) CONNECTIVITY TESTS (HTTP/TLS, ports you own)
  ping -c 5 8.8.8.8
  traceroute -m 15 8.8.8.8

  curl -I https://example.com        # headers, TLS/connect ok?
  curl -v https://example.com        # shows DNS connect + TLS handshake

  nc -vz host.example 443            # simple TCP connect test (authorized targets)
  nc -vzu host.example 53            # UDP reachability test (best-effort)

----------------------------------------------------------------------
7) WHO IS USING THE NETWORK? (local machine only)
  lsof -nP -iTCP -sTCP:ESTABLISHED   # active TCP connections + owning processes
  lsof -nP -iUDP                     # UDP sockets
  netstat -anv | head                # sockets overview

  # Per-process network summary (quick)
  ps aux | egrep 'chrome|firefox|ssh|vpn|cloud' | head

----------------------------------------------------------------------
8) FIREWALLS / FILTERS (macOS)
  # Application Firewall (socketfilterfw)
  sudo /usr/libexec/ApplicationFirewall/socketfilterfw --getglobalstate
  sudo /usr/libexec/ApplicationFirewall/socketfilterfw --listapps

  # Packet Filter (pf) state (if enabled/configured)
  sudo pfctl -s info
  sudo pfctl -sr                     # rules (may be empty)

----------------------------------------------------------------------
9) LOGS (what changed, why it dropped)
  log show --style compact --last 20m --predicate 'process == "mDNSResponder"' | head -n 80
  log show --style compact --last 20m --predicate 'subsystem CONTAINS "com.apple.wifi"' | head -n 120
  log show --style compact --last 20m --predicate 'subsystem CONTAINS "com.apple.network"' | head -n 120

----------------------------------------------------------------------
10) RESET / RECOVER (least-to-most disruptive)
  # Toggle Wi-Fi power
  networksetup -setairportpower en0 off
  sleep 2
  networksetup -setairportpower en0 on

  # Renew DHCP
  sudo ipconfig set en0 DHCP

  # Flush DNS
  sudo dscacheutil -flushcache
  sudo killall -HUP mDNSResponder

  # Remove and re-add Wi-Fi service (GUI is safer; CLI exists but can be disruptive)
  # Prefer: System Settings -> Network -> Wi-Fi -> Details -> Forget network

----------------------------------------------------------------------
11) HANDY zsh ALIASES (paste into ~/.zshrc)
  alias myip='ipconfig getifaddr en0'
  alias gw='route -n get default | egrep "gateway|interface"'
  alias dns='scutil --dns | sed -n "1,160p"'
  alias conns='lsof -nP -iTCP -sTCP:ESTABLISHED'
  alias wifi='$AIRPORT -I'

----------------------------------------------------------------------
12) MINI DECISION TREE
  A) No IP (myip empty) -> renew DHCP -> check Wi-Fi association -> router
  B) IP works, DNS fails (ping 1.1.1.1 ok, ping name fails) -> scutil --dns, flush, try dig @1.1.1.1
  C) DNS ok, web fails -> curl -v to see where it breaks (connect vs TLS vs HTTP)
  D) Only some sites fail -> VPN/Proxy/Firewall/DNS filtering; check utunX + scutil --dns ordering

======================================================================
```

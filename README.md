# openvpn-auto-installer

[![GitHub](https://img.shields.io/badge/GitHub-alvinjaison%2Fopenvpn--auto--installer-blue?logo=github)](https://github.com/alvinjaison/openvpn-auto-installer)

A bash script to install, configure, and manage a secure OpenVPN server with a single command. Supports both interactive and fully non-interactive (headless/CI) modes.

**Author:** Alvin Jaison  
**Repository:** https://github.com/alvinjaison/openvpn-auto-installer

---

## Supported Operating Systems

| OS | Versions |
|----|----------|
| Debian | 11+ |
| Ubuntu | 18.04+ |
| CentOS / Rocky Linux / AlmaLinux | 8+ |
| Fedora | Latest |
| Amazon Linux | 2023.6+ |
| Oracle Linux | 8+ |
| Arch Linux | Rolling |
| openSUSE | Leap 16+, Tumbleweed |

---

## Requirements

- Root privileges
- `/dev/net/tun` available (TUN device)
- `curl`, `openssl`, `iptables` (installed automatically if missing)

---

## Quick Start

```bash
# Download
curl -O https://raw.githubusercontent.com/alvinjaison/openvpn-auto-installer/main/openvpn-auto-installer.sh
chmod +x openvpn-auto-installer.sh

# Interactive install (guided wizard)
sudo ./openvpn-auto-installer.sh install -i

# Non-interactive install with defaults
sudo ./openvpn-auto-installer.sh install
```

---

## Usage

```
openvpn-auto-installer <command> [options]

Commands:
  install       Install and configure OpenVPN server
  uninstall     Remove OpenVPN server
  client        Manage client certificates
  server        Server management
  interactive   Launch interactive menu

Global Options:
  --verbose     Show detailed output
  --log <path>  Log file path (default: openvpn-install.log)
  --no-log      Disable file logging
  --no-color    Disable colored output
  -h, --help    Show help
```

---

## Commands

### `install`

Install and configure the OpenVPN server.

```bash
sudo ./openvpn-auto-installer.sh install [options]
```

**Network Options**

| Flag | Description | Default |
|------|-------------|---------|
| `--endpoint <host>` | Public IP or hostname for clients | Auto-detected |
| `--endpoint-type <4\|6>` | Endpoint IP version | `4` |
| `--ip <addr>` | Server listening IP | Auto-detected |
| `--client-ipv4` / `--no-client-ipv4` | Enable/disable IPv4 for VPN clients | Enabled |
| `--client-ipv6` / `--no-client-ipv6` | Enable/disable IPv6 for VPN clients | Disabled |
| `--subnet-ipv4 <x.x.x.0>` | IPv4 VPN subnet | `10.8.0.0` |
| `--subnet-ipv6 <prefix>` | IPv6 VPN subnet | `fd42:42:42:42::` |
| `--port <num>` | OpenVPN listening port | `1194` |
| `--port-random` | Use a random port (49152–65535) | — |
| `--protocol <udp\|tcp>` | Transport protocol | `udp` |
| `--mtu <size>` | Tunnel MTU | `1500` |

**DNS Options**

| Flag | Description |
|------|-------------|
| `--dns <provider>` | DNS provider (see list below) |
| `--dns-primary <ip>` | Custom primary DNS (requires `--dns custom`) |
| `--dns-secondary <ip>` | Custom secondary DNS (optional) |

Available DNS providers: `system`, `unbound`, `cloudflare`, `quad9`, `quad9-uncensored`, `fdn`, `dnswatch`, `opendns`, `google`, `yandex`, `adguard`, `nextdns`, `custom`

**Security Options**

| Flag | Description | Default |
|------|-------------|---------|
| `--cipher <cipher>` | Data channel cipher | `AES-128-GCM` |
| `--cert-type <ecdsa\|rsa>` | Certificate type | `ecdsa` |
| `--cert-curve <curve>` | ECDSA curve (`prime256v1`, `secp384r1`, `secp521r1`) | `prime256v1` |
| `--rsa-bits <size>` | RSA key size (`2048`, `3072`, `4096`) | `2048` |
| `--tls-version-min <1.2\|1.3>` | Minimum TLS version | `1.2` |
| `--hmac <alg>` | HMAC algorithm (`SHA256`, `SHA384`, `SHA512`) | `SHA256` |
| `--tls-sig <mode>` | TLS mode (`crypt-v2`, `crypt`, `auth`) | `crypt-v2` |
| `--auth-mode <pki\|fingerprint>` | Authentication mode (fingerprint requires OpenVPN 2.6+) | `pki` |
| `--server-cert-days <n>` | Server certificate validity in days | `3650` |

**Client Options (initial client created during install)**

| Flag | Description | Default |
|------|-------------|---------|
| `--client <name>` | Initial client name | `client` |
| `--client-password [pass]` | Password-protect the client key | — |
| `--client-cert-days <n>` | Client certificate validity in days | `3650` |
| `--no-client` | Skip initial client creation | — |
| `--multi-client` | Allow same certificate on multiple devices | — |

**Examples**

```bash
# Default install
sudo ./openvpn-auto-installer.sh install

# Custom port and protocol
sudo ./openvpn-auto-installer.sh install --port 443 --protocol tcp

# Quad9 DNS, AES-256 cipher
sudo ./openvpn-auto-installer.sh install --dns quad9 --cipher AES-256-GCM

# Dual-stack with IPv6 clients
sudo ./openvpn-auto-installer.sh install --client-ipv6

# Interactive wizard
sudo ./openvpn-auto-installer.sh install -i
```

---

### `client`

Manage VPN client certificates.

```bash
sudo ./openvpn-auto-installer.sh client <subcommand> [options]
```

#### `client add <name>`

```bash
sudo ./openvpn-auto-installer.sh client add alice
sudo ./openvpn-auto-installer.sh client add bob --password
sudo ./openvpn-auto-installer.sh client add charlie --cert-days 365 --output /tmp/charlie.ovpn
```

| Flag | Description |
|------|-------------|
| `--password [pass]` | Password-protect the client key |
| `--cert-days <n>` | Certificate validity in days (default: 3650) |
| `--output <path>` | Output path for `.ovpn` file (default: `~/<name>.ovpn`) |

#### `client list`

```bash
sudo ./openvpn-auto-installer.sh client list
sudo ./openvpn-auto-installer.sh client list --format json
```

#### `client revoke <name>`

```bash
sudo ./openvpn-auto-installer.sh client revoke alice
sudo ./openvpn-auto-installer.sh client revoke bob --force
```

#### `client renew <name>`

```bash
sudo ./openvpn-auto-installer.sh client renew alice
sudo ./openvpn-auto-installer.sh client renew bob --cert-days 365
```

---

### `server`

Server management commands.

#### `server status`

List currently connected VPN clients.

```bash
sudo ./openvpn-auto-installer.sh server status
sudo ./openvpn-auto-installer.sh server status --format json
```

> Note: Client data refreshes every 60 seconds.

#### `server renew`

Renew the server certificate.

```bash
sudo ./openvpn-auto-installer.sh server renew
sudo ./openvpn-auto-installer.sh server renew --cert-days 1825 --force
```

---

### `uninstall`

Completely remove OpenVPN and all configuration.

```bash
sudo ./openvpn-auto-installer.sh uninstall
sudo ./openvpn-auto-installer.sh uninstall --force
```

---

## Environment Variables

These can be set before running the script to control behaviour without CLI flags.

| Variable | Description | Default |
|----------|-------------|---------|
| `VERBOSE` | Set to `1` for detailed command output | `0` |
| `LOG_FILE` | Path to log file. Set to `""` to disable | `openvpn-install.log` |
| `OUTPUT_FORMAT` | Output format: `table` or `json` | `table` |
| `FORCE_COLOR` | Set to `1` to force colored output (e.g. in CI) | — |

**Example**

```bash
VERBOSE=1 LOG_FILE=/var/log/openvpn-setup.log sudo ./openvpn-auto-installer.sh install
```

---

## Logging

By default the script writes a timestamped log to `openvpn-install.log` in the current directory. Each entry is prefixed with a level tag:

```
2026-05-08 12:00:00 [INFO]  Installing OpenVPN...
2026-05-08 12:00:05 [OK]    OpenVPN installed successfully
2026-05-08 12:00:06 [WARN]  TLS 1.3 requires OpenVPN 2.5+ clients
2026-05-08 12:00:10 [ERROR] Failed to download EasyRSA
```

Use `--no-log` to disable file logging, or `--log <path>` to change the log location.

---

## Security Notes

- Certificates default to **10-year validity** (3650 days). Adjust with `--cert-days` / `--server-cert-days` for shorter-lived certs.
- The default cipher is **AES-128-GCM** with **ECDSA (prime256v1)** certificates and **tls-crypt-v2** — all considered secure defaults.
- **Fingerprint mode** (`--auth-mode fingerprint`) is a simplified WireGuard-like setup without a CA, suitable for small/home setups. Requires OpenVPN 2.6+.
- EasyRSA is downloaded from the official GitHub release and its **SHA256 checksum is verified** before use.

---

## License

MIT — see [LICENSE](https://github.com/alvinjaison/openvpn-auto-installer/blob/main/LICENSE)

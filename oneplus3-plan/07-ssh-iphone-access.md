# 07 — Step 7: sshd + iPhone File Editing over Tailscale

Goal: edit files on the phone from an iPhone, from any network.

**Mechanism (from architecture):** Tailscale app on iPhone (official, iOS-supported) + an SFTP client that registers as an iOS Files provider — **Secure ShellFish** suggested (common choice for this) — connecting to the phone's `sshd` on port 8022. Because it runs over Tailscale, it works on any network, not just same-WiFi.

The SSH details below are copied from the repo [README.md](../README.md). The README assumes same-WiFi connections; over Tailscale, substitute the phone's Tailscale IP/MagicDNS name for the WiFi IP.

---

## First-time SSH setup (from README)

OpenSSH is installed automatically by the setup script. In the regular Termux app (not inside the desktop):

```bash
# Start the SSH server
sshd

# Set your Termux password (you'll use this to log in over SSH)
passwd
```

Find the phone's (WiFi) IP address:

```bash
ip addr show wlan0 | grep 'inet '
```

Output looks like `inet 192.168.1.42/24` — the IP is the part before the `/`.

Find your Termux username with `whoami` — typically something like `u0_a123`.

> **Port 8022** is the default Termux SSH port (not the standard port 22, which requires root).

## Connect from a computer (from README)

```bash
ssh your-termux-username@192.168.1.42 -p 8022
```

Optional `~/.ssh/config` entry on the PC:

```
Host myphone
    HostName 192.168.1.42
    User u0_a123
    Port 8022
```

Then just `ssh myphone`.

## File transfer with SCP or SFTP (from README)

```bash
# PC → phone
scp -P 8022 myfile.txt u0_a123@192.168.1.42:~/

# phone → PC
scp -P 8022 u0_a123@192.168.1.42:~/somefile.txt ./
```

Or any SFTP client — connect to the same IP and port 8022. (This is exactly what Secure ShellFish on the iPhone does.)

## Keep SSH running when Termux is closed (from README)

```bash
# Add to ~/.bashrc so sshd auto-starts whenever Termux opens
echo 'sshd 2>/dev/null' >> ~/.bashrc
```

For unattended reboots, also start `sshd` from Termux:Boot alongside the bot (plan addition — see [06-discord-bot-service.md](06-discord-bot-service.md)).

## SSH key authentication — passwordless login (from README)

```bash
# On the client, generate a key pair if you don't have one
ssh-keygen -t ed25519

# Copy the public key to the phone
ssh-copy-id -p 8022 u0_a123@192.168.1.42
```

## Troubleshooting (from README)

**SSH connection refused** — make sure `sshd` is running (`ps aux | grep sshd`). If not, run `sshd` again. Confirm you're using port 8022, not 22.

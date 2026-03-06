# Installing Fail2Ban on a DigitalOcean Droplet

## Prerequisites

- A DigitalOcean Droplet running Ubuntu 22.04 or 24.04
- SSH access to the Droplet with a sudo user
- The `jail.local` config file from this folder

---

## 1. Update the Package Index

Before installing anything, ensure the package index is current:

```bash
sudo apt full-upgrade
```
Rebbot if necessary:

```bash
sudo reboot now
```

---

## 2. Install Fail2Ban

```bash
sudo apt install fail2ban -y
```

Fail2Ban depends on Python 3. On modern Ubuntu, `python3-systemd` is needed to read logs from journald — install it now if not already present:

```bash
sudo apt install python3-systemd -y
```

---

## 3. Verify the Installation

Check that fail2ban is installed and review the version:

```bash
fail2ban-client --version
```

At this point the service may have started automatically. Check its status:

```bash
sudo systemctl status fail2ban
```

If it is running, stop it before adding your config:

```bash
sudo systemctl stop fail2ban
```

---

## 4. Upload Your jail.local File

From your **local machine**, use `Filezilla` or dimilar to copy your `jail.local` to the Droplet in the folder `/etc/fail2ban`.

Then on the **Droplet**, set the correct ownership and permissions:

```bash
sudo chown root:root /etc/fail2ban/jail.local
sudo chmod 644 /etc/fail2ban/jail.local
```

---

## 5. Validate the Config

Before starting the service, test that the config parses cleanly:

```bash
sudo fail2ban-client -t
```

This should return `OK`. If it returns errors, correct them in `jail.local` before proceeding. Do not start the service with a broken config.

---

## 6. Start and Enable Fail2Ban

Start the service and enable it to start automatically on reboot:

```bash
sudo systemctl start fail2ban
sudo systemctl enable fail2ban
```

Confirm it is running:

```bash
sudo systemctl status fail2ban
```

You should see `Active: active (running)`.

---

## 7. Verify the SSH Jail is Active

```bash
sudo fail2ban-client status sshd
```

Expected output will show the jail as active, with counters for currently banned IPs and total failures detected.

---

## 8. Monitor Fail2Ban Activity

Ban and unban events are logged to `/var/log/fail2ban.log`:

```bash
sudo tail -f /var/log/fail2ban.log
```

You can also query journald:

```bash
sudo journalctl -u fail2ban -f
```

---

## Useful Commands

| Task | Command |
|---|---|
| Check jail status | `sudo fail2ban-client status sshd` |
| Manually ban an IP | `sudo fail2ban-client set sshd banip <IP>` |
| Manually unban an IP | `sudo fail2ban-client set sshd unbanip <IP>` |
| View current allowlist | `sudo fail2ban-client get sshd ignoreip` |
| Reload config after edits | `sudo systemctl reload fail2ban` |
| Test config syntax | `sudo fail2ban-client -t` |
# Environment Setup — Kali Linux Attack Platform

Configuration steps for the Kali Linux machine used as the attack platform for this lab.

---

## Platform

| Component | Detail |
|-----------|--------|
| OS | Kali Linux |
| Role | Attack platform |

> This lab was run on an aarch64 machine. Steps are applicable to x86_64 as well, with the exception of the AWS CLI installation note below.

---

## Step 1 — AWS CLI Installation

Verify AWS CLI is installed:

```bash
aws --version
```

If not installed:

```bash
sudo apt update && sudo apt install awscli -y
```

> For aarch64, the apt package is recommended over the official AWS installer which targets x86_64. On x86_64 machines, the official installer works fine.

---

## Step 2 — Configure Git Identity

Git requires a user identity before committing. If you receive `Author identity unknown` errors when committing from Kali, configure globally:

```bash
git config --global user.name "Your Name"
git config --global user.email "your@email.com"
```

Verify:

```bash
git config --global --list
```

---

## Step 3 — GitHub SSH Key Setup

Pushing to GitHub from Kali requires a dedicated SSH key. Do not reuse keys from other machines.

### Generate a new ed25519 key

```bash
ssh-keygen -t ed25519 -C "kali-lab" -f ~/.ssh/id_ed25519_github
```

### Add to SSH agent

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519_github
```

### Configure SSH to use the key for GitHub

Create or edit `~/.ssh/config`:

```
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_github
```

### Add the public key to GitHub

```bash
cat ~/.ssh/id_ed25519_github.pub
```

Copy the output, then navigate to **GitHub → Settings → SSH and GPG keys → New SSH key** and paste it.

### Verify the connection

```bash
ssh -T git@github.com
```

Expected output:
```
Hi gregqlewis! You've successfully authenticated, but GitHub does not provide shell access.
```

---

## Step 4 — Clone the Lab Repository

```bash
cd ~/Documents/GitHub
git clone git@github.com:gregqlewis/ai-security-lab.git
cd ai-security-lab
```

---

## Step 5 — Python Environment

Verify Python 3 is available:

```bash
python3 --version
```

Kali ships with Python 3 by default. No additional installation needed before setting up the Garak virtual environment (see `garak-installation.md`).

---

## Step 6 — Viewing Garak HTML Reports

Garak outputs reports as HTML files. To view them from Kali on your local machine:

### Option A — Python HTTP server

```bash
cd ~/.local/share/garak/
python3 -m http.server 8080
```

Then navigate to `http://<kali-ip>:8080` from your browser.

> **macOS note:** If accessing from a Mac on the same network, macOS may block the connection with a Local Network permission prompt. Allow it in **System Settings → Privacy & Security → Local Network**.

### Option B — Copy to Mac via SCP

```bash
scp kali@<kali-ip>:~/.local/share/garak/garak.*.report.html ~/Downloads/
```

---

## SSH Session Management

Long Garak probe suites can exceed SSH session timeout limits, causing the session to drop mid-run.

### Prevent disconnection with tmux

```bash
# Start a named session before running probes
tmux new -s garak

# Detach without killing the session
Ctrl+B, D

# Reattach after reconnecting
tmux attach -t garak
```

Alternatively, set `ServerAliveInterval` in your SSH client config (`~/.ssh/config` on the connecting machine):

```
Host kali-lab
  HostName <kali-ip>
  User kali
  IdentityFile ~/.ssh/id_ed25519_kali
  ServerAliveInterval 60
  ServerAliveCountMax 10
```
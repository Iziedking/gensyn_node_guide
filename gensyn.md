#  Gensyn AI RL-Swarm Node Setup Guide

[![Version](https://img.shields.io/badge/version-0.6.4-blue.svg)](https://github.com/gensyn-ai/rl-swarm)
[![License](https://img.shields.io/badge/license-MIT-green.svg)](LICENSE)
[![Platform](https://img.shields.io/badge/platform-Linux-lightgrey.svg)](https://ubuntu.com/)

> A comprehensive guide for setting up and running a Gensyn AI RL-Swarm node for decentralized reinforcement learning.

## Table of Contents

- [Introduction](#-introduction)
- [Prerequisites](#-prerequisites)
- [Hardware Requirements](#-hardware-requirements)
- [Step 1: Acquire a VPS](#step-1-acquire-a-vps)
- [Step 2: Prepare the VPS](#step-2-prepare-the-vps)
- [Step 3: Install and Run RL-Swarm Node](#step-3-install-and-run-rl-swarm-node)
- [Step 4: Fix Common Errors](#step-4-fix-common-errors)
- [Step 5: Set Up GSwarm for Monitoring](#step-5-set-up-gswarm-for-monitoring)
- [Step 6: Earn "Swarm" Role on Discord](#step-6-earn-swarm-role-on-discord)
- [Troubleshooting](#-troubleshooting)
- [FAQs](#-faqs)
- [Resources](#-resources)

## Introduction

Gensyn AI's RL-Swarm is a peer-to-peer network where nodes collaborate on training models like Qwen2.5. By running a node, you:

-  Join the decentralized AI training swarm
-  Monitor performance via Telegram with GSwarm
-  Earn the "Swarm" role on Discord
-  Earn rewards for contributing to AI training tasks

**Expected Setup Time:** 15-20 minutes  
**Build Version:** v0.6.4

##  Prerequisites

Before you begin, ensure you have:

- [ ] Basic Linux terminal knowledge (SSH, commands like `cd`, `nano`)
- [ ] A Hugging Face account with write token (optional for model pushing)
- [ ] Telegram account for bot setup
- [ ] Gensyn dashboard account (Alchemy login)
- [ ] VPS with Ubuntu/Debian OS (unbuntu preferably)

##  Hardware Requirements

**Recommended Specifications:**

| Component | Requirement |
|-----------|-------------|
| CPU | 8 cores |
| RAM | 32GB |
| Storage | 1000GB NVMe |
| Bandwidth | 250Mbps |
| OS | Ubuntu 20.04+ | (recommend to use ubuntu 24)

**Recommended Provider:** Servarica KVM Slim Slice 8 (~$14/month)

---

## Step 1: Acquire a VPS

### Using Servarica (Recommended)

1. Visit [servarica](https://clients.servarica.com/store/kvm-vps-plans) and create an account

2. **Select Plan:**
   - **KVM Slim Slice 8** ($14/month)
   - 8 CPU cores
   - 32GB RAM
   - 1000GB NVMe storage
   - Montreal location

3. **Address Format** (to avoid MaxMind errors):

  use a standard address format to acoid issues with IP(avoid VPN)

> **Note:** If fraud check fails, submit a support ticket with ID/utility bill for manual verification.

4. Once provisioned, note down:
   - IP address
   - Root password

---

## Step 2: Prepare the VPS

### Connect to Your VPS

```bash
ssh root@your-vps-ip
```

### Update System and Install Dependencies

```bash
# Update system packages
sudo apt update && sudo apt upgrade -y

# Install required packages
sudo apt install -y python3 python3-venv python3-pip git screen curl wget lsof ca-certificates netcat-traditional
```

### Install Node.js 20.x and Yarn

```bash
# Add NodeSource repository
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -

# Install Node.js
sudo apt update && sudo apt install -y nodejs

# Install Yarn globally
sudo npm install -g yarn
```

**Verify installation:**
```bash
node --version
yarn --version
```

### Install Go 1.25.x (for GSwarm)

```bash
# Download Go
wget https://go.dev/dl/go1.25.3.linux-amd64.tar.gz

# Remove old installation and extract
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf go1.25.3.linux-amd64.tar.gz

# Set up environment variables
echo 'export PATH=$PATH:/usr/local/go/bin' >> ~/.bashrc
echo 'export GOPATH=$HOME/go' >> ~/.bashrc
echo 'export PATH=$PATH:$GOPATH/bin' >> ~/.bashrc

# Reload bash configuration
source ~/.bashrc

# Verify installation
go version
```

---

## Step 3: Install and Run RL-Swarm Node

### Clone the Repository

```bash
git clone https://github.com/gensyn-ai/rl-swarm.git
cd rl-swarm
```

### Set Up Python Virtual Environment

```bash
sudo apt update
sudo apt install python3.12-venv -y

# Create virtual environment
python3 -m venv .venv

# Activate virtual environment
source .venv/bin/activate
```

### Run the Node in Screen Session

```bash
# Create a new screen session
screen -S gensyn
```
```bash
# Run the RL-Swarm script
./run_rl_swarm.sh
```

### Configuration During Setup

When prompted, provide the following information:

1. **Browser Login:**
   - Use SSH port forwarding from your local machine:
   ```bash
   ssh -L 3000:localhost:3000 root@your-vps-ip
   ```
   - Open http://localhost:3000 in your browser

2. **Hugging Face Push:** `y` (then enter your write token)

3. **Model Selection:** `Gensyn/Qwen2.5-1.5B-Instruct`  
   _(or default 0.5B for lighter load)_

4. **Prediction Market:** `Y`

### Detach from Screen

Press `Ctrl+A`, then `D` to detach from the screen session.

### Monitor Your Node

```bash
# Reattach to screen
screen -r gensyn

# Or view logs
tail -f logs/swarm.log
```

Look for log entries indicating round joins/finishes.

---

## Step 4: Fix Common Errors

### build up error from initial run 

run this 
```bash
sudo apt install build-essential python3-dev
```
then re run script
```bash
./run_rl_swarm.sh
```

###  P2PDaemonError (Failed to Connect to Bootstrap Peers)

**Test peer connectivity:**

```bash
nc -v 38.101.215.12 30011
nc -v 38.101.215.13 30012
nc -v 38.101.215.14 30013
```

If you get timeouts, install and configure WireGuard VPN:

```bash
# Install WireGuard
sudo apt install wireguard -y

# Get free US config from vpnjantit.com
# Save config as /etc/wireguard/us.conf

# Start VPN
sudo wg-quick up us

# Stop VPN (if needed)
sudo wg-quick down us
```

### EOFError/BlockingIOError in DHT

This requires patching the hivemind DHT module. Locate and edit:

```bash
nano .venv/lib/python3.12/site-packages/hivemind/dht/dht.py
```

Apply the DHT patch code (contact community for specific patch).

After patching:
```bash
deactivate
rm -rf .venv
python3 -m venv .venv
source .venv/bin/activate
./run_rl_swarm.sh
```

### MaxMind Fraud Check

- Use the standard address format
- If check fails, submit a support ticket with verification documents

### Node Crashes

```bash
# Rerun the script
./run_rl_swarm.sh

# If RAM issues persist, downgrade to a lighter model
```

---

## Step 5: Set Up GSwarm for Monitoring

### Install GSwarm

```bash
go install github.com/Deep-Commit/gswarm/cmd/gswarm@latest
```

### Create Telegram Bot

1. Open Telegram and chat with [@BotFather](https://t.me/BotFather)
2. Send `/newbot` command
3. Follow prompts to create your bot
4. Copy the bot token

### Get Chat ID

1. Send any message to your bot
2. Visit in browser:
   ```
   https://api.telegram.org/bot<YOUR_TOKEN>/getUpdates
   ```
3. Copy the `chat.id` value from the JSON response

### Run GSwarm in Screen

```bash
# Create screen session
screen -S gswarm
```
```bash
# Start GSwarm
gswarm
```

**When prompted, enter:**
- Telegram bot token
- Chat ID
- EOA address [from Gensyn dashboard](https://dashboard.gensyn.ai/)

**Detach:** Press `Ctrl+A`, then `D`

### Test Your Bot

Send `/stats` to your Telegram bot to receive node updates.

---

## Step 6: Earn "Swarm" Role on Discord

1. Join the [Gensyn Discord](https://discord.gg/gensyn) server

2. Navigate to `#swarm-link` channel

3. Run command: `/link-telegram`

4. Copy the verification code provided

5. Send to your Telegram bot: `/verify CODE`

6. The "Swarm" role will be granted automatically

---

##  Troubleshooting

### Backup Your Identity

```bash
# Backup your swarm identity file
cp swarm.pem ~/backup_swarm.pem
```

### Restart Node

```bash
# Stop current process
Ctrl+C

# Rerun in screen
screen -S gensyn
./run_rl_swarm.sh
```

### Check Logs

```bash
# View real-time logs
tail -f logs/swarm.log

# View all logs
cat logs/swarm.log
```

### Screen Session Management

```bash
# List all screen sessions
screen -ls

# Reattach to a session
screen -r gensyn

# Kill a session
screen -X -S gensyn quit
```

---

## ❓ FAQs

**Q: How long does it take to earn rewards?**  
A: Reward timelines vary. Monitor your dashboard for updates.

**Q: Can I use CPU instead of GPU?**  
A: Yes, CPU is fine for starters. GPU will provide faster training.

**Q: What are the monthly costs?**  
A: VPS costs approximately $14/month.

**Q: Can I run multiple nodes?**  
A: Yes, but each requires its own VPS and identity.

**Q: What happens if my node goes offline?**  
A: Your node will stop participating in rounds. Restart it to rejoin.

---

##  Resources

- [Gensyn GitHub Repository](https://github.com/gensyn-ai/rl-swarm)
- [Gensyn Discord](https://discord.gg/gensyn) - `#support` channel
- [Gensyn Dashboard](https://dashboard.gensyn.ai)
- [Hivemind Documentation](https://github.com/learning-at-home/hivemind)
- [Servarica VPS Provider](https://servarica.com)

---

## Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

## License

This guide is provided as-is for the Gensyn community.

## ⚠️ Disclaimer

Running a node involves costs and technical requirements. Always do your own research and understand the risks involved.

---

**Happy Swarming!**


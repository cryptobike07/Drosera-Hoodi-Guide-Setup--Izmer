# Drosera Trap & Operator Setup Guide (Hoodi Testnet)

This repository documents the setup and deployment of a **Drosera Trap** and a **Drosera Operator** node running on the Hoodi Ethereum testnet.

---

## Table of Contents

- [Prerequisites](#prerequisites)
- [Install Dependencies](#install-dependencies)
- [Install Docker](#install-docker)
- [Drosera Trap Setup](#drosera-trap-setup)
- [Trap Configuration (](#trap-configuration-droseratoml)[`drosera.toml`](#trap-configuration-droseratoml)[)](#trap-configuration-droseratoml)
- [Apply the Trap Config](#apply-the-trap-config)
- [Check Trap in Dashboard](#check-trap-in-dashboard)
- [Bloom Boost Trap](#bloom-boost-trap)
- [Drosera Operator Setup](#drosera-operator-setup)
- [Register Your Operator](#register-your-operator)
- [Firewall Configuration](#firewall-configuration)
- [Useful Commands & Updates](#useful-commands--updates)
- [Contact and Support](#contact-and-support)

---

## Prerequisites

- Ubuntu/Linux environment (WSL Ubuntu works well)
- At least 4 CPU cores and 8GB RAM recommended
- Basic CLI knowledge
- Ethereum private key with funds on Hoodi testnet
- Open ports: `31313` and `31314` (or your configured ports)

---

## Install Dependencies

```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

---

## Install Docker

```bash
sudo apt update -y && sudo apt upgrade -y

# Remove old docker packages if exist
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done

sudo apt-get update

sudo apt-get install ca-certificates curl gnupg -y

sudo install -m 0755 -d /etc/apt/keyrings

curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg

sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y

sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y

# Test Docker
sudo docker run hello-world
```

---

## 1.Drosera Trap Setup

### Install Required Tools

```bash
# Drosera CLI
curl -L https://app.drosera.io/install | bash
source ~/.bashrc
droseraup

# Foundry CLI (Solidity development)
curl -L https://foundry.paradigm.xyz | bash
source ~/.bashrc
foundryup

# Bun (JavaScript runtime)
curl -fsSL https://bun.sh/install | bash
source ~/.bashrc
```

### Initialize Trap Project

```bash
mkdir ~/my-drosera-trap
cd ~/my-drosera-trap

git config --global user.email "your_github_email@example.com"
git config --global user.name "your_github_username"

forge init -t drosera-network/trap-foundry-template
```

### Build Trap

```bash
bun install
forge build
```

---
### Edit Trap Configuration

```bash
cd my-drosera-trap
nano drosera.toml
```
Add the following codes at `drosera.toml`:

## Trap Configuration (`drosera.toml`)

```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.helloworld]
path = "out/HelloWorldTrap.sol/HelloWorldTrap.json"
response_contract = "0x183D78491555cb69B68d2354F7373cc2632508C7"
response_function = "helloworld(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR_OPERATOR_WALLET_ADDRESS"]

# New Users:
# address = "DELETE THIS LINE WHEN APPLYING" (it will generate address after apply trap config)

# Existing Users:
# If you've deployed a trap with your wallet previously (Hoodi not Holesky), add your trap address here:
# address = "TRAP_ADDRESS"
```

---

## Apply the Trap Config (New users) / Re-apply (Existing users)

```bash
DROSERA_PRIVATE_KEY=your_eth_private_key_here drosera apply
```
---

## Check Trap in Dashboard

Go to [https://app.drosera.io/](https://app.drosera.io/)\
Connect your Drosera EVM wallet\
Change network to Hoodi\
Search your trap by wallet address or trap config address generated after applying\
You can send Bloom Boost or monitor your trap here

---

## Bloom Boost Trap

Drosera lets you increase your trap’s priority on-chain by depositing Hoodie ETH to boost response speed.

To boost a trap:

```bash
drosera bloomboost --trap-address <trap_address> --eth-amount <amount>
```

---

## 2.Drosera Operator Setup

### Create Drosera Network Folder

```bash
mkdir ~/Drosera-Network
cd ~/Drosera-Network
```

### Download & Install Operator CLI
```bash
curl -LO https://github.com/drosera-network/releases/releases/download/v1.20.0/drosera-operator-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
sudo cp drosera-operator /usr/bin/
drosera-operator --version
```

Check for latest releases here:\
[https://github.com/drosera-network/releases/releases](https://github.com/drosera-network/releases/releases)

###  Edit `docker-compose.yaml`

Edit `docker-compose.yaml` file:
```bash
nano docker-compose.yaml
```

## Docker Compose (docker-compose.yaml)

```yaml
version: '3'
services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:v1.20.0
    container_name: drosera-operator
    network_mode: host
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
    volumes:
      - drosera_data:/data
    command: ["node"]
    restart: always

volumes:
  drosera_data:
```

### Create .env file

Create a `.env` file in the same folder as your `docker-compose.yaml`:

```env
ETH_PRIVATE_KEY=your_eth_private_key_here
VPS_IP=your_vps_public_ip_here
```
### Install Docker Image

```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

## Compose & Run Operator
```bash
# Stop & remove existing Docker volumes (clean start)
docker compose down -v

# Stop and remove any old container named 'drosera-node' (if exists)
docker stop drosera-node
docker rm drosera-node

# Start the operator in detached mode
docker compose up -d

# View live logs
docker compose logs -f
```


---

## Register Your Operator

```bash
drosera-operator register \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

## Opt-in your trap config

```bash
drosera-operator optin \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --trap-config-address your_trap_address_here
```

---

## Firewall Configuration

```bash
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
sudo ufw enable
```

---

## Useful Commands & Updates

```bash
# View operator logs
docker logs -f drosera-operator

# Stop operator
docker compose down

# Restart operator
docker compose restart

# Reapply trap config after changes
DROSERA_PRIVATE_KEY=your_eth_private_key_here drosera apply

# Update droseraup utility (run regularly)
curl -L https://app.drosera.io/install | bash
# or simply
droseraup
```

---

## Folder Structure Notes

Drosera Trap folder: `~/my-drosera-trap`\
Drosera Operator folder: `~/Drosera-Network`

---

## Contact and Support

Official docs: [https://dev.drosera.io/](https://dev.drosera.io/)\
Discord: [https://discord.com/invite/drosera](https://discord.com/invite/drosera)


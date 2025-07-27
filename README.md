# Drosera Trap & Operator Setup Guide (Hoodi Testnet)

This repository documents the setup and deployment of a **Drosera Trap** and a **Drosera Operator** node running on the Hoodi Ethereum testnet.

## 1. Prerequisites

- Ubuntu/Linux environment (WSL Ubuntu works well)
- At least 4 CPU cores and 8GB RAM recommended
- Basic CLI knowledge
- Metamask Wallet > Get Ethereum private key
- Add Hoodi Chain (chainlist.org)
- Get Hoodi Eth Testnet here:
  https://stakely.io/faucet/ethereum-hoodi-testnet-eth


## 2. Tutorial on Setting up VPS + Linux.
Rent a VPS VPS is (Virtual Private Server that runs 24/7) “I’m using VPS in all my Linux NODES.” Hardware Requirement: Works with simple hardware — easy to get started but I can run it in my VPS3.

How to Run Octra Node using VPS? Order Here: (Use the Link and Euro(€) For Discount) https://www.jdoqocy.com/click-101100040-15022370 Click Cloud VPS(I used VPS 3 coz I run other Nodes here) > Region Any(I used Japan) > Storage Type 1200GB > Ubuntu v22.04> Log in Password (Don’t forget) > 6–8 settings default only > Click Next > Fill up your details > Payment. “I used Gotymebank or Maya(both vitual cards) link to Paypal(all are perfored in the website). I paid €23 or 1500+ pesos for first month(+ set up fee), second only €14 or 900+ pesos .” Once Paid > Wait for the Email to Arrived > Follow instruction.

Setting up on Your Laptop/PC Windows: Now you have your own VPS server. Download putty.org > log in to your IP there. After Connected, We can now Start. Still Confuse? More Guide in VPS Access: https://www.facebook.com/share/p/zhHCh3773653iXZF/ You have now your own Ubuntu(Linux Server) that runs 24/7 in other country. Let’s Navigate!


## 3. Run Commands:

## Open Port
 Open ports: `31313` and `31314` (or your configured ports) 
```bash
ufw allow 31313
ufw allow 31314
```

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

# 1. Drosera Trap Setup

### Install Required Tools

```bash
cd #ensure you in root directory

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
cd ~/my-drosera-trap
nano drosera.toml
```
Add the following codes at `drosera.toml`:

### Trap Configuration (`drosera.toml`)

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

# New Users/Migrate:
# address = "DELETE THIS LINE WHEN APPLYING" (it will generate address after apply trap config)

# Existing Users:
# If you've deployed a trap with your wallet previously (Hoodi not Holesky), add your trap address here:
# address = "TRAP_ADDRESS"
```

---

### Apply the Trap Config (New users/Migrate) / Re-apply (Existing users)

```bash
DROSERA_PRIVATE_KEY=your_eth_private_key_here drosera apply
```
### After apply it will generate address automatically (New users/Migrate)
![drosera toml](Asset/drosera%20toml.png)
---

### Check Trap in Dashboard

Go to [https://app.drosera.io/](https://app.drosera.io/)\
Connect your Drosera EVM wallet\
Change network to Hoodi\
Search your trap by wallet address or trap config address generated after applying\
You can send Bloom Boost or monitor your trap here
![change network](Asset/change%20network.png)

---

### Bloom Boost Trap ([use dashboard instead click here](#configure-through-dashboard-instead))

Drosera lets you increase your trap’s priority on-chain by depositing Hoodi ETH to boost response speed.

To boost a trap:

```bash
drosera bloomboost --trap-address <trap_address> --eth-amount <amount>
```

---

# 2. Drosera Operator Setup
## A. Choose Docker or SystemD

Choose one Installation Method:

- [Method 1: Install using Docker (click here)](#method-1-install-using-docker)
- [Method 2: Install using SystemD (click here)](#method-2-install-using-systemd)

# Method 1: Install using Docker
### Create Drosera Network Folder

```bash
mkdir ~/Drosera-Network
cd ~/Drosera-Network
```
###  Edit `docker-compose.yaml`

Edit `docker-compose.yaml` file:
```bash
nano docker-compose.yaml
```

### Docker Compose (docker-compose.yaml) 🔴 **DON'T PUT PRIVATE KEY AND VPS IP HERE — create `.env` file**


```yaml
version: '3.8'

services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    ports:
      - "31313:31313"   # P2P
      - "31314:31314"   # HTTP
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
      - RUST_LOG=info,drosera_operator=debug
      - DRO__ETH__RPC_TIMEOUT=30s
      - DRO__ETH__RETRY_COUNT=5
      # Optional: increase internal peer retry behavior (if supported by app)
      # - DRO__NETWORK__RETRY_INTERVAL=10s
      # - DRO__NETWORK__RETRY_COUNT=5
    volumes:
      - drosera_data:/data
    restart: always   # Ensures automatic container recovery
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
    healthcheck:   # Add Docker-native watchdog
      test: ["CMD", "curl", "-f", "http://localhost:31314/health"]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 30s
    command: node

volumes:
  drosera_data:


```

### Create .env file

Create a `nano .env` file in the same folder as your `docker-compose.yaml`:

```env
ETH_PRIVATE_KEY=your_eth_private_key_here
VPS_IP=your_vps_public_ip_here
```
### Install Docker Image

```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

### Compose & Run Operator
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

# Method 2: Install using SystemD

### Remove Old Drosera SystemD Service (if any)

Before creating a new systemd service for Drosera, stop, disable, and remove any old service to avoid conflicts:
```bash
# Stop the old service if running
sudo systemctl stop drosera

# Disable the old service so it won't start on boot
sudo systemctl disable drosera

# Remove the old service file
sudo rm /etc/systemd/system/drosera.service

# Reload systemd to apply changes
sudo systemctl daemon-reload
```

---

### Prepare your environment variables

- `ETH_PRIVATE_KEY`: Your Ethereum private key (used for signing)
- `VPS_IP`: Your VPS public IP address (just the IP, no protocol or port)
- Use **Hoodi RPC URLs**:

```
https://ethereum-hoodi-rpc.publicnode.com
```

Backup URL can be the same or a different one if you have it.

---

### Create the SystemD service file

Run this command in terminal **after replacing** `PV_KEY` with your private key, and `VPS_IP` with your VPS IP:

```bash
sudo tee /etc/systemd/system/drosera.service > /dev/null <<EOF
[Unit]
Description=drosera node service
After=network-online.target

[Service]
User=$USER
Restart=always
RestartSec=15
LimitNOFILE=65535
ExecStart=$(which drosera-operator) node --db-file-path $HOME/.drosera.db --network-p2p-port 31313 --server-port 31314 \
    --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
    --eth-backup-rpc-url https://rpc.hoodi.ethpandaops.io \
    --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D \
    --eth-private-key PV_KEY \
    --listen-address 0.0.0.0 \
    --network-external-p2p-address VPS_IP \
    --disable-dnr-confirmation true \
    --eth-chain-id 560048

[Install]
WantedBy=multi-user.target
EOF
```

---

### Reload systemd and start the service

```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera
```

---

### Check & Control Node

```bash
# Follow logs live
journalctl -u drosera.service -f

# Check node service status
sudo systemctl status drosera

# Stop the node service
sudo systemctl stop drosera

# Restart the node service
sudo systemctl restart drosera

```
---

## B. Register your Operator ([FAILED to register ? click here](#-problem-register-operator-transaction-fails))

```bash
drosera-operator register \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
```

## C. Opt-in your trap config ([use dashboard instead click here](#configure-through-dashboard-instead))

```bash
drosera-operator optin \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key your_eth_private_key_here \
  --trap-config-address your_trap_address_here
```

---

## D. Firewall Configuration

```bash
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
sudo ufw enable
```
# 🟩🟩EVERYTHING IS PERFECT! WELCOME ON HOODI !🟩🟩



HELLO WORLD TRAP (light green)
![Drosera Hoodie One](Asset/hoodie%20one.png)
---
[DISCORD TRAP (dark green) (cadet role, click here)](#get-cadet-role-discord)
![Cadet Dark Green](Asset/cadet%20dark%20green.png)
---

---
# 🟥🟥 I DONE EVERYTHING MY NODE STILL RED :( 🟥🟥

##  How to Fix common IP/firewall Issues Red Node

---

### 🔧 For VPS:
- ✅ **Ensure external firewall is open**  
  *(e.g., cloud provider security group, firewalls in Vultr, AWS, Oracle, etc.)*
- ✅ **Ensure internal firewall is open**

```bash
sudo ufw allow <your_node_port>
sudo ufw reload
```

---

### 🖥️ For Your Own PC (Self-hosting Node Red):

- 🔁 **Ensure you have a static public IP**  
  ❗ Without it, DNAT and port forwarding may fail!

---
#### Options to Get Public Access:

1. 📞 **Call your ISP and subscribe to a static IP plan**  
   - Cost: ~$200/year (depends on country)
   - Pros: Direct access, no relays or VPNs
   - Cons: Expensive, may require router config

2. 🌐 **Rent a low-spec VPS + Use Port Forwarding**  
   - Cost: ~$20/year for a minimal VPS (only networking usage)
   - Combine with:
     - 🧰 **WireGuard VPN on VPS**
     - 🔧 **WireGuard VPN on your PC**
     - 🧱 **Port forwarding with UFW or iptables**
---


## 🛡️ Step 1: VPS Setup (Ubuntu 22.04)

**1. Configure WireGuard on VPS**  
```bash
sudo apt update && sudo apt install -y wireguard resolvconf
sudo su
umask 077
wg genkey > /etc/wireguard/privatekey
wg pubkey < /etc/wireguard/privatekey > /etc/wireguard/publickey
cat /etc/wireguard/privatekey  # <VPS_PRIVATE_KEY>
cat /etc/wireguard/publickey   # <VPS_PUBLIC_KEY>
```

**2. Create VPS WireGuard Config**  
```bash
nano /etc/wireguard/wg0.conf
```
```ini
[Interface]
PrivateKey = <VPS_PRIVATE_KEY>
Address = 10.8.0.1/24
ListenPort = 51820
MTU = 1280

# Enable IP forwarding
PostUp = sysctl -w net.ipv4.ip_forward=1

# Allow SSH always (optional but recommended)
PostUp = iptables -A INPUT -p tcp --dport 22 -j ACCEPT

# NAT traffic from VPN clients to Internet
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE

# Allow forwarding between VPN and Internet
PostUp = iptables -A FORWARD -i wg0 -o eth0 -j ACCEPT
PostUp = iptables -A FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT

# DNAT TCP ports to 10.8.0.2 (Linux client)
PostUp = iptables -t nat -A PREROUTING -i eth0 -p tcp -m multiport --dports 31313,31314 -j DNAT --to-destination 10.8.0.2

# DNAT UDP ports to 10.8.0.2 (Linux client)
PostUp = iptables -t nat -A PREROUTING -i eth0 -p udp -m multiport --dports 31313,31314 -j DNAT --to-destination 10.8.0.2

# SNAT replies from 10.8.0.2 to appear from VPS public IP
PostUp = iptables -t nat -A POSTROUTING -s 10.8.0.2 -j SNAT --to-source <VPS_PUBLIC_IP>

# --- Cleanup rules ---
PostDown = iptables -D INPUT -p tcp --dport 22 -j ACCEPT
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.0/24 -o eth0 -j MASQUERADE
PostDown = iptables -D FORWARD -i wg0 -o eth0 -j ACCEPT
PostDown = iptables -D FORWARD -i eth0 -o wg0 -m state --state RELATED,ESTABLISHED -j ACCEPT
PostDown = iptables -t nat -D PREROUTING -i eth0 -p tcp -m multiport --dports 31313,31314 -j DNAT --to-destination 10.8.0.2
PostDown = iptables -t nat -D PREROUTING -i eth0 -p udp -m multiport --dports 31313,31314 -j DNAT --to-destination 10.8.0.2
PostDown = iptables -t nat -D POSTROUTING -s 10.8.0.2 -j SNAT --to-source <VPS_PUBLIC_IP>

[Peer]
PublicKey = <WSL_PUBLIC_KEY> or <LINUX_PUBLIC_KEY> # GET AT STEP 2 WINDOW/LINUX SETUP
AllowedIPs = 10.8.0.2/32

```
Add accordingly input before save `<VPS_PRIVATE_KEY>`, `<VPS_PUBLIC_IP>`,`<WSL_PUBLIC_KEY> or <LINUX_PUBLIC_KEY>`

**3. Enable and Start WireGuard**  
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```
**4. VPS Firewall Fix:**
   ```bash
   sudo ufw allow 51820/udp
   sudo ufw allow 22/tcp
   sudo ufw allow 31313/tcp
   sudo ufw allow 31314/tcp
   ```

# Choose your local device setup method:

- **[Step 2: Windows WSL2 Setup](#-step-2-windows-wsl2-setup)**
- **[Step 2: Linux Setup](#-step-2-linux-setup)**
 

## 💻 Step 2: Windows WSL2 Setup

**1. Install WireGuard on Windows**
- Download from https://www.wireguard.com/install/

**2. Generate Keys in WSL**  
```bash
sudo apt update && sudo apt install -y wireguard-tools
umask 077
wg genkey | tee privatekey | wg pubkey > publickey
cat privatekey  # <WSL_PRIVATE_KEY>
cat publickey   # <WSL_PUBLIC_KEY>
```

**3. Create Windows Tunnel Config**
1. Open WireGuard Windows app
2. Click "Add Tunnel" > "Add empty tunnel"
3. Paste this config:
```ini
[Interface]
PrivateKey = <WSL_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 1.1.1.1, 8.8.8.8
MTU = 1280

[Peer]
PublicKey = <VPS_PUBLIC_KEY>
Endpoint = <VPS_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25

```
Add accordingly input before save `<WSL_PRIVATE_KEY>`, `<VPS_PUBLIC_KEY>`, `<VPS_PUBLIC_IP>`

**4. Activate Tunnel**
- Click "Activate" in WireGuard Windows app
- Verify connection:
```powershell
ping 10.8.0.1
```
**5. Local Firewall Fix:**
   ```bash
   sudo ufw allow 51820/udp
   sudo ufw allow 22/tcp
   sudo ufw allow 31313/tcp
   sudo ufw allow 31314/tcp
   ```
## 💻 Step 2: Linux Setup

**1. Install WireGuard**
```bash
sudo apt update && sudo apt install -y wireguard resolvconf
```

---

**2. Generate WireGuard Keys**
```bash
umask 077
wg genkey | sudo tee /etc/wireguard/privatekey | wg pubkey | sudo tee /etc/wireguard/publickey
```

**View your keys:**
```bash
sudo cat /etc/wireguard/privatekey  # <LINUX_PRIVATE_KEY>
sudo cat /etc/wireguard/publickey   # <LINUX_PUBLIC_KEY>
```

---

**3. Create WireGuard Config**
```bash
sudo nano /etc/wireguard/wg0.conf
```

Paste this (replace placeholders):
```ini
[Interface]
PrivateKey = <LINUX_PRIVATE_KEY>
Address = 10.8.0.2/24
DNS = 1.1.1.1, 8.8.8.8
MTU = 1280

[Peer]
PublicKey = <VPS_PUBLIC_KEY>
Endpoint = <VPS_PUBLIC_IP>:51820
AllowedIPs = 0.0.0.0/0
PersistentKeepalive = 25
```
Add accordingly input before save `<LINUX_PRIVATE_KEY>`, `<VPS_PUBLIC_KEY>`, `<VPS_PUBLIC_IP>`


**4. Start WireGuard Tunnel**
```bash
sudo systemctl enable wg-quick@wg0
sudo systemctl start wg-quick@wg0
```
**5. Local Firewall Fix:**
   ```bash
   sudo ufw allow 51820/udp
   sudo ufw allow 22/tcp
   sudo ufw allow 31313/tcp
   sudo ufw allow 31314/tcp
   ```

## 🐳 Step 3: Docker in WSL2/Linux

**1. Update docker-compose.yaml**  
```bash
nano docker-compose.yaml
```
```yaml
version: '3.8'

services:
  drosera-operator:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-operator
    ports:
      - "31313:31313"   # P2P
      - "31314:31314"   # HTTP
    environment:
      - DRO__DB_FILE_PATH=/data/drosera.db
      - DRO__DROSERA_ADDRESS=0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D
      - DRO__LISTEN_ADDRESS=0.0.0.0
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__PRIVATE_KEY=${ETH_PRIVATE_KEY}
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${VPS_IP}
      - DRO__SERVER__PORT=31314
      - RUST_LOG=info,drosera_operator=debug
      - DRO__ETH__RPC_TIMEOUT=30s
      - DRO__ETH__RETRY_COUNT=5
      # Optional: increase internal peer retry behavior (if supported by app)
      # - DRO__NETWORK__RETRY_INTERVAL=10s
      # - DRO__NETWORK__RETRY_COUNT=5
    volumes:
      - drosera_data:/data
    restart: always   # Ensures automatic container recovery
    logging:
      driver: "json-file"
      options:
        max-size: "10m"
        max-file: "5"
    healthcheck:   # Add Docker-native watchdog
      test: ["CMD", "curl", "-f", "http://localhost:31314/health"]
      interval: 60s
      timeout: 10s
      retries: 3
      start_period: 30s
    command: node

volumes:
  drosera_data:


```

**2. Update .env file**  
```bash
nano .env
```
```
ETH_PRIVATE_KEY=your_private_key_here
VPS_IP=your_vps_ip_here
```

**3. Start the Container**  
```bash
docker compose up -d
docker compose logs -f
```

---

✅ **Tip:** Always restart UFW or reload iptables rules after editing  
✅ **Tip:** You can also monitor traffic with `tcpdump` or `nload` for debugging

## ❗❗❗ Problem: Register Operator Transaction Fails❗❗❗

When attempting to register the operator, the following error appears:

```
2025-07-09T11:32:03.370017Z  INFO drosera_operator::register: Registering operator's BLS public key into registry.
Error: The Register transaction execution failed. Execution reverted. Reason: FunctionDoesNotExist
```

Even after running `droseraup`, the error still persists.

---
## A. Try these solution :

Choose Update Operator cli or Clean Reinstall Dependencies:

- [Solution 1: Update Operator CLI (click here)](#-solution-1-update-operator-cli)
- [Solution 2: Clean Reinstall Dependencies (click here)](#-solution-clean-reinstall-dependencies)
- [Solution 3: Use Docker for Registration (click here)](#-solution-3-use-docker-for-registration)
  
## ✅ Solution 1: Update Operator cli

You can manually install a stable version of the Drosera Operator CLI.

### 🔽 1. Download v1.20.0 Release

```bash
cd ~
curl -LO https://github.com/drosera-network/releases/releases/download/v1.20.0/drosera-operator-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
```

> 📌 You can verify or get other versions here:  
> https://github.com/drosera-network/releases/releases

---

### 📦 2. Extract the Archive

```bash
tar -xvf drosera-operator-v1.20.0-x86_64-unknown-linux-gnu.tar.gz
```

---

### ✅ 3. Test the CLI (Optional)

```bash
./drosera-operator --version
```

You should see something like:
```
drosera-operator 1.20.0
```

---

### 🚀 4. Install Globally

```bash
sudo cp drosera-operator /usr/bin
```

Now you can run it globally using:

```bash
drosera-operator register
```

---

This ensures you are using a known working version while bypassing any issues with `droseraup`.

## ✅ Solution 2: Clean Reinstall Dependencies

To resolve this, **completely remove all existing installations** and reinstall the required components from scratch.

### 1. Remove Old Installations

```bash
rm -rf ~/.drosera ~/.foundry ~/.bun
sed -i '/drosera\|foundry\|bun/d' ~/.bashrc ~/.profile
```

### 2. Reinstall Drosera CLI

```bash
curl -L https://app.drosera.io/install | bash
echo 'export PATH="$HOME/.drosera/bin:$PATH"' >> ~/.bashrc
```

### 3. Reinstall Foundry

```bash
curl -L https://foundry.paradigm.xyz/ | bash
echo 'export PATH="$HOME/.foundry/bin:$PATH"' >> ~/.bashrc
```

### 4. Reinstall Bun

```bash
curl -fsSL https://bun.sh/install | bash
echo 'export PATH="$HOME/.bun/bin:$PATH"' >> ~/.bashrc
```

### 5. Apply Environment Changes

```bash
source ~/.bashrc
exec bash
```

After this, re-run the operator setup . The error should be resolved if the environment is now correctly configured.
```bash
drosera-operator register
```
## ✅ Solution 3: Use Docker for Registration

If you're using Docker for the operator, you **don't need to install the CLI locally**. You can run the registration command directly inside Docker:

```bash
docker run -it --rm ghcr.io/drosera-network/drosera-operator:latest register \
  --eth-chain-id 560048 \
  --eth-rpc-url https://0xrpc.io/hoodi \
  --drosera-address 0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D \
  --eth-private-key YOUR_WALLET/OPERATOR_KEY
```

🛡️ This is a clean and containerized way to run the `register` command without polluting your host machine with dependencies.


# 🧑‍💻🧑‍💻Drosera Network Multi-Operator Setup (Hoodi Network)🧑‍💻🧑‍💻

## 1. Configure Trap
```bash
nano ~/my-drosera-trap/drosera.toml
```
```toml
private_trap = true
whitelist = ["0xOperator1Address", "0xOperator2Address"]

#Add operator 1 and operator 2 address accordingly
```

Apply changes:
```bash
DROSERA_PRIVATE_KEY=your_eth_private_key_here drosera apply --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

## 2. Register Operators
```bash
# Operator 1
drosera-operator register \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key YOUR_OPERATOR1_KEY

# Operator 2  
drosera-operator register \
  --eth-rpc-url https://rpc.hoodi.ethpandaops.io \
  --eth-private-key YOUR_OPERATOR2_KEY
```

## 3. Docker Setup
```bash
nano ~/Drosera-Network/docker-compose.yaml
```
`docker-compose.yaml`:
```yaml
version: '3.8'
services:
  operator1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    network_mode: host
    command: ["node"]
    environment:
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://ethereum-hoodi-rpc.publicnode.com
      - DRO__ETH__BACKUP_RPC_URL=https://rpc.hoodi.ethpandaops.io   # add backup RPC
      - DRO__ETH__PRIVATE_KEY=${OP1_KEY}
      - DRO__ETH__RPC_TIMEOUT=30s           # increase RPC timeout
      - DRO__ETH__RETRY_COUNT=5             # retry RPC calls on failure
      - DRO__NETWORK__P2P_PORT=31313
      - DRO__SERVER__PORT=31314
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${SERVER_IP}
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__LOG__LEVEL=debug
    volumes:
      - op1_data:/data
    restart: always

  operator2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    network_mode: host
    command: ["node"]
    environment:
      - DRO__ETH__CHAIN_ID=560048
      - DRO__ETH__RPC_URL=https://rpc.hoodi.ethpandaops.io
      - DRO__ETH__BACKUP_RPC_URL=https://ethereum-hoodi-rpc.publicnode.com  # add backup RPC
      - DRO__ETH__PRIVATE_KEY=${OP2_KEY}
      - DRO__ETH__RPC_TIMEOUT=30s           # increase RPC timeout
      - DRO__ETH__RETRY_COUNT=5             # retry RPC calls on failure
      - DRO__NETWORK__P2P_PORT=31315
      - DRO__SERVER__PORT=31316
      - DRO__NETWORK__EXTERNAL_P2P_ADDRESS=${SERVER_IP}
      - DRO__DISABLE_DNR_CONFIRMATION=true
      - DRO__LOG__LEVEL=debug
    volumes:
      - op2_data:/data
    restart: always

volumes:
  op1_data:
  op2_data:
```
```bash
nano ~/Drosera-Network/.env
```
`.env file`:
```env
SERVER_IP=your.server.ip
OP1_KEY=operator1_private_key
OP2_KEY=operator2_private_key
```
## 4. Firewall Configuration for additional ports

```bash
sudo ufw allow 31313/tcp
sudo ufw allow 31314/tcp
sudo ufw allow 31315/tcp
sudo ufw allow 31316/tcp
sudo ufw enable
```
## 5. Launch & Verify
```bash
docker compose up -d
docker logs operator1 --tail 50
docker logs operator2 --tail 50
```

## 6. Opt-In Operators 1 & 2 ([use dashboard instead click here](#configure-through-dashboard-instead))
```bash
drosera-operator optin \
  --eth-rpc-url https://ethereum-hoodi-rpc.publicnode.com \
  --eth-private-key YOUR_WALLET/OPERATOR_KEY \
  --trap-config-address YOUR_TRAP_ADDRESS
```
![Dual Operator](Asset/2%20operator.png)


# Configure through dashboard instead
Go to [https://app.drosera.io/](https://app.drosera.io/)\
![configure through dashboard](Asset/configure%20through%20dashboard.png)

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

<a name="get-cadet-role-discord"></a>
## GET CADET ROLE DISCORD (HOODI VERSION YEHOOOO) 🟥![cadet role](Asset/cadet%20role.png)

Assuming your Trap is deployed and your operator is running, let's set up a new Trap to submit your Discord username on-chain and unlock an exclusive Cadet role.

---

### 1. Create New Trap

#### 1- Move to your trap directory:

```bash
cd ~/my-drosera-trap
```

#### 2- Create a new Trap.sol file:

```bash
nano src/Trap.sol
```

#### 3- Paste the following contract code in it:

```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    // Updated response contract address
    address public constant RESPONSE_CONTRACT = 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608;
    string constant discordName = "YOURDISCORD"; // Replace with your Discord username

    function collect() external view returns (bytes memory) {
        bool active = IMockResponse(RESPONSE_CONTRACT).isActive();
        return abi.encode(active, discordName);
    }

    function shouldRespond(bytes[] calldata data) external pure returns (bool, bytes memory) {
        (bool active, string memory name) = abi.decode(data[0], (bool, string));
        if (!active || bytes(name).length == 0) {
            return (false, bytes(""));
        }

        return (true, abi.encode(name));
    }
}
```

Replace `YOURDISCORD` with your Discord username.  
To save: **Ctrl+X**, then **Y**, and **Enter**

---

### 2. Edit `drosera.toml` config

```bash
nano drosera.toml
```

Modify the values of the specified variables as follows:

```toml
ethereum_rpc = "https://ethereum-hoodi-rpc.publicnode.com"
drosera_rpc = "https://relay.hoodi.drosera.io"
eth_chain_id = 560048
drosera_address = "0x91cB447BaFc6e0EA0F4Fe056F5a9b1F14bb06e5D"

[traps]

[traps.mytrap]
path = "out/Trap.sol/Trap.json"
response_contract = "0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608"
response_function = "respondWithDiscordName(string)"
cooldown_period_blocks = 33
min_number_of_operators = 1
max_number_of_operators = 2
block_sample_size = 10
private_trap = true
whitelist = ["YOUR_OPERATOR_ADDRESS"]
address = "YOUR_TRAP_CONFIG_ADDRESS"
```

Your final `drosera.toml` file should match the example shown above.

---

### 3. Deploy Trap

#### 1- Compile your Trap's Contract:

```bash
forge build
```

If you get errors like: **command not found**, enter:

```bash
source /root/.bashrc
```

or reinstall dependencies from [here].

#### 2- Test the trap before deploying:

```bash
drosera dryrun
```

#### 3- Apply and Deploy the Trap:

```bash
DROSERA_PRIVATE_KEY=xxx drosera apply
```

Replace `xxx` with your EVM wallet private key (Ensure it's funded with Hoodi ETH).  
When prompted, write **ofc** and press Enter.

---

### 4. Verify Trap can respond

After the trap is deployed, check if the user has responded by calling the `isResponder` function on the response contract.

```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "isResponder(address)(bool)" OWNER_ADDRESS --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```

Replace `OWNER_ADDRESS` with your Trap's owner address is commonly whitelist/wallet/operator address NOT TRAP ADDRESS (your main address that deployed the Trap's contract).  
If you receive `true` as a response, it means you have successfully completed all the steps.

> It may take a few minutes to successfully receive `true` as a response.

---

### 5. Re-run Operator nodes

```bash
#Docker user
cd
cd ~/Drosera-Network
docker compose up -d

#SystemD user
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl restart drosera

```

### 6. View the List of submitted Discord Names

```bash
source /root/.bashrc
cast call 0x25E2CeF36020A736CF8a4D2cAdD2EBE3940F4608 "getDiscordNamesBatch(uint256,uint256)(string[])" 0 2000 --rpc-url https://ethereum-hoodi-rpc.publicnode.com
```
---
![discord username](Asset/discord%20username%20izmers.png)




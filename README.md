# Drosera Network Node Setup Guide

A comprehensive guide to setting up and running Drosera Network operators on testnet.

## System Requirements

- **CPU**: 2 cores minimum
- **RAM**: 4 GB minimum  
- **Storage**: 20 GB minimum
- **Network**: Stable internet connection with open ports
- **OS**: Ubuntu/Debian Linux

## Prerequisites

### 1. Server Setup

Update your system:
```bash
sudo apt-get update && sudo apt-get upgrade -y
sudo apt install curl ufw iptables build-essential git wget lz4 jq make gcc nano automake autoconf tmux htop nvme-cli libgbm1 pkg-config libssl-dev libleveldb-dev tar clang bsdmainutils ncdu unzip libleveldb-dev -y
```

### 2. Docker Installation

Remove old Docker packages:
```bash
for pkg in docker.io docker-doc docker-compose podman-docker containerd runc; do sudo apt-get remove $pkg; done
```

Install Docker:
```bash
sudo apt-get update
sudo apt-get install ca-certificates curl gnupg
sudo install -m 0755 -d /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
sudo chmod a+r /etc/apt/keyrings/docker.gpg

echo \
"deb [arch="$(dpkg --print-architecture)" signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
"$(. /etc/os-release && echo "$VERSION_CODENAME")" stable" | \
sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update -y && sudo apt upgrade -y
sudo apt-get install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin

# Test Docker installation
sudo docker run hello-world
```

### 3. Install Required Tools

#### Drosera CLI
```bash
curl -L https://app.drosera.io/install | bash
source /root/.bashrc
droseraup
```

#### Foundry CLI
```bash
curl -L https://foundry.paradigm.xyz | bash
source /root/.bashrc
foundryup
```

#### Bun Runtime
```bash
curl -fsSL https://bun.sh/install | bash
source /root/.bashrc
```

### 4. Configure Git
```bash
git config --global user.email "your_email@example.com"
git config --global user.name "Your Name"
```

## Trap Setup

### 1. Create Trap Directory
```bash
mkdir my-drosera-trap
cd my-drosera-trap
```

### 2. Initialize Trap Template
```bash
forge init -t drosera-network/trap-foundry-template
```

### 3. Install Dependencies
```bash
bun install
```

### 4. Compile Trap
```bash
forge build
```

### 5. Deploy Trap
```bash
DROSERA_PRIVATE_KEY=your_private_key drosera apply
```
- Replace `your_private_key` with your EVM wallet private key (ensure it's funded with Holesky ETH)
- When prompted, type `ofc` and press Enter

**For existing users**: If you've deployed a trap previously, add your trap address to `drosera.toml`:
```toml
address = "YOUR_TRAP_ADDRESS"
```

### 6. Configure Private Trap
Edit `drosera.toml`:
```bash
nano drosera.toml
```

Add at the bottom:
```toml
private_trap = true
whitelist = ["YOUR_OPERATOR_ADDRESS"]
```

Update configuration:
```bash
DROSERA_PRIVATE_KEY=your_private_key drosera apply
```

### 7. Fund Your Trap
1. Visit [https://app.drosera.io/](https://app.drosera.io/)
2. Connect your wallet
3. Click "Traps Owned" to find your trap
4. Click "Send Bloom Boost" and deposit Holesky ETH

## Operator Setup

### 1. Download and Install Operator
```bash
cd ~
# Check latest version at: https://github.com/drosera-network/releases/releases
curl -LO https://github.com/drosera-network/releases/releases/download/v1.17.2/drosera-operator-v1.17.2-x86_64-unknown-linux-gnu.tar.gz
tar -xvf drosera-operator-v1.17.2-x86_64-unknown-linux-gnu.tar.gz
sudo cp drosera-operator /usr/bin

# Verify installation
drosera-operator --version
```

### 2. Pull Docker Image
```bash
docker pull ghcr.io/drosera-network/drosera-operator:latest
```

### 3. Register Operator
```bash
drosera-operator register --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key YOUR_PRIVATE_KEY
```

### 4. Configure Firewall
```bash
# Enable firewall
sudo ufw allow ssh
sudo ufw allow 22
sudo ufw enable

# Allow Drosera ports
sudo ufw allow 31313/tcp
sudo ufw allow 31313/udp
sudo ufw allow 31314/tcp
sudo ufw allow 31314/udp
```

## Running the Node

### Method 1: Docker (Recommended)

1. **Clone setup repository**:
```bash
git clone https://github.com/0xmoei/Drosera-Network
cd Drosera-Network
```

2. **Configure environment**:
```bash
cp .env.example .env
nano .env
```

Set your values:
```env
ETH_PRIVATE_KEY=your_private_key
VPS_IP=your_server_ip
```

3. **Configure Docker Compose**:
```bash
nano docker-compose.yaml
```

Update RPC URLs to your preferred Ethereum Holesky RPC endpoints.

4. **Start the node**:
```bash
docker compose up -d
```

5. **Monitor logs**:
```bash
docker logs -f drosera-node
```

### Method 2: SystemD Service

Create service file:
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
--eth-rpc-url https://ethereum-holesky-rpc.publicnode.com \
--eth-backup-rpc-url https://1rpc.io/holesky \
--drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 \
--eth-private-key YOUR_PRIVATE_KEY \
--listen-address 0.0.0.0 \
--network-external-p2p-address YOUR_VPS_IP \
--disable-dnr-confirmation true

[Install]
WantedBy=multi-user.target
EOF
```

Start service:
```bash
sudo systemctl daemon-reload
sudo systemctl enable drosera
sudo systemctl start drosera
```

Monitor logs:
```bash
journalctl -u drosera.service -f
```

## Connect to Trap

1. **Visit Dashboard**: [https://app.drosera.io/](https://app.drosera.io/)
2. **Connect Wallet**: Use the same wallet as your operator
3. **Opt-in to Trap**: Click "Opt in" to connect your operator to your trap
4. **Wait for Green Blocks**: Your node should start producing green blocks in the dashboard

## Running Multiple Operators

### 1. Create Second Operator
- Create new EVM wallet and fund with Holesky ETH
- Register the second operator:
```bash
drosera-operator register --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key SECOND_PRIVATE_KEY
```

### 2. Update Trap Whitelist
Edit `drosera.toml`:
```toml
whitelist = ["FIRST_OPERATOR_ADDRESS", "SECOND_OPERATOR_ADDRESS"]
```

Apply changes:
```bash
DROSERA_PRIVATE_KEY=your_trap_private_key drosera apply
```

### 3. Configure Additional Ports
```bash
sudo ufw allow 31315/tcp
sudo ufw allow 31315/udp
sudo ufw allow 31316/tcp
sudo ufw allow 31316/udp
```

### 4. Update Docker Compose
Create dual-operator configuration:
```yaml
version: '3'
services:
  drosera1:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node1
    network_mode: host
    volumes:
      - drosera_data1:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31313 --server-port 31314 --eth-rpc-url RPC_URL_1 --eth-backup-rpc-url https://holesky.drpc.org --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 --eth-private-key ${ETH_PRIVATE_KEY} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always
  drosera2:
    image: ghcr.io/drosera-network/drosera-operator:latest
    container_name: drosera-node2
    network_mode: host
    volumes:
      - drosera_data2:/data
    command: node --db-file-path /data/drosera.db --network-p2p-port 31315 --server-port 31316 --eth-rpc-url RPC_URL_2 --eth-backup-rpc-url https://holesky.drpc.org --drosera-address 0xea08f7d533C2b9A62F40D5326214f39a8E3A32F8 --eth-private-key ${ETH_PRIVATE_KEY2} --listen-address 0.0.0.0 --network-external-p2p-address ${VPS_IP} --disable-dnr-confirmation true
    restart: always
volumes:
  drosera_data1:
  drosera_data2:
```

Update `.env`:
```env
ETH_PRIVATE_KEY=first_operator_private_key
ETH_PRIVATE_KEY2=second_operator_private_key
VPS_IP=your_server_ip
```

### 5. Opt-in Second Operator
Either:
- **Via Dashboard**: Login with second operator wallet and opt-in to trap
- **Via CLI**:
```bash
drosera-operator optin --eth-rpc-url https://ethereum-holesky-rpc.publicnode.com --eth-private-key SECOND_PRIVATE_KEY --trap-config-address YOUR_TRAP_ADDRESS
```

## Discord Username Trap (Bonus)

Deploy a special trap to submit your Discord username on-chain and earn a Cadet role:

### 1. Create Discord Trap Contract
```bash
cd my-drosera-trap
nano src/Trap.sol
```

Paste this content (replace `YOUR_DISCORD_USERNAME`):
```solidity
// SPDX-License-Identifier: MIT
pragma solidity ^0.8.20;

import {ITrap} from "drosera-contracts/interfaces/ITrap.sol";

interface IMockResponse {
    function isActive() external view returns (bool);
}

contract Trap is ITrap {
    address public constant RESPONSE_CONTRACT = 0x4608Afa7f277C8E0BE232232265850d1cDeB600E;
    string constant discordName = "YOUR_DISCORD_USERNAME";

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

### 2. Update Configuration
Edit `drosera.toml`:
```toml
path = "out/Trap.sol/Trap.json"
response_contract = "0x4608Afa7f277C8E0BE232232265850d1cDeB600E"
response_function = "respondWithDiscordName(string)"
```

### 3. Deploy Discord Trap
```bash
forge build
drosera dryrun
DROSERA_PRIVATE_KEY=your_private_key drosera apply
```

### 4. Verify Response
```bash
cast call 0x4608Afa7f277C8E0BE232232265850d1cDeB600E "isResponder(address)(bool)" YOUR_TRAP_OWNER_ADDRESS --rpc-url https://ethereum-holesky-rpc.publicnode.com
```

If you receive `true`, you've successfully completed the Discord username submission.

## Management Commands

### Restart Nodes
```bash
cd ~/Drosera-Network
docker compose restart

# Or restart individually
docker compose restart drosera1
docker compose restart drosera2
```

### Stop Nodes
```bash
docker compose down -v
```

### Check Logs
```bash
# Follow logs in real-time
docker logs -f drosera-node1

# Check last 50 lines
docker logs --tail 50 drosera-node1

# Check for errors
docker logs drosera-node1 | grep -i error
```

### Check Node Status
```bash
# Check if containers are running
docker ps

# Check port usage
netstat -tuln | grep 31313

# Check node version
drosera-operator --version
```

## Troubleshooting

### Common Issues

1. **Rate Limiting (429 Errors)**:
   - Use private RPC endpoints (Alchemy, QuickNode)
   - Switch to different public RPC providers
   - Reduce number of simultaneous requests

2. **InsufficientPeers Warning**:
   - Normal when starting up
   - Check firewall settings
   - Verify ports are open
   - Wait for peer discovery

3. **Red Blocks in Dashboard**:
   - Ensure trap is funded (Bloom Boost)
   - Verify operator is opted-in to trap
   - Check node logs for errors
   - Confirm trap configuration is correct

4. **Connection Timeouts**:
   - Check network connectivity
   - Verify VPS provider allows P2P traffic
   - Try restarting nodes

### Getting Help

- **Drosera Documentation**: [https://docs.drosera.io](https://docs.drosera.io)
- **Discord Community**: Check Drosera's official Discord
- **GitHub Issues**: [https://github.com/drosera-network](https://github.com/drosera-network)

## RPC Endpoints

### Recommended Holesky RPC Providers
- **Public**: 
  - `https://ethereum-holesky-rpc.publicnode.com`
  - `https://holesky.drpc.org`
  - `https://1rpc.io/holesky`
  - `https://rpc.ankr.com/eth_holesky`

- **Private** (Recommended for production):
  - [Alchemy](https://dashboard.alchemy.com/)
  - [QuickNode](https://dashboard.quicknode.com/)
  - [Infura](https://infura.io/)

## Security Notes

- **Never share your private keys**
- **Use environment variables for sensitive data**
- **Keep your system updated**
- **Monitor your node regularly**
- **Use strong passwords and SSH keys**
- **Enable firewall with minimal required ports**

---

*This guide is based on the official Drosera Network documentation and community contributions. Always refer to the latest official documentation for the most up-to-date information.*

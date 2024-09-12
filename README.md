# Story

## Server Setup

1. **Log in to your server**:
   Open your terminal and connect via SSH:

   ```bash
   ssh user@your_server_ip
   ```

2. **Update and upgrade your server**:
   It’s always a good idea to make sure your system is up-to-date:

   ```bash
   sudo apt update && sudo apt upgrade -y
   ```

3. **Install necessary dependencies**:
   You’ll need to install various dependencies to set up the Story node:

   ```bash
   sudo apt install curl build-essential git wget jq -y
   ```

4. **Install Docker** (optional but recommended):
   Docker helps to manage containers for isolated environments.

   ```bash
   sudo apt install docker.io -y
   sudo systemctl start docker
   sudo systemctl enable docker
   ```

---

## Clone the Story Node Repository

1. **Clone the Story Foundation repository**:
   You’ll need the official codebase to set up your node.

   ```bash
   git clone https://github.com/storyfoundation/story-node.git
   cd story-node
   ```

2. **Checkout the latest stable release**:
   Make sure you're using the latest stable version of the Story node software.

   ```bash
   git checkout tags/v1.x.x
   ```

---

## Build and Configure the Node

1. **Build the node**:
   Use the following commands to build the Story node from the source.

   ```bash
   make install
   ```

2. **Initialize your node**:
   Initialize your node with the name `sixNoda` or a custom name.

   ```bash
   storyd init sixNoda --chain-id story-chain
   ```

3. **Download the genesis file**:
   The genesis file is essential for connecting your node to the network.

   ```bash
   curl -o $HOME/.story/config/genesis.json https://story.foundation/genesis.json
   ```

4. **Set peers and seeds**:
   Add persistent peers and seeds for better network connectivity.

   ```bash
   PEERS="peer_address@ip:port"
   SEEDS="seed_address@ip:port"
   sed -i.bak -e "s/^persistent_peers *=.*/persistent_peers = \"$PEERS\"/" $HOME/.story/config/config.toml
   sed -i.bak -e "s/^seeds *=.*/seeds = \"$SEEDS\"/" $HOME/.story/config/config.toml
   ```

---

## Start Your Validator Node

1. **Start the node service**:
   After setting everything up, you can now start your node.

   ```bash
   storyd start
   ```

   Check the logs to ensure that everything is running smoothly:

   ```bash
   journalctl -u storyd -f
   ```

2. **Create a systemd service** (optional):
   To run your node in the background and ensure it restarts on server reboot, create a systemd service.

   ```bash
   sudo tee /etc/systemd/system/storyd.service > /dev/null <<EOF
   [Unit]
   Description=Story Node
   After=network-online.target

   [Service]
   User=$USER
   ExecStart=$(which storyd) start
   Restart=on-failure
   LimitNOFILE=4096

   [Install]
   WantedBy=multi-user.target
   EOF
   ```

   Then enable and start the service:

   ```bash
   sudo systemctl enable storyd
   sudo systemctl start storyd
   ```

---

## Create and Submit Your Validator

1. **Create a wallet**:
   You’ll need a wallet address to manage your validator and funds.

   ```bash
   storyd keys add sixNodaWallet
   ```

2. **Fund your wallet**:
   Send some Story tokens to your wallet to cover the staking fees and validator setup.

3. **Submit your validator**:
   Once your node is running and your wallet is funded, create your validator:

   ```bash
   storyd tx staking create-validator \
     --amount=1000000ustory \
     --pubkey=$(storyd tendermint show-validator) \
     --moniker="sixNoda" \
     --chain-id=story-chain \
     --commission-rate="0.05" \
     --commission-max-rate="0.20" \
     --commission-max-change-rate="0.01" \
     --min-self-delegation="1" \
     --fees=5000ustory \
     --from=sixNodaWallet \
     --yes
   ```

---

## Monitor and Maintain Your Validator

1. **Check your node status**:

   ```bash
   storyd status
   ```

2. **Ensure uptime**:
   Regularly monitor the node’s health and performance. Use services like Grafana or Prometheus for advanced monitoring.

3. **Handling slashing events**:
   Make sure to avoid downtime and double-signing blocks to prevent slashing.

---

# Step-by-Step Guide: Installing and Running POP Node in a tmux Session

# System Requirements
1. Linux
2. Minimum 4GB RAM (configurable), more the better for higher rewards
3. At least 100GB free disk space (configurable). 200-500GB is a sweet spot
4. Internet connectivity available 24/7

## Step 1: Install tmux
Before starting the node setup, install tmux to ensure the process runs even after SSH disconnection.

## For Ubuntu/Debian:
```console
sudo apt update && sudo apt install -y tmux
```
## For CentOS/RHEL:
```console
sudo yum install -y tmux
```
## For macOS (using Homebrew):
```console
brew install tmux
```

## Step 2: Start a New tmux Session
Create a new tmux session named pop-node to run the node inside it.
```console
tmux new-session -s pop-node
```
## Step 3: Download and Set Up the POP Binary
```console
# Download the compiled pop binary
curl -L -o pop "https://permissionless-labs.us10.list-manage.com/track/click?u=43df23ffe2d20a869f8209457&id=c334474a55&e=18706de02f"

# Assign executable permission to the pop binary
chmod +x pop

# Create a folder to be used for the download cache
mkdir download_cache
```
## Step 4: Run the POP Node Inside tmux
```console
./pop
```
## Optional: Use Referral Code

If you'd like to use a referral, here is one you can use:
```console
referral=62a43039f81528cc
```
 # Running POP Node as a Systemd Service

If you prefer running the node as a background service managed by systemd, follow these steps:

## Create a Service User and Required Directories
```console
sudo useradd -r -m -s /sbin/nologin pop-svc-user -d /home/pop-svc-user 2>/dev/null || true
sudo mkdir -p /opt/pop /var/lib/pop /var/cache/pop/download_cache
```
## Move the POP Binary and Set Permissions
```console
sudo mv -f ~/pop /opt/pop/
sudo chmod +x /opt/pop/pop 2>/dev/null || true
```
##  Handle Existing node_info.json
```console
sudo mv -f ~/node_info.json /var/lib/pop/ 2>/dev/null || true
```
## Set Ownership
```console
sudo chown -R pop-svc-user:pop-svc-user /var/lib/pop
sudo chown -R pop-svc-user:pop-svc-user /var/cache/pop
sudo chown -R pop-svc-user:pop-svc-user /opt/pop
```
 ## Create a Systemd Service File
```console
 sudo tee /etc/systemd/system/pop.service << 'EOF'
[Unit]
Description=Pipe POP Node Service
After=network.target
Wants=network-online.target

[Service]
User=pop-svc-user
Group=pop-svc-user
ExecStart=/opt/pop/pop \
    --ram=12 \
    --pubKey 7ugorwsqWvT6fwHoZhejAUPGWxK4fJtE9W8f8vebVh2S \
    --max-disk 175 \
    --cache-dir /var/cache/pop/download_cache \
    --no-prompt
Restart=always
RestartSec=5
LimitNOFILE=65536
LimitNPROC=4096
StandardOutput=journal
StandardError=journal
SyslogIdentifier=pop-node
WorkingDirectory=/var/lib/pop

[Install]
WantedBy=multi-user.target
EOF
```
## Replace the pubkey with your SOL wallet address.
Modify **pop.service**: Customize the parameters (such as --ram, --max-disk, --pubKey) based on your preferences.

## Set Config File Symlink and Create a POP Alias
```console
ln -sf /var/lib/pop/node_info.json ~/node_info.json
grep -q "alias pop='cd /var/lib/pop && /opt/pop/pop'" ~/.bashrc || echo "alias pop='cd /var/lib/pop && /opt/pop/pop'" >> ~/.bashrc && source ~/.bashrc
```
## Set Up Alias for Easy Access
```console
echo "alias pop='/opt/pop/pop'" >> ~/.bashrc
source ~/.bashrc
```
## Enable and Start the Service
```console
sudo systemctl daemon-reload
sudo systemd-analyze verify pop.service && sudo systemctl enable pop.service && sudo systemctl start pop.service
```
## Verify the Service Status
```console
sudo systemctl status pop
```
# Monitoring and Using POP Commands

## View Metrics
```console
./pop --status
```
## Check Points
```cosole
./pop --points-route
```
## Generate a Referral Code
```console
./pop --gen-referral-route
```
## Sign Up Using a Referral Code
```console
./pop --signup-by-referral-route <CODE>
```
# Backup Your Node

To prevent data loss, back up the node_info.json file and store it offline.

## If Running as a Service:
```console
cp /var/lib/pop/node_info.json ~/node_info.backup2-4-25
```
## If Running Quickstart:
```console
cp ~/node_info.json ~/node_info.backup2-4-25
```
## You can download the backup file off your machine using SCP or other methods.

# Important Notes:
1. ## Detach from the tmux Session

To keep the process running while exiting the SSH session, detach from tmux using:

 Ctrl + B, then press D

To reattach to the session later, use:
```console
tmux attach-session -t pop-node
```
2. ## Modify pop.service: Customize the parameters (such as --ram, --max-disk, --pubKey) based on your preferences.

3. Running POP Commands:

Change to the working directory first: cd /var/lib/pop && /opt/pop/pop [command]

OR use the provided alias: pop [command] (recommended).

4. ## Reattaching to tmux: If you used tmux, reattach anytime with:
```console
tmux attach-session -t pop-node
```
## Refer to pipe docs for more info:
https://docs.pipe.network/devnet-2

## Update 
## Download the compiled pop binary
```console
curl -L -o pop "https://dl.pipecdn.app/v0.2.4/pop"
```
## Assign executable permission to the pop binary
```console
chmod +x pop
```
# Troubleshooting Common Issues

## 1️⃣ "No such file or directory" when running pop

If you see this error:

-bash: ./pop: No such file or directory

Try running:
```console
/opt/pop/pop --status
```
If that works, set up the alias as mentioned above.

## 2️⃣ Checking if the service is running

To confirm the node is active:
```console
sudo systemctl status pop.service
```
## 3️⃣ Verifying if POP is listening on the correct port
```console
ss -tulnp | grep 8003
```
Expected output:

tcp   LISTEN 0  128  0.0.0.0:8003  0.0.0.0:*  users:("pop",pid=XXXX,fd=XX)

## 4️⃣ Enabling Port Forwarding for Remote Access

If external access is required, open port 8003:

For UFW (Ubuntu/Debian):
```console
sudo ufw allow 8003/tcp
```
For Firewalld (CentOS/RHEL):
```console
sudo firewall-cmd --add-port=8003/tcp --permanent
sudo firewall-cmd --reload
```
If behind a router, forward port 8003 (TCP) to your machine.

## 5️⃣ Testing External Access

From another machine:
```console
telnet <your-server-ip> 8003
```
If you get:

Connected to <your-server-ip>.
HTTP/1.1 408 Request Timeout

This means your node is reachable. 🎉

## 6️⃣ Restarting the Service

If needed, restart the service:
```console
sudo systemctl restart pop.service
```
## 7️⃣ Checking Logs for Errors

journalctl -u pop.service -f

# Your node should now be up and running! 🚀





















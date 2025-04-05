# 🚀 Deploy a WireGuard VPN on AWS EC2 via CloudFormation

This guide helps you deploy a WireGuard VPN server using a reusable AWS CloudFormation template.
You can choose between two CPU architectures: x86_64 and arm64, with SSH password authentication and optional IPv4/IPv6 access restrictions.

🔗 Official AWS EC2 architecture compatibility guide:
👉 https://aws.amazon.com/ec2/instance-types/

## ⚙️ Requirements
- ✅ An AWS account
- ✅ WireGuard client installed
- ✅ Manually generate Wireguard public and private key (macOs/Linux only)
- ⚠️ (Optional) Public IPv4/IPv6 address to restrict access to your IP
- ⚙️ (Optional) AWS CLI installed and configured (you can also use the AWS Console)

## 🔧 Setup & Installation

## Create a new AWS Free Tier account
https://aws.amazon.com/free

### Install wireguard client (Support Windows, Mac, Ubuntu, Android, iOS, Debian, etc..)
👉 Official installation guide:
https://www.wireguard.com/install/

### 🔐 Generate WireGuard Public/Private Key
**macOS/Linux**

    📁 Create the .wireguard directory
    ```bash
    mkdir -p ~/.wireguard
    chmod 700 ~/.wireguard
    ```

    🔑 Generate private and public keys
    ```bash
    wg genkey | tee ~/.wireguard/privatekey > ~/.wireguard/publickey
    ```

    🔒 Secure the private key
    ```bash
    chmod 600 ~/.wireguard/privatekey
    ```

    🔑 What you get
    ``~/.wireguard/privatekey`` Your private key  → Keep this safe!
    ``~/.wireguard/publickey`` → Share this with the EC2 server stack (as parameter)

**Windows**

    If you're using the official WireGuard Windows app, keys are typically generated and managed within the app GUI.

## 🔌 Connecting with WireGuard

1. 🎯 **Prepare your Client Keys**
    If you're on macOS or Linux, and you followed the key generation steps earlier:

    Your keys should be in the ~/.wireguard folder:
    ```bash
    ~/.wireguard/privatekey   # 🔐 Keep this private
    ~/.wireguard/publickey    # 📤 Share this with the VPN server
    ```
    💡 Windows Users: When creating a new tunnel in the WireGuard app, the client keys are auto-generated and stored inside the app. You can copy the public key from the interface.

2. 🚀 **Launch the EC2 VPN Server**
    Deploy the CloudFormation stack (use x86_64 t2.micro for free tier).

    Wait ~30 seconds for the instance to launch.

3. 📬 **Retrieve Server Info**
    After stack creation:
    - Use AWS CLI or Console to get the Public IPv4 / IPv6 address of your EC2 instance.
    - Connect via SSH using the password you set during stack creation:
    ```bash
      ssh vpnuser@<EC2_PUBLIC_IP>
    ```
    Retrieve the WireGuard server public key:

    ```bash
    sudo cat /etc/wireguard/publickey
    ```
    Copy and save this value — you'll need it on your client.

4. 🛠️ **Configure the Tunnel (Client Side)**
Open your WireGuard client app, and:

- Click "Add Tunnel" → "Add Empty Tunnel".
- Fill in the configuration like this:
    Fill in the configuration like this:

    ```ini
    [Interface]
    PrivateKey = <your-client-private-key>
    Address = 10.0.1.2/32
    DNS = 1.1.1.1
    MTU = 1280

    [Peer]
    PublicKey = <server-public-key>
    AllowedIPs = 0.0.0.0/0, ::/0
    Endpoint = <EC2_PUBLIC_IPV4_OR_IPV6>:51820
    PersistentKeepalive = 25

    ```

    ⚠️ Replace ``<your-client-private-key>`` and ``<server-public-key>``  and ``<EC2_PUBLIC_IPV4_OR_IPV6>`` with actual values.

5. ✅ **Connect**
    - Enable the tunnel in your WireGuard app.
    - On macOS, you can also manage the VPN via System Settings → VPN.

6. 🌍 **Test Your Connection**

    Visit:
    👉 https://whatismyipaddress.com
    If the displayed IP address matches your EC2 instance's public IP, you're securely connected through the VPN!

 ## AWS-CLI commands

 ### Create Stack
 👉 Check commands in either arm64/README.md or x86_64/README.md

 ### Printout Stack creation output (EC2 IPS, server keys)
 ```shell
 aws cloudformation describe-stacks \
  --stack-name <your-stack-name> \
  --query "Stacks[0].Outputs" \
  --output table \
  --region <your-region>
```
### Delete Stack
```shell
  aws cloudformation delete-stack \
  --region <your-region> \
  --stack-name <your-stack-name>
```
  If stuck in DELETE_FAILED, manually check dependencies (e.g., VPC IPv6 blocks) or use the console to troubleshoot.

# 🛠️ Troubleshooting Guide
## 🔌 No Internet via VPN?

### ⚙️(macOS/iOS only) Unchecked on-demand-option

On macOS or iOS, open the WireGuard client and go to your tunnel configuration.

Make sure the following options are unchecked:

- "Activate on demand (Wi-Fi)"

- "Activate on demand (Cellular / Internet)"

### 📡 Check IP Forwarding on the Server

```bash
sudo sysctl net.ipv4.ip_forward
sudo sysctl net.ipv6.conf.all.forwarding
```
Expected: both should return = 1

### 🔄  Ensure NAT (MASQUERADE) rule is set on server

```bash
sudo iptables -t nat -L POSTROUTING -n -v
```
You should see a rule like:
```shell
MASQUERADE  all  --  10.100.0.0/24
```

### 🔐Check EC2 Security Group Rules
Make sure your EC2 instance’s Security Group allows the necessary inbound traffic.

If you haven't specified any IP restriction (i.e., open to all), your Security Group should have 6 inbound rules:

|Protocol	|Port	|Source	|Purpose|
|----------|------|---------------|----------------|
|UDP|	51820	|0.0.0.0/0	|WireGuard (IPv4 access)|
|UDP|	51820|	::/0|	WireGuard (IPv6 access)|
|TCP|	22	|0.0.0.0/0|	SSH (IPv4 access)|
|TCP|	22|	::/0	|SSH (IPv6 access)|
|ICMP|	All	|0.0.0.0/0	|Ping (IPv4)|
|ICMPv6|	All|	::/0	|Ping (IPv6)|

🔐 Note: If you specified a specific IP (like 85.203.80.129/32) when creating the stack, replace 0.0.0.0/0 or ::/0 with that IP in the rules above.

**You can view and edit these rules in:**
- AWS Console → EC2 → Security Groups → Inbound Rules
- Or via AWS CLI using describe-security-groups

### 🚦 Ensure the WireGuard Interface is Up
```bash
  sudo wg show
  ip a | grep wg0
```
You should see interface details including your VPN peer.

### 🌐 Verify EC2 Has Internet Access
From your EC2 instance, run:
```bash
  curl https://ifconfig.me
  ping 1.1.1.1
```
✅ If successful, your server can reach the internet.

## 🪟 Windows WireGuard Setup & Troubleshooting
✅ Recommended Configuration (Client)
Use the following WireGuard configuration file as a base. Replace the placeholder values with your actual server details:
```ini 
[Interface]
PrivateKey = <client-private-key>
Address = 10.0.1.2/32
DNS = 1.1.1.1
MTU = 1420

[Peer]
PublicKey = <wg-server-public-key>
AllowedIPs = 0.0.0.0/1, 128.0.0.0/1
Endpoint = <ec2-instance-ip>:51820
PersistentKeepalive = 25
```
The AllowedIPs = 0.0.0.0/1, 128.0.0.0/1 trick routes all internet traffic through the tunnel while excluding the VPN server’s public IP, so the connection stays alive.

🧩 Important Notes for Windows Users
### Run WireGuard as Administrator
Right-click on the WireGuard shortcut and select "Run as administrator". This is required so the app can set routes and network permissions correctly.

### Uncheck "Block untunneled traffic"
In the WireGuard UI, make sure the option "Block untunneled traffic (kill-switch)" is unchecked unless you know what you're doing.

If enabled too early, it can prevent the VPN connection from being established.

### No Handshake? Check Firewall & Permissions
If wg shows latest handshake: never, the client can't reach the server. Check:

  - That you’re using the correct public key and IP.
  - Windows Firewall or other security software isn't blocking UDP 51820.
  - You’ve run the app with administrative privileges.

### Don’t worry about blank adapter IPs in the UI
The WireGuard interface in "Network Connections" may show empty fields for IPv4 configuration — this is normal. WireGuard configures routes internally, not through the Windows UI.
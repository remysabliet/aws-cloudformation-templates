# 🚀 Deploy the WireGuard EC2 Stack (via AWS CLI)
You can deploy the VPN stack using AWS CLI. This command launches a VPN server EC2 instance using a CloudFormation template.

🔐 Before running this command, make sure you have:

- Your WireGuard client public key
- A secure SSH password for logging into the EC2 instance
- (Optional) Your IPv4 and/or IPv6 public address if you want to restrict access to your device only

## Create the Stack

```bash
aws cloudformation create-stack \
  --region <your-server-region> \
  --stack-name vpn-ec2-arm64 \
  --template-body file://wireguard-ec2-vpn-arm64.yaml \
  --capabilities CAPABILITY_NAMED_IAM \
  --parameters \
    ParameterKey=Password,ParameterValue=<your-ssh-password> \
    ParameterKey=ClientPublicKey,ParameterValue=<your-wireguard-client-public-key>
```

Optional Parameters (to restrict IP access)
You can add one or both of these to limit VPN access to your own public IP(s):
```ssh
    ParameterKey=IPv4Address,ParameterValue=<your-client-ipv4>/32 \
    ParameterKey=IPv6Address,ParameterValue=<your-client-ipv6>/128
```

### Parameter Details
``--region <your-server-region>``: Mandatory

*Specifies the AWS region where the stack will be created. Choose a region where you want to leverage an IP from. For a list of available AWS regions and their codes, refer to the AWS Regional Services List.*

``--stack-name vpn-ec2-arm64``: Mandatory
Defines the name of the CloudFormation stack. Replace vpn-ec2-arm64 with your preferred stack name.

``--template-body`` file://wireguard-ec2-vpn-arm64.yaml: Mandatory
Points to the CloudFormation template file. Ensure the path is correct and the file exists.

``--capabilities CAPABILITY_NAMED_IAM``: Mandatory
Acknowledges that the stack might create IAM resources with custom names.

``--parameters``: Mandatory
Specifies the parameters required by the CloudFormation template:

 - ``ParameterKey=Password,ParameterValue=<your-ssh-password>``: Mandatory

  Sets the SSH password for the vpnuser account on the EC2 instance.

 - ``ParameterKey=ClientPublicKey,ParameterValue=<your-wireguard-client-public-key>`` : Mandatory

  Provides your WireGuard client public key for VPN authentication.

 - ``ParameterKey=IPv4Address,ParameterValue=<your-client-ipv4>/32``: Optional

  Specifies your public IPv4 address to restrict VPN access. Important: Append /32 to denote a single IP address. Example: 85.203.80.129/32.

 - ``ParameterKey=IPv6Address,ParameterValue=<your-client-ipv6>/128``: Optional

  Specifies your public IPv6 address to restrict VPN access. Important: Append /128 to denote a single IP address. Example: 2a01:e34:abcd:5678::1/128.

### Parameter Breakdown

|Parameter|Required|Description|
|----------|------|------------|
| Password | ✅ Yes | The password used for SSH login to the EC2 instance (vpnuser)|
| ClientPublicKey | ✅ Yes | Your WireGuard client public key used to connect to the VPN |
| IPv4Address|❌ No|Your public IPv4 (e.g. 85.203.80.129/32) to restrict access (use /32 CIDR)|
| IPv6Address|❌ No|Your public IPv6 (e.g. 2a01:e34:abcd:5678::1/128) to restrict access (use /128 CIDR)

⚠️ Important: When specifying your IP address:

- Always include the CIDR suffix:

  - /32 for IPv4 (single IP address)
  - /128 for IPv6 (single IP address)

  ✅ Correct:
    - 85.203.80.129/32
    - 2a01:e34:abcd:5678::1/128

  ❌ Incorrect (missing CIDR):
    - 85.203.80.129
    - 2a01:e34:abcd:5678::1

💡 Why IPv4 and IPv6 are optional:
If you don’t specify them, the stack will open the VPN and SSH ports to all IPs by default (0.0.0.0/0, ::/0).
If you provide them, only those IPs will be allowed to connect to the EC2 instance via SSH or WireGuard.


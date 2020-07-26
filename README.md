# cfn-cheapest-nat

[![build status][3]][4]

The never ending quest to find the cheapest AWS VPC NAT solution for personal projects.

|solution                           |network  |cost/GB|cost/hour**|cost/month**|
|-----------------------------------|---------|-------|-----------|------------|
|[NAT Gateway][1]                   |5-45 Gbps|  0.059|0.059      |42.48       |
|[NAT Instance (t3a.nano)][2]       |0-5  Gbps|0-0.114|0.0059     | 4.25       |
|[NAT Instance (t3a.nano) (spot)][2]|0-5  Gbps|0-0.114|0.0018*    | 1.30*      |

\* variable costs.

\*\* region ap-southeast-2.

## Deploy

```
STACK_NAME=examples-nat \
PRIVATE_ROUTE_TABLES=rtb-0eee90cf29e333813,rtb-0c1d060b614e74b88 \
PUBLIC_SUBNET=subnet-03ad595bb28ce7679 \
  ./bin/deploy
```

## Testing

I use the AWS System Manager Session Manager to SSH into an instance in a private subnet utilizing the NAT and run:

```
yum install python python-pip -y \
 && pip install --upgrade pip \
 && pip install speedtest-cli \
 && speedtest-cli
Retrieving speedtest.net configuration...
Testing from Amazon.com (54.206.26.162)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Telstra (Sydney) [1.01 km]: 1.82 ms
Testing download speed................................................................................
Download: 3283.42 Mbit/s
Testing upload speed................................................................................................
Upload: 2274.26 Mbit/s
```

[1]: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
[2]: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html
[3]: https://codebuild.us-east-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiWVdkQTF5ekRUSm1FNjgxT0RsL0ZBanFER1dSRG1kQWI0VUNLS2NlS0EwZ0pjdmN5a1RVSGI5K2p5Ty9vZFVZZ2gxck1GOWM4bHJ3WC9VVzJhZDVieE9vPSIsIml2UGFyYW1ldGVyU3BlYyI6IlhoM1dkMGw4M3VFNXlZWU4iLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master
[4]: https://console.aws.amazon.com/codesuite/codebuild/projects/examples-cheapest-nat/history?region=us-east-1

## Notes

[VPC NAT Instance Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)

```
cat /etc/sysctl.d/10-nat-settings.conf
```

```
#
# NAT AMI settings
#

net.ipv4.ip_forward = 1
net.ipv4.conf.eth0.send_redirects = 0
```

```
cat /usr/sbin/configure-pat.sh
```

```
#!/bin/bash
# Configure the instance to run as a Port Address Translator (PAT) to provide
# Internet connectivity to private instances.

function log { logger -s -t "vpc" -- $1; }

function die {
    [ -n "$1" ] && log "$1"
    log "Configuration of PAT failed!"
    exit 1
}

# Sanitize PATH
export PATH="/usr/sbin:/sbin:/usr/bin:/bin"

log "Determining the MAC address on eth0..."
ETH0_MAC=$(cat /sys/class/net/eth0/address) ||
    die "Unable to determine MAC address on eth0."
log "Found MAC ${ETH0_MAC} for eth0."

# This script is intended to run only on a NAT instance for a VPC
# Check if the instance is a VPC instance by trying to retrieve vpc id
VPC_ID_URI="http://169.254.169.254/latest/meta-data/network/interfaces/macs/${ETH0_MAC}/vpc-id"

VPC_ID=$(curl --retry 3 --silent --fail ${VPC_ID_URI})
if [ $? -ne 0 ]; then
   log "The script is not running on a VPC instance. PAT may masquerade traffic for Internet hosts!"
fi

log "Enabling PAT..."
sysctl -q -w net.ipv4.ip_forward=1 net.ipv4.conf.eth0.send_redirects=0 &&
(
    iptables -t nat -C POSTROUTING -o eth0 -j MASQUERADE 2> /dev/null ||
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE ) ||
die

sysctl net.ipv4.ip_forward net.ipv4.conf.eth0.send_redirects | log
iptables -n -t nat -L POSTROUTING | log

log "Configuration of PAT complete."
exit 0
```

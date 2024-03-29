# cfn-cheapest-nat

[![build status][3]][4]

Cheapest AWS VPC NAT solution for personal projects.

## Context

The current solutions *is*:

- EC2 running on Spot.
- Auto Healing 
  - automatically replaces the unhealthy instance.
  - re-attaches a persistent network interface to recover transport level details such as routes.

The solution is *not*:

- Highly Available
  - instance unavailability will cause NAT disruption.
- Fault Tolerant
  - the persistent network interface results in dependency on a single zone.

## Logical Diagram

![Logical Diagram](/docs/logical.drawio.svg)

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

## Costs

|solution                           |network  |cost/GB|cost/hour**|cost/month**|
|-----------------------------------|---------|-------|-----------|------------|
|[NAT Gateway][1]                   |5-45 Gbps|  0.059|0.059      |42.48       |
|[NAT Instance (t3a.nano)][2]       |0-5  Gbps|0-0.114|0.0059     | 4.25       |
|[NAT Instance (t3a.nano) (spot)][2]|0-5  Gbps|0-0.114|0.0018*    | 1.30*      |

\* variable costs.

\*\* region ap-southeast-2.

## AMI

### Documentation

- [VPC NAT Instance Documentation](https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html)

### Configuration

- /etc/sysctl.d/10-nat-settings.conf
- /usr/sbin/configure-pat.sh

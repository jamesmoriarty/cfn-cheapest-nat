# cfn-cheapest-nat

[![build status][3]][4]

The never ending quest to find the cheapest NAT solution for personal projects.

|solution                           |network  |cost/hour**|cost/month**|
|-----------------------------------|---------|-----------|------------|
|[NAT Gateway][1]                   |5-45 Gbps|0.045      |32.40       |
|[NAT Instance (t3a.nano)][2]       |0-5  Gbps|0.0059     | 4.25       |
|[NAT Instance (t3a.nano) (Spot)][2]|0-5  Gbps|0.0018*    | 1.30*      |

\* variable costs.

\*\* region ap-southeast-2.

## Deploy

```
STACK_NAME=examples-nat \
PRIVATE_ROUTE_TABLES=rtb-0eee90cf29e333813,rtb-0c1d060b614e74b88 \
PUBLIC_SUBNETS=subnet-03ad595bb28ce7679,subnet-09f9df1d1d8a2c2c9 \
  ./bin/deploy
```

## Testing

### speedtest-cli

```
./speedtest-cli
Retrieving speedtest.net configuration...
Testing from Amazon.com (13.211.165.98)...
Retrieving speedtest.net server list...
Selecting best server based on ping...
Hosted by Telstra (Sydney) [1.01 km]: 2.611 ms
Testing download speed................................................................................
Download: 2651.82 Mbit/s
Testing upload speed................................................................................................
Upload: 2638.67 Mbit/s
```

### Example VPC

```
aws cloudformation deploy \
  --no-fail-on-empty-changeset \
  --region ap-southeast-2 \
  --template-file cfn/vpc.yml \
  --stack-name examples-vpc \
  --parameter-overrides \
    EnvironmentName=Examples
```

[1]: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
[2]: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html
[3]: https://codebuild.us-east-1.amazonaws.com/badges?uuid=eyJlbmNyeXB0ZWREYXRhIjoiWVdkQTF5ekRUSm1FNjgxT0RsL0ZBanFER1dSRG1kQWI0VUNLS2NlS0EwZ0pjdmN5a1RVSGI5K2p5Ty9vZFVZZ2gxck1GOWM4bHJ3WC9VVzJhZDVieE9vPSIsIml2UGFyYW1ldGVyU3BlYyI6IlhoM1dkMGw4M3VFNXlZWU4iLCJtYXRlcmlhbFNldFNlcmlhbCI6MX0%3D&branch=master
[4]: https://console.aws.amazon.com/codesuite/codebuild/projects/examples-cheapest-nat/history?region=us-east-1


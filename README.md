# cfn-cheapest-nat

The never ending quest to find the cheapest NAT solution for personal projects.

|solution                           |network  |cost/hour|cost/month|
|-----------------------------------|---------|---------|----------|
|[NAT Gateway][1]                   |5-45 Gbps|0.45     |32.40     |
|[NAT Instance (t3a.nano)][2]       |0-5  Gbps|0.0059   | 4.25     |
|[NAT Instance (t3a.nano) (Spot)][2]|0-5  Gbps|0.0018*  | 1.30*    |

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

### Example VPC

```
aws cloudformation deploy \
  --no-fail-on-empty-changeset \
  --region ap-southeast-2 \
  --template-file cfn/vpc.yml \
  --stack-name examples-vpc \
  --parameter-overrides \
    EnvironmentName=examples
```

[1]: https://docs.aws.amazon.com/vpc/latest/userguide/vpc-nat-gateway.html
[2]: https://docs.aws.amazon.com/vpc/latest/userguide/VPC_NAT_Instance.html


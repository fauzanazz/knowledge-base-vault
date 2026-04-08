---
title: "Serverless Architecture"
category: backend-architecture
summary: "A cloud execution model where code runs in stateless, ephemeral functions managed entirely by the provider — abstracting away server provisioning, scaling, and infrastructure management."
sources:
  - web-research
updated: 2026-04-08T11:00:00.000Z
---

# Serverless Architecture

> A cloud execution model where code runs in stateless, ephemeral functions managed entirely by the provider — abstracting away server provisioning, scaling, and infrastructure management.

## Overview

Serverless — more precisely **Function as a Service (FaaS)** — lets developers deploy individual functions that execute on-demand in response to events. The cloud provider (AWS Lambda, Google Cloud Functions, Azure Functions) handles all infrastructure: provisioning, patching, scaling, and high availability. You pay only for execution time, not idle capacity.

"Serverless" is a misnomer — servers exist, but they're invisible to developers.

## Core Components

### Functions (FaaS)
Stateless, single-purpose code units triggered by events. Runtime limits typically 15 minutes (AWS Lambda). Memory: 128MB–10GB.

```python
# AWS Lambda handler
def handler(event, context):
    user_id = event['pathParameters']['userId']
    user = db.get_user(user_id)
    return {
        'statusCode': 200,
        'body': json.dumps(user)
    }
```

### Triggers / Event Sources
Functions respond to events from many sources:
- **HTTP**: API Gateway → Lambda
- **Storage**: S3 object created → Lambda (resize image, index content)
- **Queue**: SQS message → Lambda (process order)
- **Stream**: Kinesis/DynamoDB Streams → Lambda (real-time analytics)
- **Schedule**: CloudWatch Events/cron → Lambda (nightly report)
- **Database**: DynamoDB Streams, RDS Proxy triggers

### Backend as a Service (BaaS)
Managed services replace custom backend code: Auth0/Cognito (auth), Firebase/DynamoDB (database), S3 (storage), SES/SendGrid (email). Truly serverless apps combine FaaS + BaaS to minimize custom code.

## Cold Starts

The most significant operational challenge in serverless. When no warm container exists for a function, the provider must:
1. Allocate a container (~100–200ms)
2. Start the runtime (JVM: 1–3s, Node.js: 50–200ms, Python: 50–150ms)
3. Initialize the function code (varies)

**Mitigation strategies**:
- **Provisioned Concurrency** (AWS Lambda): Pre-warm N instances — eliminates cold starts at cost
- **Keep functions warm**: Scheduled ping every 5 minutes (hack, but effective)
- **Choose faster runtimes**: Node.js/Python cold start faster than Java/C#
- **Reduce package size**: Smaller deployment packages initialize faster
- **Use Snap Start** (AWS Lambda for Java): Snapshot post-init state

```
Cold start latency by runtime (approximate):
Node.js 18:  ~200ms
Python 3.11: ~300ms
Go:          ~150ms
Java 17:     ~1000–3000ms (without SnapStart)
Java 17 + SnapStart: ~200ms
```

## Event-Driven Patterns

### Fan-Out
One event triggers multiple functions in parallel:
```
S3 Upload
    ├──► Lambda: Generate thumbnail
    ├──► Lambda: Extract metadata
    ├──► Lambda: Run virus scan
    └──► Lambda: Notify user
```

### Saga / Orchestration
Step Functions (AWS) or Durable Functions (Azure) orchestrate multi-step workflows:
```
Order Placement Saga:
  Step 1: Reserve inventory (Lambda)
  Step 2: Charge payment (Lambda)
  Step 3: Send confirmation (Lambda)
  On failure: Compensation functions run in reverse
```

### Event Sourcing + Streams
Lambda processes Kinesis streams for real-time event processing. Netflix uses Lambda for processing viewing event streams, triggering recommendations updates without dedicated consumers.

## Real-World Adoption

**Coca-Cola**: Replaced vending machine backend with AWS Lambda — went from $13,000/month (EC2) to $4,500/month (Lambda).

**iRobot**: Processes millions of Roomba telemetry events/day via Lambda + Kinesis.

**Netflix**: Uses Lambda for encoding pipeline triggers, A/B test event processing, and operational tooling — not core streaming (which requires persistent compute).

**Airbnb**: Lambda for image processing, data transformation pipelines, and internal tooling.

## Serverless Framework Patterns

### AWS SAM (Serverless Application Model)
```yaml
Resources:
  OrderProcessor:
    Type: AWS::Serverless::Function
    Properties:
      Runtime: python3.11
      Handler: handler.process_order
      Events:
        SQSEvent:
          Type: SQS
          Properties:
            Queue: !GetAtt OrderQueue.Arn
            BatchSize: 10
      Environment:
        Variables:
          DB_URL: !Ref DatabaseUrl
```

### Terraform + Lambda
Infrastructure-as-code for complex multi-function systems with VPC, IAM, and scaling configurations.

## Cost Model

```
AWS Lambda pricing (us-east-1):
- Requests:   $0.20 per 1M requests
- Duration:   $0.0000166667 per GB-second

Example: 10M requests/month, avg 200ms, 512MB memory
= $0.20 * 10 (requests) + $0.0000166667 * 10M * 0.2s * 0.5GB
= $2.00 + $16.67 = ~$18.67/month

vs. EC2 t3.medium: ~$30/month (always on)
```
Break-even: Functions win for bursty, unpredictable, or low-traffic workloads.

## Trade-offs

| Pros | Cons |
|------|------|
| Zero infrastructure management | Cold starts affect latency-sensitive workloads |
| Auto-scales to zero (no idle cost) | Vendor lock-in (proprietary triggers, IAM, etc.) |
| Pay per execution, not per hour | Hard to test locally — different from production |
| Infinite horizontal scalability | 15-minute max execution limits |
| Rapid deployment of isolated features | Distributed tracing is complex |
| | Statelessness requires external state stores |

## Anti-Patterns

- **Monolithic Lambda**: One Lambda handles all routes — defeats the purpose; becomes hard to maintain
- **Lambda-to-Lambda synchronous chains**: Creates tight coupling and compounding latency
- **Storing state in Lambda memory**: State is lost on container recycling
- **Ignoring concurrency limits**: Default Lambda concurrency is 1,000/account — reserved concurrency matters
- **Not setting timeouts**: Runaway functions waste money and hit limits

## When to Use

✅ **Use when**: Event-driven processing (file uploads, webhooks, stream processing); unpredictable or bursty traffic; APIs with variable load; batch jobs; scheduled tasks; MVPs with unpredictable scale needs.

❌ **Avoid when**: Latency-sensitive APIs where cold starts are unacceptable; long-running processes (>15min); WebSocket servers requiring persistent connections; high-throughput steady traffic (containers cheaper).

---
*Related: [[Event-Driven Architecture]], [[API Gateway Pattern]], [[Backend for Frontend]], [[Circuit Breaker Pattern]]*

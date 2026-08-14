# ADR-002: .NET 10 and Serverless AWS Demo Architecture

- **Status:** Accepted
- **Decision date:** 2026-08-14
- **Decision authority:** Platform architecture and demonstration governance

## Context

The portfolio needs a reproducible zero-AWS-cost local experience and an optional hosted demonstration that proves the first walking skeleton without implying production readiness or deploying every planned context. AWS prices, Free Tier eligibility, service availability, and runtime support can change.

`Accepted` approves the runtime and hosting direction. Runtime code, infrastructure, provider configuration, repositories, and production certification remain absent and `Planned` or `Deferred` as their owning sources state.

## Decision drivers

- Demonstrate current .NET and AWS serverless architecture skills.
- Preserve domain and ownership semantics across local and hosted profiles.
- Avoid persistent-cost infrastructure and uncontrolled portfolio charges.
- Deploy only the smallest approved vertical slice.
- Prefer managed, usage-based services with explicit teardown.

## Decision

Use .NET 10 for future application/runtime implementation. Local Zero AWS Cost is the default development and demonstration profile. An optional, ephemeral, Free-Tier-aware AWS Demo uses managed, usage-based AWS services and initially deploys only the approved walking skeleton.

AWS currently documents the managed `.NET 10` Lambda runtime identifier `dotnet10` on Amazon Linux 2023 and a .NET 10 Lambda base image. Packaging remains an implementation choice and must be revalidated before deployment.

### Local Zero AWS Cost

The local profile:

- requires no AWS account or credentials and is the only profile that guarantees zero AWS cost;
- uses fictitious data and deterministic provider fakes;
- uses replaceable local persistence, event, queue, object, identity, scheduling, and provider adapters;
- preserves ownership, authorization, idempotency, retry, recovery, audit, retention, and verified-deletion semantics;
- does not require LocalStack.

### Ephemeral AWS Demo

The first hosted walking skeleton contains only Application Process, Customer & Identity, Credit Decisioning, the minimum public edge, authentication, asynchronous delivery, service-owned persistence, essential logs, and cost controls.

The approved service direction is:

- AWS Lambda with .NET 10;
- API Gateway HTTP API;
- Amazon Cognito for demo authentication;
- Amazon EventBridge for integration-event routing;
- one Amazon SQS queue and DLQ per deployed consumer;
- owner-scoped DynamoDB persistence;
- optional minimal Amazon S3 only for required protected objects;
- short-retention CloudWatch Logs;
- AWS Budgets and Cost Anomaly Detection controls, subject to current availability, billing latency, and pricing.

AWS-managed encryption is used where it satisfies the demo threat model and avoids separately billed key infrastructure.

## Scope

This ADR selects application runtime direction and the Local/AWS Demo execution profiles. Production, exact Region/account topology, IaC, packaging details, service sizing, and undeployed bounded contexts are outside scope.

## Consequences

### Positive

- Developers can demonstrate behavior without an AWS account or AWS charges.
- The hosted slice uses managed, usage-based services and limited operational surface.
- Ports and adapters preserve domain portability between profiles.

### Negative

- Local adapters cannot prove every managed-service behavior.
- Serverless cold starts, quotas, delivery behavior, and cost require measurement.
- An ephemeral environment requires reliable provisioning, inventory, and teardown automation.

### Neutral and operational

- Free Tier and credits reduce possible cost, but AWS Demo is not guaranteed to be free and can produce a bill.
- The exact Lambda packaging option remains a service/IaC decision.
- AWS pricing and eligibility must be checked for the selected account and Region before every deployment.

## Alternatives considered

| Alternative | Benefits | Drawbacks for this platform | Why not selected now |
| --- | --- | --- | --- |
| AWS-only development | Highest hosted-environment fidelity | Requires credentials and can incur cost for every contributor | Conflicts with Local Zero AWS Cost. |
| LocalStack as a prerequisite | AWS-like local APIs | Adds resource cost, complexity, and emulator coupling | Replaceable ports and deterministic adapters are sufficient for the default profile. |
| Always-running ECS/Fargate or EC2 | Familiar long-running hosting | Persistent compute cost and larger operational footprint | The demonstration workload is intermittent and event-driven. |
| EKS | Broad orchestration flexibility | Significant complexity and persistent infrastructure | Not justified by the walking skeleton. |
| RDS, MSK, OpenSearch, or always-on caches | Rich managed capabilities | Persistent cost and additional lifecycle burden | DynamoDB, EventBridge, SQS, and small projections cover the approved slice. |
| Deploy all bounded contexts initially | Shows the complete target landscape | Expands cost and implementation before entry criteria are met | The walking skeleton intentionally proves only three contexts. |

## Constraints and guardrails

- Use one Region, bounded deployment windows, explicit expiry tags, and a pre-deployment cost estimate.
- Revalidate runtime support, service pricing, and Free Tier/credit eligibility immediately before deployment.
- Use small synthetic payloads and conservative memory, timeout, concurrency, and logging.
- Disable verbose logging by default; do not enable unnecessary distributed tracing or Provisioned Concurrency.
- Configure budget/anomaly notifications while recognizing their detection and billing latency.
- Apply the retention and verified-teardown policy in the [Security Model](../architecture/SECURITY_MODEL.md#10-aws-demo-retention-and-verified-deletion).
- Do not include NAT Gateway, EKS, EC2, always-running ECS/Fargate, RDS, MSK, OpenSearch, DAX, Global Tables, multi-Region resources, always-on caches, persistent-cost infrastructure, or separately billed key infrastructure by default.
- Keep `Q-002` open: exact Region, account topology, SCPs, IAM permission boundaries, preventative controls, and deployment-account isolation are not decided here.

## Validation and revisit triggers

Revisit before the first deployment; when Lambda runtime support, pricing, Free Tier, credits, or service availability changes; when load/cost tests challenge `H-004` or `H-005`; or when a new context enters AWS Demo. Production adoption requires a separate architecture and security review.

## References

- [Assumption A-002 and Q-002](../discovery/ASSUMPTIONS.md)
- [Walking skeleton](../discovery/SCOPE.md#walking-skeleton-boundary)
- [Execution profiles](../architecture/CONTAINER_DIAGRAM.md#5-local-zero-aws-cost-overlay)
- [Security Model](../architecture/SECURITY_MODEL.md)
- [AWS Lambda supported runtimes](https://docs.aws.amazon.com/lambda/latest/dg/lambda-runtimes.html)
- [AWS Lambda .NET container images](https://docs.aws.amazon.com/lambda/latest/dg/csharp-image.html)
- [AWS Lambda pricing](https://aws.amazon.com/lambda/pricing/)
- [API Gateway pricing](https://aws.amazon.com/api-gateway/pricing/)
- [EventBridge pricing](https://aws.amazon.com/eventbridge/pricing/)
- [Amazon SQS pricing](https://aws.amazon.com/sqs/pricing/)
- [Amazon SQS at-least-once delivery](https://docs.aws.amazon.com/AWSSimpleQueueService/latest/SQSDeveloperGuide/standard-queues-at-least-once-delivery.html)
- [DynamoDB pricing](https://aws.amazon.com/dynamodb/pricing/)
- [Amazon Cognito pricing](https://aws.amazon.com/cognito/pricing/)
- [Amazon S3 pricing](https://aws.amazon.com/s3/pricing/)
- [Amazon CloudWatch pricing](https://aws.amazon.com/cloudwatch/pricing/)
- [AWS Free Tier](https://docs.aws.amazon.com/awsaccountbilling/latest/aboutv2/free-tier.html)
- [AWS Budgets](https://docs.aws.amazon.com/cost-management/latest/userguide/budgets-managing-costs.html)
- [AWS Cost Anomaly Detection](https://docs.aws.amazon.com/cost-management/latest/userguide/getting-started-ad.html)

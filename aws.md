Table of contents

Core AWS security components (short explainer)

VPC, Subnets, Route Tables

Security Groups (SG) vs Network ACLs (NACL)

IAM & STS

S3, KMS

EC2, ECS, EKS, Lambda

API Gateway, ALB, CloudFront, WAF

Secrets Manager & SSM Parameter Store

Logging & Detection (CloudTrail, VPC Flow Logs, GuardDuty)

How IAM Roles work & how they are attached

Design: Service-to-Service (Inside AWS)

Design: Service-to-Service (AWS → External service)

Design: User-to-Service (User → App → DB)

Quick hardening checklist

References / next steps (practical labs)

Core AWS security components (short explainer)
VPC (Virtual Private Cloud)

A virtual network in which you run your AWS resources.

Controls IP ranges, subnets, route tables, and network gateways.

Use multiple subnets (public/private) to separate internet-facing components from internal ones.

Subnets & Route Tables

Public subnet: route to Internet Gateway — hosts internet-facing resources (ALB, NAT per design).

Private subnet: no direct internet route — hosts databases and internal services.

Route tables define where traffic from a subnet can go.

Security Groups (SG)

Virtual firewalls attached to an instance (EC2, ENI, RDS) or workload (ECS task).

Stateful (responses to allowed outbound are automatically allowed inbound).

Rules are attached by allow semantics only.

Best practice: specific host/port rules, limit CIDRs, restrict to other SGs (allow SG-to-SG).

Network ACLs (NACL)

Stateless subnet-level firewall applied to inbound/outbound traffic.

Process both allow and deny rules, order matters.

Useful for coarse-grained controls (e.g., block suspicious IP ranges) but not for fine-grained host filtering—use SGs for that.

IAM (Identity and Access Management)

Controls who/what can do what on AWS resources. Key concepts:

Principals: users, groups, roles, services.

Policies: JSON documents defining Allow/Deny of actions on resources.

Least privilege: give only the minimum permissions necessary.

Managed policies vs Inline policies vs Resource policies (S3 bucket policy, KMS key policy).

STS (Security Token Service)

Creates temporary credentials (short-lived access keys) via AssumeRole / AssumeRoleWithWebIdentity.

Used for cross-account access and for giving short-lived access to services/users.

Highly recommended over long-lived static keys.

S3

Object store. Security controls:

Bucket policies and ACLs.

Block public access settings.

Encryption at rest using S3-managed keys or customer-managed KMS keys.

Access via IAM policies (and signed URLs for temporary access).

KMS (Key Management Service)

Manage encryption keys (CMKs).

Use KMS to encrypt EBS, S3 objects, RDS at rest.

KMS key policies + IAM policies control who can use/ administer keys.

EC2 / ECS / EKS / Lambda (compute)

EC2: VM instances with instance profiles (IAM role via instance profile).

ECS (EC2 or Fargate): tasks have task roles.

EKS: pods are Kubernetes; attach AWS permissions via IAM Roles for Service Accounts (IRSA) (recommended).

Lambda: execution role attached to function; grants permissions needed at runtime.

API Gateway / ALB / CloudFront

API Gateway: public HTTP endpoint for APIs; supports IAM auth (SigV4), Cognito, Lambda authorizers, custom authorizers.

ALB (Application Load Balancer): front for HTTP(S) services in a VPC.

CloudFront: CDN; can terminate TLS, integrate WAF, and front API Gateway/ALB.

WAF (Web Application Firewall)

Protect applications from OWASP-style attacks (SQLi, XSS), bot management, rate limiting.

Attach to CloudFront, ALB, API Gateway.

Secrets Manager & SSM Parameter Store

Secrets Manager: rotate and manage secrets, credentials, DB passwords.

SSM Parameter Store: store parameters; can be encrypted with KMS.

Logging & Detection

CloudTrail: records API calls for auditing and forensics.

VPC Flow Logs: network traffic logs for subnets/NICs.

GuardDuty: threat detection service analyzing logs/flows.

CloudWatch: metrics, logs, alarms.

Centralize logs to S3/ElasticSearch/Splunk for SIEM.

How IAM roles work & how they are attached
What is an IAM Role?

A role is an AWS identity with a set of permissions (policy). It is assumed by trusted principals (an EC2 instance, Lambda function, a user, another AWS account, or a web identity such as Cognito).

Why roles vs keys?

Roles produce temporary credentials (via STS) and avoid long-lived static access keys on disk. They support least-privilege and rotation by design.

How roles are attached (common examples)

EC2: Attach a role via an instance profile. Processes on the instance call the instance metadata service to obtain temporary credentials.

Lambda: Assign an execution role to the function. Lambda automatically assumes the role when executing.

ECS: Use task role (for accessing AWS APIs from containers) and task execution role (for pulling images, sending logs).

EKS: Use IAM Roles for Service Accounts (IRSA) — associate a Kubernetes service account with an IAM role and an OIDC provider; pods that run under that service account can assume the IAM role.

API Gateway: Can use IAM authorizers (SigV4) or invoke Lambdas using an IAM role for integration.

Cross-account: Use AssumeRole to grant cross-account access; STS issues temporary creds.

Role trust policy vs permissions policy

Trust policy: Who is allowed to assume the role (Principal section).

Permissions policy: What actions the role can perform (attached policy).

Example: EC2 instance profile

Create IAM role with a trust policy that allows ec2.amazonaws.com to assume it.

Attach permissions (e.g., S3:GetObject for the application).

Attach the role to the instance via an instance profile. Processes use the metadata endpoint to obtain temporary credentials.

Design — Service-to-Service (Inside AWS)
Goals

Ensure least privilege for each service

Keep traffic internal (private endpoints, VPC endpoints)

Enforce network segmentation + runtime checks

Secure authentication between services (IAM roles, mTLS, tokens)

Key building blocks

Private subnets for services/data

Security Groups allowing only required ports and SG-to-SG rules

VPC endpoints (PrivateLink) for AWS services (S3, Secrets Manager) to avoid internet egress

IAM task/instance/service-account roles for auth

mTLS or signed JWTs for service-to-service application-level auth (optional but recommended)

Logging & runtime detection (CloudWatch, Falco/third-party)

Mermaid diagram (inside AWS)
flowchart TD
  subgraph VPC
    subgraph Private_Subnet
      SvcA[ECS/EKS pod: svc-a]
      SvcB[RDS / Postgres]
      SvcBsg[SG: db-sg]
      SvcAsg[SG: svc-a-sg]
    end
    subgraph Public_Subnet
      ALB[ALB internal / NLB]
    end
  end

  ALB -->|forward| SvcA
  SvcA ---|connects over port 5432| SvcB
  SvcA ---|IAM role (task role) via IRSA| IAM[(IAM Role)]
  SvcB ---|Protected by security group| SvcBsg
  SvcA ---|SG rules allow only SvcA SG| SvcAsg

  note right of SvcA: - Runs in private subnet\n- Task role for AWS APIs\n- Uses secrets from Secrets Manager (VPC endpoint)

Explanation

Service A runs in a private subnet and only allows inbound traffic from the ALB or from specific SG(s).

Service B (DB) is in a private subnet, only accessible on DB port from SvcA's SG.

IAM Role for the service is attached (ECS task role or IRSA for EKS), granting only required AWS API permissions (e.g., decrypt with KMS, get secrets).

Use VPC endpoints (Interface endpoints for Secrets Manager, S3 gateway endpoints) to avoid internet egress.

Monitor and detect via CloudWatch, GuardDuty, and Falco/other runtime tools.

Design — Service-to-Service (AWS → External Service)
Considerations

Use egress controls (NAT + SG rules + proxy) to restrict outbound traffic.

Use TLS, mTLS, or signed requests (SigV4 or JWT).

Use PrivateLink or API Gateway + VPC Link for more secure connectivity when possible.

For third-party SaaS, avoid embedding credentials; use short-lived credentials or OAuth flows.

Mermaid diagram (AWS → External)
flowchart TD
  subgraph VPC
    SvcA[ECS/EKS pod: svc-a]
    NGW[NAT Gateway]
    Proxy[Outbound Proxy (Squid / Envoy)]
  end

  SvcA -->|outbound via| Proxy
  Proxy -->|mtls or TLS| ExternalAPI[External API (3rd party)]
  Proxy -->|use IAM/STS| STS[(STS AssumeRole or OAuth token)]

Explanation / controls

Egress proxy ensures all outbound requests are inspected, and credentials are centrally managed or rotated.

Use mTLS for strong mutual authentication.

Exchange tokens via secure flows (OAuth) or use STS/AssumeRole for cross-account AWS services.

Lock down SGs to prevent lateral outbound from other subnets.

Design — User-to-Service (User → App → DB)
Goals

Protect public entry points (WAF, rate limiting)

Authenticate & authorize users (Cognito / OIDC / OAuth2)

Keep app servers and DB in private subnets

Use IAM for service-level operations only, not for user-level app auth

Mermaid diagram (User → App → DB)
flowchart LR
  User[User (browser / mobile)]
  CF[CloudFront (optional) + WAF]
  API[API Gateway / ALB]
  App[Service (Lambda / ECS / EKS)]
  Secrets[Secrets Manager / Parameter Store]
  DB[RDS (private subnet)]

  User --> CF
  CF --> API
  API --> App
  App --> Secrets
  App --> DB
  App -->|assume role| IAM[(IAM Role attached to App)]


ASCII fallback:

User (browser)
   |
CloudFront + WAF (rate-limit, OWASP rules)
   |
API Gateway or ALB (TLS, authorizer e.g., Cognito/OIDC)
   |
App (Lambda / ECS / EKS)  --(IAM role for AWS access)--> Secrets Manager
   |
Private DB (RDS) in private subnet, only accessible from App's SG

Explanation / recommended flow

Edge protections: CloudFront + WAF to block common web attacks and filter bad traffic early.

Authentication: API Gateway or ALB uses an authorizer (Cognito, OIDC, or custom JWT/Lambda authorizer). This handles user auth, not IAM.

Application layer: App runs in private subnets. It uses an IAM role (Lambda execution role, ECS task role, or EKS pod IRSA) to access AWS APIs and secrets.

Secrets: Store DB credentials in Secrets Manager and grant the App’s IAM role secretsmanager:GetSecretValue only for the specific secret.

DB: RDS in private subnet, SG allows only the App SG on DB port. No public access.

Encryption: TLS in transit for user ↔ edge and app ↔ DB; KMS for data-at-rest encryption.

Auditing: CloudTrail logs API calls; VPC Flow Logs + DB audit logs to detect suspicious behavior.

IAM: more practical notes & examples
Typical minimal-role attachments

Lambda: lambda:CreateFunction (admin) — but runtime ExecutionRole typically has secretsmanager:GetSecretValue, kms:Decrypt for KMS-encrypted secrets, and any service API calls the function needs (S3, DynamoDB minimal actions).

EC2: instance profile with role that has S3:GetObject if the app pulls assets from S3.

ECS: taskRoleArn — used by app code; executionRoleArn used by ECS agent to pull images or send logs.

EKS: IRSA maps Kubernetes service account to an IAM role - does kubectl apply for the service account and IAM OIDC provider configuration.

Cross-account patterns

Create role in account B with trust policy allowing arn:aws:iam::A:role/CallerRole or principal account A.

From account A, AssumeRole (STS) to get short-lived creds for account B resources.

Policy examples (high-level)

Very specific resource ARNs, e.g.:

{
  "Effect":"Allow",
  "Action":["secretsmanager:GetSecretValue"],
  "Resource":["arn:aws:secretsmanager:us-east-1:123456789012:secret:prod/db-creds-123"]
}


Avoid * unless absolutely necessary.

Quick hardening checklist (runtime & infra)

Network & perimeter

Enforce WAF on public endpoints.

Use CloudFront for caching + edge blocking + TLS.

Close direct internet access to backend resources; use private subnets.

Identity & access

Use IAM roles for all compute, avoid static credentials.

Principle of least privilege: least privilege IAM policies, narrow resource ARNs.

Use STS & short-lived credentials for cross-account access.

Secrets & keys

Store secrets in Secrets Manager / Parameter Store (KMS encrypted).

Use KMS CMKs with narrow key policies.

Rotate keys and secrets regularly.

Images & CI/CD

Scan images (Trivy/Grype) and IaC (Checkov/tfsec) in CI.

Enforce image signing (Cosign) and verify signatures before deploy.

Implement CI test gating — fail pipelines on high/critical issues.

Runtime & detection

Deploy Falco / eBPF-based detection in clusters (or host agents).

Enable CloudTrail, VPC Flow Logs, and forward logs to SIEM.

Enable GuardDuty and integrate with response playbooks.

Data protection

TLS for all in-transit traffic; enable RDS encryption & S3 encryption.

Limit S3 bucket policies; block public access unless needed.

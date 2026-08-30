# AWS Revision Notes — Mock Interview Session

Each topic: full explanation (the "why") → **Say this** (tightened spoken answer) → follow-up notes.

Note: your resume only shows hands-on EC2 deployment experience. If asked something deep outside this doc, an honest "I've worked hands-on with EC2 for deployment, but haven't gone deep on [X]" is a perfectly fine answer — better than a shaky guess.

---

## 1. SQS vs SNS
**Explanation:** SQS is a **queue** — point-to-point. A producer drops a message in; **one consumer** polls (pulls) the queue, processes the message, and deletes it. Good for work that should happen exactly once, like "process this order." SNS is **pub/sub** — a message published to a topic is **pushed** out to **every subscriber** (could be multiple SQS queues, Lambda functions, email endpoints). Good for broadcasting one event to multiple independent systems. Common real pattern: **SNS fan-out to SQS** — SNS broadcasts to multiple SQS queues, combining durability/buffering with broadcast behavior. Conceptually similar to Kafka: SQS ≈ one consumer group processing a queue; SNS ≈ a topic with multiple independent consumer groups.

**Say this:**
> "SQS is a queue — it acts as a buffer between producer and consumer. The producer drops a message in, and the consumer polls the queue to pull messages, processes them, and deletes them once done. SNS is pub/sub — publish/subscribe — push-based: it pushes the notification out to every subscriber, instead of subscribers polling for it. A common pattern is combining them — SNS fans out to multiple SQS queues, so you get both the durability of a queue and the broadcast behavior of pub/sub."

---

## 2. S3 + Presigned URLs
**Explanation:** S3 is **object storage** — files (objects) live in buckets, identified by a key, accessed over HTTP(S). Not a traditional filesystem. Fully managed — no servers to run. Useful for anything that shouldn't sit in your database or a container's local disk: invoice PDFs, uploads, backups, logs.

**Why not just store files on the container/EC2 instance itself:** two real problems, not "containers can't access files":
1. **Ephemeral storage** — if a container restarts, crashes, or gets replaced during a rolling deploy, its local filesystem is wiped — anything written there is gone.
2. **Instance-specific storage** — with multiple replicas running (for scaling), each has its own separate local disk; a file written by one instance isn't visible to another.

S3 solves both — durable (survives restarts) and shared (every instance/replica can access the same bucket via HTTP).

**Presigned URLs:** the problem — you don't want the bucket public (anyone could access every file, forever), but you do want one specific user to access one specific file temporarily. Solution: your backend (using its own AWS credentials) generates a special URL granting **temporary, time-limited access to one specific object** (e.g. 15 minutes). The client uses that URL directly against S3 — no AWS credentials needed client-side, and the file never passes through your backend server.

**Say this:**
> "S3 is object storage — it stores files as objects inside buckets, identified by a key, accessed over HTTP. It's fully managed, so you don't run servers for it. It's not that containers can't access files — it's that container storage is ephemeral and instance-specific: if a container restarts it loses local files, and multiple replicas don't share local disk. S3 is durable and shared across every instance. A presigned URL solves a related problem — you don't want the bucket public, since that exposes every file indefinitely. Instead, the backend generates a temporary, time-limited URL scoped to one specific object, and the client uploads or downloads directly to or from S3 using that URL, without ever having AWS credentials themselves."

---

## 3. IAM Users vs IAM Roles
**Explanation:** Every AWS API call needs proof of identity — something has to tell AWS "who is making this request, are they allowed." **IAM User** = a permanent identity with long-lived credentials (password for console, or access key + secret key for programmatic/API access). These **never expire on their own** — only manual rotation/deletion removes them. **IAM Role** = no permanent credentials at all. Something (an EC2 instance, ECS task, Lambda) **assumes** the role, and AWS hands out **short-lived, auto-rotating temporary credentials** (typically ~1 hour) for as long as it's needed.

**Why roles are used for EC2/ECS instead of hardcoded user access keys:** if a user's access key leaks (committed to Git, exposed in logs, extracted from a compromised container), it's valid **indefinitely** until someone manually notices and revokes it — open-ended damage. A role's temporary credentials are only ever valid for a short window even if leaked — bounded, mostly harmless damage. There's also nothing to leak in the first place, since no long-lived key exists anywhere in the code/config.

**How a role actually attaches to EC2 — the mechanism:**
1. Create an IAM role with a policy attached (e.g. "allow s3:PutObject on bucket X").
2. At EC2 launch (or ECS task definition), attach that role via **"IAM instance profile"** (EC2) or **"task role"** (ECS).
3. Once running, the instance can reach an internal-only **instance metadata service** (fixed internal address `169.254.169.254`, reachable only from inside that instance) that hands out temporary credentials for that specific role.
4. The AWS SDK inside your app is already coded to automatically check that metadata endpoint and use those credentials — no code change, no keys in config.
5. Credentials auto-expire (~1hr) and auto-refresh, for as long as the instance runs with that role attached.

**Result:** anything running on that instance automatically inherits the role's permissions — your app never needs to know the role exists.

**Developer (human) access — a separate, legitimate use case:** an admin creates an IAM user for you, scoped with least-privilege permissions (only what you need). You get console access and/or programmatic access (access key + secret key). Locally, `aws configure` stores your credentials at `~/.aws/credentials` so your terminal/AWS CLI can run commands authenticated as you. This is **separate** from how the application itself authenticates in production (which uses a role, not your personal credentials). Best practice: MFA on developer accounts, and increasingly IAM Identity Center (SSO) for temporary rotating credentials instead of static long-lived keys.

**Say this (user vs role):**
> "An IAM user has long-lived credentials — access keys that don't expire on their own, meant for a person or occasionally a long-lived external app. An IAM role has no permanent credentials — something like an EC2 instance or ECS task assumes the role, and AWS hands out short-lived, auto-rotating temporary credentials for the duration it's needed. You use a role instead of hardcoding a user's access keys because hardcoded keys are a security risk — if they leak, they're valid indefinitely — while a role's temporary credentials expire automatically."

**Say this (attachment mechanism, if asked "how"):**
> "You create the role with a policy defining its permissions, then attach it to the EC2 instance at launch through an IAM instance profile. Once running, the instance can reach an internal metadata service that hands out temporary credentials for that role, and the AWS SDK in your app automatically fetches and refreshes them — no keys anywhere in the code or config."

**Say this (developer access, if asked):**
> "As a developer, you'd be given an IAM user scoped with least-privilege permissions, and configure the AWS CLI locally with your access key using `aws configure` to run commands authenticated as yourself. That's separate from how the application authenticates in production — the app uses an IAM role, not a developer's personal credentials."

---

## 4. RDS — Multi-AZ vs Read Replicas
**Explanation:** RDS = AWS's managed relational database service (Postgres, MySQL, etc.) — AWS handles the server, patching, and backups.

**Multi-AZ (high availability / disaster recovery):** "AZ" = Availability Zone, a physically separate data center within a region. RDS maintains a **synchronous standby replica** in a different AZ — every write is replicated in real time. If the primary fails, RDS **automatically fails over** to the standby (usually within a minute or two), no data loss. The standby is **not used for regular traffic** — it's a hidden hot backup only, purely for uptime/disaster recovery.

**Read Replicas (read scalability):** a separate, **actively queryable** copy you send `SELECT` traffic to, offloading reads from the primary. Replication is typically **asynchronous** — small lag (ms to seconds) between a write on primary and it appearing on the replica. Not for disaster recovery (though it *can* be manually promoted in an emergency) — its main job is scaling read-heavy workloads.

**Can use both together** — Multi-AZ for failover safety, read replicas for scaling reads — they solve different problems.

**Say this:**
> "RDS is AWS's managed relational database service — it handles the underlying server, patching, and backups. Multi-AZ is about high availability — RDS maintains a synchronous standby replica in a different Availability Zone, and if the primary fails, it automatically fails over with no data loss, usually within a minute or two. The standby isn't used for regular traffic — it's purely a hot backup. A read replica is different — it's an actively queryable copy you send read traffic to, offloading reads from the primary, with replication that's typically asynchronous. You'd use Multi-AZ for disaster recovery and read replicas for scaling read-heavy workloads — they solve different problems and you can use both together."

---

## 5. VPC (Virtual Private Cloud)
**Explanation:** AWS runs shared physical infrastructure across millions of customers. A VPC is your own **logically isolated network** carved out within AWS — you define your own private IP range (e.g. `10.0.0.0/16`), and everything you launch (EC2, RDS, etc.) lives inside it, invisible to and isolated from every other customer's VPC.

**Subnets** divide a VPC into smaller IP ranges, each tied to one Availability Zone:
- **Public subnet** — has a route to the internet (via an Internet Gateway). Put internet-facing things here — a load balancer, a bastion host.
- **Private subnet** — no direct internet route. Put your actual app servers and databases here — they can initiate *outbound* connections (via a NAT Gateway) but nothing from the internet can reach *in* directly.

**Concrete example:** your order platform's EC2 instances and RDS database sit in a private subnet — not directly reachable from the internet — while a load balancer in a public subnet is the only thing exposed, forwarding traffic inward. Standard pattern: minimize what's directly internet-facing.

**Say this:**
> "A VPC is your own logically isolated network within AWS, with your own private IP range. You divide it into subnets — public subnets, which have a route to the internet, for things like load balancers, and private subnets, with no direct internet route, for your actual application servers and databases."

---

## 6. Security Groups
**Explanation:** While a VPC/subnet controls broad network layout, a **Security Group** is a virtual firewall controlling exactly what traffic is allowed **in and out of a specific resource** (an EC2 instance, RDS instance, etc.) — by port, protocol, and source/destination.

**Key property — stateful:** if inbound traffic on a port is allowed, the response traffic is automatically allowed back out — no matching outbound rule needed. (Contrast: Network ACLs, a subnet-level construct, are stateless and need explicit rules both directions — less commonly asked, but a good contrast to know.)

**Common real pattern — security groups referencing each other, not just IPs:** instead of "allow traffic from IP range X," you write "allow traffic from any resource in Security Group Y." Example: an RDS database's security group allows inbound Postgres (port 5432) only from instances in the `order-service-sg` group — meaning only your actual app servers can reach the database, regardless of their specific IP, even if instances get replaced or rescaled. Far more maintainable than hardcoded IPs.

**Concrete example (your order platform, layered):**
- Load balancer's security group (`web-sg`) — allows inbound HTTPS (443) from anywhere, since it's the public entry point.
- EC2 order service's security group (`app-sg`) — allows inbound traffic only from `web-sg`, not from the internet directly.
- RDS's security group (`db-sg`) — allows inbound Postgres (5432) only from `app-sg`.

Each layer only trusts the layer directly in front of it — the database never talks to the internet, and the app server is never reachable except through the load balancer.

**Say this:**
> "A security group is a stateful virtual firewall attached to a specific resource, like an EC2 instance — it controls inbound and outbound traffic by port and source. A common pattern is having security groups reference each other instead of hardcoded IPs — for example, an RDS security group that only allows traffic from instances in your application's security group, so only your app servers can reach the database, regardless of their actual IP. In a typical setup, you'd layer this — the load balancer's security group allows public HTTPS traffic, the app server's security group only allows traffic from the load balancer, and the database's security group only allows traffic from the app server — so nothing is more exposed than it needs to be."

**How the load balancer fits into this setup:**
- **Single public entry point** — it's the only component with a public IP; clients hit its DNS name, never the EC2 instances directly.
- **Distributes traffic** across multiple backend instances (round-robin or load-based) when running more than one replica for scaling/availability.
- **Health checks** — continuously pings each backend instance (e.g. a `/health` endpoint) and stops routing to any unhealthy instance automatically. Same underlying goal as a circuit breaker, but at the infrastructure layer instead of inside application code.
- **SSL/TLS termination** — handles HTTPS decryption at the edge; can forward plain HTTP internally within AWS's private network, offloading encryption cost from app servers.
- **Pairs with Auto Scaling Groups** — can work with an ASG that adds/removes EC2 instances based on load, automatically picking up new instances once they pass health checks.
- Sits in the public subnet specifically because it's the only component that genuinely needs a route to/from the internet — everything behind it stays private.

**Security groups with multiple applications on one instance:**
Security groups attach at the **network interface level** — effectively the whole EC2 instance — not per application. They only see ports and IPs, with no concept of "which process" owns a port. If two apps run on the same instance (e.g. order service on 8080, a reporting service on 8081), you configure rules **per port** — any traffic to an allowed port reaches whatever's listening there, regardless of which app that is:
```
Inbound rules (attached to the whole instance):
  - Allow TCP 8080 from web-sg   → reaches order service
  - Allow TCP 8081 from web-sg   → reaches reporting service
```
If two apps on the same instance genuinely need **different** access rules, a single security group can't fully separate them. Real solutions:
- Separate EC2 instances per app — cleanest, but more infrastructure.
- **ECS task-level security groups** — each ECS task (not the whole instance) can get its own security group.
- **Kubernetes Network Policies** — a K8s-native way to control which pods can talk to which, independent of which node they're scheduled on.
- Multiple ENIs per instance — less common, more advanced.

**Say this (if asked about multi-app instances):**
> "Security groups attach at the network interface level — essentially the whole EC2 instance — not per application. If multiple apps run on one instance, you configure rules per port, and any traffic to an allowed port reaches whatever's listening there, regardless of which app it is. If two apps on the same instance genuinely need different access rules, a single security group can't fully separate them — that's usually solved either with separate instances, or in a containerized setup, with ECS task-level security groups or Kubernetes Network Policies, which let you control access per container or pod instead of per instance."

---

## 7. Quick one-liners (lower priority, but worth a sentence if asked)

**CloudWatch** — AWS's monitoring and logging service — metrics, alarms, dashboards, and log aggregation for your services.

**Lambda (serverless)** — functions that run without managing servers, billed per invocation. Key trade-off: **cold starts** — the first invocation after idle time is slower because AWS has to spin up a fresh execution environment. This is more pronounced for JVM-based Lambdas (Java) than lighter runtimes like Node/Python, since JVM startup itself is heavier — a known real-world concern for Java on Lambda.

---

## Delivery reminders
- On anything genuinely outside your hands-on experience (deep ECS/EKS internals, Data Mesh, X-Ray specifics), it's fine to say plainly: "I've worked hands-on with EC2 for deployment, but I haven't gone deep on that specifically."
- Keep IAM user vs role anchored to the leaked-credential story — that's the reasoning that actually made it click tonight, use that same story if you get a follow-up.
- RDS: the one-line hook to remember — "Multi-AZ is a hidden backup for failover, a read replica is an actively queryable copy for scaling reads."

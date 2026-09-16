# AWS EC2 Instance Metadata Service (IMDS): Session Token Engineering

This technical document details the hypervisor link-layer communication mechanics of the EC2 Instance Metadata Service (IMDS), contrasting the open access model of IMDSv1 with the session-token defense protocols mandated by IMDSv2.

---

## 📡 1. The Link-Local Boundary (`169.254.169.254`)

Every EC2 instance accesses its own operational metadata via a non-routable, link-local IPv4 address: **`http://169.254.169`**.
*   **Hypervisor Isolation:** This address is unique to the specific underlying host hardware. It does not transit across the public internet or private VPC networks. It can only be called from inside the operating system of that explicit instance.
*   **Data Scoping:** Programmatic scripts use this pathway to dynamically discover instance parameters at runtime, including the machine's `instance-id`, local `public-ipv4`, network `security-groups`, and temporary IAM security credentials generated via attached instance profiles.

---

## 🛡️ 2. Architectural Security Vulnerability: IMDSv1 vs. IMDSv2

The architectural evolution from IMDSv1 to IMDSv2 represents a critical boundary shift in cloud infrastructure hardening, designed to mitigate **Server-Side Request Forgery (SSRF)** attack vectors.

### A. IMDSv1: The Open Request Vector (SSRF Target)
Under the legacy IMDSv1 architecture, data extraction relies on basic HTTP GET communication channels:
```bash
# IMDSv1 Data Extraction Syntax
curl http://169.254.169iam/security-credentials/Corporate-Web-Role
```
*   **The Flaw:** If a developer deploys an application with an unpatched reverse-proxy vulnerability or a bad code input validation bug, a remote attacker can exploit the app to make an internal query to the link-local address. The app reads the metadata and passes the administrative temporary IAM keys right back to the attacker over the public internet.

### B. IMDSv2: The Session-Oriented Defense Matrix
IMDSv2 completely closes this vector by mandating **Session-Oriented Handshakes**. It enforces two strict constraints: a stateful **Token Request** and a strict **Hop-Limit Guardrail**.

1.  **The Token Request (PUT Block):** An application cannot simply request metadata. It must first execute a cryptographic handshake using an HTTP `PUT` request containing a required time-to-live header to generate a temporary, single-use access token.
2.  **The Metadata Request (GET Block):** The app must pass that explicit session token inside an authorization header on all subsequent metadata calls.

```text
  [ EC2 Instance Terminal ]                         [ Hypervisor Metadata Store ]
              │                                                   │
              │  ─── 1. HTTP PUT (Token Request with TTL) ────►   │
              │  ◄── 2. Returns Session Token String ──────────   │
              │                                                   │
              │  ─── 3. HTTP GET (Passes Token in Header) ────►   │
              │  ◄── 4. Returns Secure Metadata Output ────────   │
```

---

## 💻 3. Programmatic Real-World Execution Blueprints

To enforce IMDSv2 via terminal automation scripts or secure system runbooks, the execution pipeline must be written in a sequential, session-bound chain:

```bash
#!/bin/bash
# Step 1: Execute the stateful PUT command to generate an explicit 60-second Session Token
TOKEN=\$(curl -X PUT "http://169.254.169" -H "X-aws-ec2-metadata-token-ttl-seconds: 60")

# Step 2: Utilize the variable token header to securely extract instance metadata fields
INSTANCE_ID=(curl -H "X-aws-ec2-metadata-token: TOKEN" -s http://169.254.169instance-id)
PUBLIC_IP=(curl -H "X-aws-ec2-metadata-token: TOKEN" -s http://169.254.169public-ipv4)

echo "Node Initialization Complete: Target Host \(INSTANCE_ID bound to IP\)PUBLIC_IP"
```

---

## 🔒 4. Enterprise Compliance Enforcement

To satisfy security audit perimeters, modern production environments completely disable IMDSv1 account-wide using explicit CLI command flags:

```bash
# Forcefully restrict compute slot to accept IMDSv2 ONLY (Blocks legacy GET vectors)
aws ec2 modify-instance-metadata-options \
    --instance-id i-0123456789abcdef0 \
    --http-tokens required \
    --http-endpoint enabled
```
*   **The Hop-Limit Rule:** By setting the metadata HTTP hop limit to `1`, AWS ensures that if an application uses a containerized architecture (like Docker), the packet cannot jump out of the container to hit the metadata line, completely neutralizing advanced container container-escape attacks.

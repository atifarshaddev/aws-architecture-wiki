# AWS High Availability & Elastic Scalability: ELB & ASG Architecture

This technical document details the design specifications for routing tier orchestration via Elastic Load Balancing (ELB) and the self-healing, automated scaling parameters managed by Auto Scaling Groups (ASG).

---

## 🚦 1. Elastic Load Balancing (ELB) Traffic Control Plane

An Elastic Load Balancer (ELB) serves as a decoupled, public-facing single point of entry for infrastructure networks, programmatically distributing incoming client streams across a target group of backend computing nodes.

### A. Core Functional Classifications
*   **Application Load Balancer (ALB - Layer 7):** Operates at the application layer of the OSI model. Processes HTTP and HTTPS traffic protocols natively. Enables advanced content-based routing mechanics, including URL pathway routing (e.g., `/api` vs `/static`) and host-header evaluations.
*   **Network Load Balancer (NLB - Layer 4):** Operates at the transport layer, processing raw TCP, UDP, and TLS streams. Engineered for ultra-high throughput performance requirements capable of handling millions of concurrent requests per second with microsecond latencies.

### B. Target Group Interactivity & State Mechanics
*   **Health Verification Loops:** The load balancer continuously executes active application pings against backend instances based on pre-defined thresholds (e.g., HTTP 200 response on path `/`). If a node fails successive verification checks, it is dynamically isolated from the routing pool.
*   **Target Group Decoupling:** Compute instances are abstractly managed inside Target Groups. This abstraction allows servers to be torn down, upgraded, or scaled out automatically without modifying public DNS address endpoints.

---

## 🔄 2. Auto Scaling Group (ASG) Automated Lifecycle Mechanics

An Auto Scaling Group provides horizontal elasticity and total fault tolerance by dynamically expanding or contracting the EC2 instance pool to precisely match real-time system resource demands.

### A. Capacity Optimization Boundaries
The scaling cluster is enforced by three programmatic baseline metrics:
*   **Minimum Size (Min):** The infrastructure floor. Guarantees a permanent minimum allocation of compute capacity across multiple Availability Zones to ensure baseline system survival even during zero-traffic conditions.
*   **Maximum Size (Max):** The budget and scale ceiling. Prevents rogue application logic errors (e.g., recursive memory leaks) or massive traffic spikes from triggering infinite instance generation and blowing out enterprise cloud expenditure.
*   **Desired Capacity:** The active targeted runtime state. The ASG automation engine continually self-heals infrastructure to match this target.

### B. Self-Healing Pipelines & Cooldown Guardrails
*   **Chained Health Evaluation:** When the ASG is linked to an ELB, the group overrides basic hypervisor checks and tracks application health. If the ALB marks an instance unhealthy, the ASG terminates the failed node, reads the master Launch Template, and initializes a fresh, bootstrapped clone automatically.
*   **The Scaling Cooldown Period:** Following any active scaling expansion or contraction event, the ASG enters a strict, timed **Cooldown Period** (Default: 300 seconds). During this window, all further scaling triggers are forcefully frozen. This allows newly initialized servers adequate time to run their User Data bootstrap scripts, join the target group network, and safely absorb traffic before the cloud metrics engine measures system utilization loops again.

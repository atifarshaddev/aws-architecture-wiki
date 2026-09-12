# AWS EBS Storage Fabrics, Snapshot Delta Engineering, and AMI Blueprints

This technical document details the architectural boundaries of Elastic Block Store (EBS) network drives, the block-level mechanics of incremental snapshot backups, and the composition of bootable Amazon Machine Images (AMIs).

---

## 💾 1. Pluggable Storage Layer Optimization Matrix

When provisioning compute instances, storage must be decoupled based on durability, persistence, and performance requirements to prevent data loss during hardware failures.

### A. Amazon EBS (Elastic Block Store) - Network-Attached Persistence
An EBS volume is a virtual hard drive that does not physically reside inside the instance’s host hardware rack [3.1]. Instead, it is network-attached via a hyper-fast private storage fabric [3.1].
*   **Hardware Crash Survival:** Because EBS exists independently on separate storage hardware, if the physical computer rack running your EC2 compute node encounters an unrecoverable hardware failure, the EBS disk remains 100% safe and intact. It can be instantly detached and mounted to a brand-new instance.
*   **Mounting Boundaries:** By default, standard EBS volumes operate on a strict **1-to-1 instance mapping ratio** [3.1]. However, premium Provisioned IOPS tiers (`io1`/`io2`) support **EBS Multi-Attach**, allowing up to 16 separate instances in the same Availability Zone to read and write to the same raw block drive simultaneously.
*   **The Termination Flag Trap:** During instance configuration, the `Delete on Termination` property is active by default for the Root OS volume. If you explicitly terminate a server, AWS purges the drive to prevent idle resource billing. Turning this flag off enforces total persistence independent of the compute lifecycle.

### B. EC2 Instance Store - Ephemeral Host Storage
Unlike EBS, an Instance Store consists of physical NVMe/SSD disks physically bolted directly onto the motherboard of the host hardware rack [2.1].
*   **Performance:** Delivers the absolute lowest disk latency and highest hardware-level IOPS since data does not cross the network layout.
*   **The Risk Factor:** It is **entirely ephemeral**. If the host hardware fails, or if the instance is stopped or terminated, the storage blocks lose power and all data is permanently vaporized. It is used strictly for temporary caches or scratch workloads.

---

## 📸 2. Snapshot Delta Engineering & AMI Composition

To automate disaster recovery and mass server deployments, architects transform persistent block arrays into reusable system templates [4.1].

### A. Incremental Block-Level Snapshots
An EBS Snapshot is a point-in-time copy of a virtual hard drive stored securely inside Amazon S3. 
*   **Block-Level Delta Storage:** Subsequent snapshots do not fully duplicate your data. Instead, they are inherently **incremental**. The first snapshot copies all blocks. The second snapshot only tracks and saves the exact raw data blocks that have changed (deltas) since the prior timestamp, maximizing storage efficiency and slashing account costs.

### B. Amazon Machine Image (AMI) Architecture
An AMI is a bootable, pre-configured software blueprint used to clone and scale out identical virtual computers instantly [4.1]. 

An AMI is fundamentally **a snapshot with a system brain attached** [4.1]. It is constructed out of three distinct architectural parts:
1.  **A Base Root Snapshot:** A point-in-time copy of the Operating System configuration and files (e.g., Ubuntu Linux with your web server pre-installed) [4.1].
2.  **Block Device Mapping:** Metadata dictating exactly what size and type of EBS volumes should be automatically generated and plugged into the server when it is born.
3.  **Launch Permissions:** Control plane rules determining which AWS account numbers or groups are legally allowed to boot instances using this specific template.

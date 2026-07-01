# M.Set-NAS-system
True NAS System Build 
Problem Statement
Managing personal data across multiple household members revealed the same challenges that regulated organizations face at scale: rising subscription costs, third-party data dependency, limited control over access policies, and vulnerability to vendor-driven terms changes.
Each family member maintained individual cloud storage accounts through their mobile carriers. As storage capacity limits were reached across multiple accounts, the default solution — increasing monthly subscriptions per user — represented an ongoing, compounding cost with no ownership benefit. Commercial providers retain the right to modify pricing, access terms, and data policies at any time, creating a vendor lock-in risk that mirrors the third-party data dependency challenges commonly faced by government-funded nonprofits managing sensitive client information.
Additional concerns drove the decision to build a self-hosted solution:

Data sovereignty — commercial cloud storage providers retain access to stored content, creating privacy and security exposure for personal media and documents
Cyber attack surface — centralized commercial platforms are high-value targets; self-hosted storage with zero-trust remote access reduces exposure
Cost predictability — a one-time hardware investment with no recurring per-user subscription fees provides long-term cost control, replacing variable monthly spend with a fixed capital expenditure model
Access continuity — self-hosted infrastructure eliminates the risk of account suspension or lockout by a third-party provider

This project applies the same data governance principles used in highly regulated social services environments — controlled access, cost optimization, data ownership, and secure remote availability — to a private multi-user storage infrastructure.

Architecture Decisions
Dell Precision 3630 — minimum power consumption reducing ongoing monthly electricity cost
TrueNAS SCALE — free, open source, Linux-based, Docker support, enterprise ZFS file system, no licensing cost
ZFS Mirror — data protection against drive failure, accepting 50% capacity tradeoff in exchange for zero data loss risk
Tailscale — encrypted zero-trust tunneling, no open ports, reduced internet exposure surface
Nextcloud — self-hosted private cloud, multi-user accounts, mobile auto-sync, one-time setup cost with no recurring per-user fees
rclone + Azure Blob — offsite protection against physical threats (fire, theft, ransomware) that local redundancy cannot address, reinforces Azure certification track
2.5GbE networking — faster transfer speeds for multi-user simultaneous access, justified by minimal cost difference over 1GbE


Ready for me to draft Section 2 — Architecture Decisions from these answers?YesHere's the drafted Architecture Decisions section:

Architecture Decisions
Every component in this build was selected based on three criteria: data protection, cost efficiency, and transferability to enterprise and regulated-organization environments.

Hardware — Dell Precision 3630 Tower
The Dell Precision 3630 was selected as the host machine primarily for its power consumption profile. At approximately 30-50W idle, it operates significantly below workstation-class alternatives that draw 100W or more at idle. For a system running continuously, this difference translates directly into measurable monthly electricity cost savings over the lifetime of the device — a FinOps-style total cost of ownership decision applied to on-premises infrastructure.

Operating System — TrueNAS SCALE
TrueNAS SCALE was selected over alternatives such as Unraid for the following reasons:

No licensing cost — fully free and open source with no per-seat or subscription fees
Linux foundation — skills developed administering TrueNAS SCALE transfer directly to cloud and enterprise Linux environments
Docker support — enables containerized applications including Nextcloud to run natively on the same hardware
ZFS file system — enterprise-grade storage with built-in data integrity verification and snapshot capabilities
Enterprise pedigree — TrueNAS is used in production business environments, not exclusively by home users


Storage — ZFS Mirror Pool (2x WD Red Pro 14TB)
Storage is configured as a ZFS mirror pool across two WD Red Pro 14TB drives. This configuration writes identical data to both drives simultaneously, providing complete redundancy against drive failure.
The capacity tradeoff is deliberate — 14TB of usable storage from 28TB of raw capacity was accepted in exchange for zero data loss risk on single drive failure. In the event of a drive failure the system continues operating normally from the surviving drive while the failed drive is replaced and the mirror rebuilt automatically.
WD Red Pro drives were selected specifically for their NAS optimization — designed for continuous operation, higher workload ratings, and vibration compensation in multi-drive enclosures.
This decision mirrors the data protection requirements common in regulated nonprofit environments where client records and grant documentation cannot be recreated if lost.

Remote Access — Tailscale
Remote access is handled exclusively through Tailscale rather than traditional port forwarding or VPN configuration. Tailscale implements a zero-trust networking model — devices must be authenticated before they can access the network, and no ports are exposed to the public internet.
This approach directly addresses two key concerns:

Reduced attack surface — nothing is visible or reachable from the internet without prior authentication
Encrypted tunneling — all traffic between devices is encrypted end to end

This zero-trust model mirrors the network security requirements common in government-funded social services organizations managing sensitive client data.

Private Cloud Layer — Nextcloud
Nextcloud provides the user-facing private cloud storage interface — functionally equivalent to Google Drive or iCloud but entirely self-hosted. Key selection criteria:

Data sovereignty — all files remain on hardware we own and control
Multi-user support — each family member has an individual account with isolated storage
Mobile auto-sync — phones automatically back up photos and files without manual intervention
One-time setup cost — eliminates recurring per-user monthly subscription fees across multiple family members
Runs as a Docker container on TrueNAS SCALE — no additional hardware required


Offsite Backup — rclone + Azure Blob Storage
Local redundancy through ZFS mirroring protects against drive failure but does not protect against physical threats — fire, flood, theft, or ransomware encrypting all local data simultaneously. Offsite replication to Azure Blob Storage addresses these scenarios.
rclone automates encrypted backup transfers from the NAS to Azure Blob on a scheduled basis. Azure Blob was selected over AWS S3 for alignment with the Azure certification roadmap (AZ-900 → AZ-104 → AZ-500), providing direct hands-on experience with Azure storage services that appear in certification exam objectives.
This hybrid architecture — on-premises primary storage with cloud offsite replication — mirrors the data backup and retention compliance requirements common in regulated nonprofit organizations managing grant-funded program data.

Networking — 2.5GbE
The network infrastructure was upgraded from standard 1GbE to 2.5GbE throughout — TP-Link TX201 NIC in the host machine, NICGIGA 8-port fanless switch, and SABRENT USB-C to 2.5GbE adapters on client laptops.
At 2.5 Gigabit speeds file transfer rates reach approximately 312 MB/s compared to 125 MB/s at 1GbE — a 2.5x improvement that prevents bottlenecks when multiple users access the NAS simultaneously for photo backup, video streaming, or document sync.
The cost difference between 1GbE and 2.5GbE hardware at the time of purchase was minimal relative to the performance gain — a cost-benefit evaluation consistent with FinOps principles of optimizing spend relative to demonstrated value.


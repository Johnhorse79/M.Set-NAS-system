Private Cloud NAS — Self-Hosted Family Storage System
A self-hosted private cloud storage system built for a multi-user household using enterprise-grade components and compliance-focused architecture. This project applies the same data governance principles used in highly regulated social services environments — controlled access, cost optimization, data ownership, and secure remote availability — to a private multi-user storage infrastructure.

The Problem
Managing personal data across multiple family members revealed the same challenges that regulated organizations face at scale:

Rising subscription costs across multiple per-user cloud storage accounts
Third-party data dependency — commercial providers control pricing, access terms, and data policies
No ownership of the underlying infrastructure
Vendor lock-in risk — accounts can be suspended, prices increased, or terms changed at any time

Rather than continuing to pay recurring fees to commercial providers for storage on hardware we don't own, this project builds a private cloud storage system on owned hardware — eliminating recurring per-user costs while maintaining full data sovereignty.

Solution Overview
A self-hosted NAS running TrueNAS SCALE with Nextcloud for private cloud storage, Tailscale for secure zero-trust remote access, and Azure Blob Storage for offsite backup — all running on a single low-power tower workstation.

Tech Stack
ComponentTechnologyPurposeHardwareDell Precision 3630 TowerHost machine — low power, enterprise gradeOSTrueNAS SCALE 25.10NAS operating systemStorageZFS Mirror Pool — 2x WD Red Pro 14TBRedundant data storagePrivate CloudNextcloud 34.0.1File storage and phone auto-syncRemote AccessTailscaleZero-trust encrypted remote accessOffsite Backuprclone + Azure Blob StorageDisaster recovery backupNetworkingTP-Link TX201 2.5GbE NICHigh speed local network

Key Features

Private cloud storage — Nextcloud provides a Google Drive / iCloud equivalent running entirely on owned hardware
Phone auto-sync — iPhone and Android camera rolls back up automatically
Remote access from anywhere — Tailscale enables encrypted access from any network without port forwarding
Offsite backup — rclone replicates all data to Azure Blob Storage protecting against physical threats
Multi-user — individual accounts with isolated storage quotas for each family member
Data sovereignty — no data leaves your hardware without your explicit action
Zero recurring per-user fees — one-time hardware investment replaces monthly subscription costs

Hardware Specifications
ComponentModelSpecTowerDell Precision 3630i7-8700, 32GB RAM, 500GB SSDStorage drivesWD Red Pro 14TB x2ZFS mirror poolNICTP-Link TX2012.5GbE PCIeSwitchNICGIGA 8-port2.5GbE fanlessUPSCyberPower ST425Battery backup

Idle power consumption: ~30-50W — significantly below workstation-class alternatives at 100W+ idle. Selected specifically for 24/7 operation cost efficiency.

Project Status
ComponentStatusHardware assembly✅ CompleteTrueNAS SCALE installation✅ CompleteStatic IP configuration✅ CompleteZFS storage pool✅ Single drive (mirror pending replacement drive)Nextcloud✅ RunningPhone auto-sync✅ ActiveTailscale remote access✅ ActiveAzure Blob backup✅ ConfiguredAutomated backup schedule⬜ PendingZFS mirror completion⬜ Waiting for replacement drive

Documentation
SectionDescriptionProblem StatementWhy this project was builtArchitecture DecisionsComponent selection reasoningImplementationStep by step build processTroubleshootingChallenges encountered and solutionsLessons LearnedWhat worked, what didn't, what's next

Professional Context
This project was built as part of an active career transition into cloud infrastructure and FinOps. The architecture decisions — zero-trust remote access, hybrid cloud backup, cost-optimized hardware selection, and compliance-focused data governance — directly reflect the principles applied in regulated nonprofit and government-funded social services environments.
Every component was chosen with a deliberate cost-benefit rationale. Every problem was diagnosed methodically. Every solution was documented. This is not a tutorial reproduction — it is a real system built on real hardware solving a real problem, with real failures along the way.

Architecture section:
Family Devices (iPhone, Android, Laptop)
        ↓
Tailscale (encrypted zero-trust tunnel)
        ↓
Dell Precision 3630 — TrueNAS SCALE
[YOUR_LOCAL_IP]
        ↓
Nextcloud [YOUR_LOCAL_IP:30027]

Author
Built and documented by a nonprofit management professional with 20 years of experience in government-funded, heavily regulated social services in New York City — transitioning into cloud infrastructure and FinOps with a focus on regulated industries.

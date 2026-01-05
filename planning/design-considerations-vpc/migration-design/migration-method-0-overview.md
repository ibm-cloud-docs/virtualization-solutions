---

copyright:
  years: 2025
lastupdated: "2026-01-05"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Migration Methods
{: #virt-sol-vpc-migration-design-methods}

There are four primary methods for migrating VMware VMs to VPC VSI, each with distinct design implications, advantages, and constraints. Your choice depends on VM characteristics, source environment access, team expertise, and scale requirements.

* [Method 1: Image Import (Template-Based Migration)](migration-method-1.md)
* [Method 2: Direct Volume Copy (Multi-Disk Migration)](migration-method-2.md)
* [Method 3: Live Network Transfer (Recommended for Scale)](migration-method-3.md)
* [Method 4: VDDK Direct Extraction (vCenter Only)](migration-method-4.md)

## Migration Method Selection Matrix
{: #virt-sol-vpc-migration-design-methods-matrix}

| Criterion | Method 1 (Image) | Method 2 (Volume) | Method 3 (Network) | Method 4 (VDDK) |
| ----------- | ------------------ | ------------------- | -------------------- | -------------------- |
| **Single-disk VM** | ✓ Ideal | ✓ Works | ✓ Works | ✓ Works |
| **Multi-disk VM** | ✗ Partial only | ✓ Ideal | ✓ Ideal | ✓ Ideal |
| **VCFaaS source** | ✓ | ✓ | ✓ | ✗ vCenter only |
| **vCenter source** | ✓ | ✓ | ✓ | ✓ |
| **Avoids export overhead** | ✗ | Partial | ✓ | ✓ |
| **Windows driver injection** | Manual | ✓ (virt-v2v) | ✓ (virt-v2v) | ✓ Built-in |
| **Parallel migration** | ✓ | ✓ | ✓ Excellent | ✓ |
| **Setup complexity** | Low | Medium | Medium-High | High |
| **Per-VM complexity** | Low | Medium | Medium | Low (once setup) |
| **Automation potential** | Medium | High | High | Very High |
| **Template reuse** | ✓ Excellent | Limited | Limited | Limited |
| **Image proliferation** | ✗ High | ✓ Avoids | ✓ Avoids | ✓ Avoids |
{: caption="Migration Method Selection Matrix" caption-side="bottom"}

**General Guidance**:
- **Small migrations (<10 VMs), mostly single-disk**: Method 1
- **Medium migrations (10-50 VMs), mix of configurations**: Method 2 or 3
- **Large migrations (50+ VMs), efficiency critical**: Method 3
- **vCenter environment, automation-first approach**: Method 4
- **VCFaaS environment**: Methods 1, 2, or 3

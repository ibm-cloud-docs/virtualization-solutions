---

copyright:
  years: 2025
lastupdated: "2026-10-06"

keywords: IBM Cloud VPC migration methods, VMware VM migration comparison, image import migration VPC, direct volume copy migration, live network transfer VPC, VDDK direct extraction, virt-v2v migration methods, VMware to VPC migration technique, migration method selection IBM Cloud, compare VMware migration approaches

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# {{site.data.keyword.cloud_notm}} VPC migration methods for VMware VMs
{: #virt-sol-vpc-migration-design-methods}

Compare {{site.data.keyword.cloud_notm}} VPC migration methods for VMware VMs: Image Import, Direct Volume Copy, Live Network Transfer, and VDDK direct extraction from vCenter.
{: shortdesc}

* Method 1: [Image Import (Template-Based Migration)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method1)
* Method 2: [Copying direct volume (multi-disk method)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method2)
* Method 3: [Live Network Transfer (Recommended for Scale)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method3)
* Method 4: [VDDK Direct Extraction (vCenter Only)](/docs/virtualization-solutions?topic=virtualization-solutions-virt-sol-vpc-migration-design-method4)

## Migration method selection matrix
{: #virt-sol-vpc-migration-design-methods-matrix}

Use the following comparison matrix to evaluate migration methods against your workload criteria, tooling preferences, and architectural constraints:

| Criterion | Method 1 (Image) | Method 2 (Volume) | Method 3 (Network) | Method 4 (VDDK) |
| ----------- | ------------------ | ------------------- | -------------------- | -------------------- |
| Single-disk virtual machine | ✓ Ideal | ✓ Works | ✓ Works | ✓ Works |
| Multi-disk virtual machine | ✗ Partial only | ✓ Ideal | ✓ Ideal | ✓ Ideal |
| VCFaaS source | ✓ | ✓ | ✓ | ✗ vCenter only |
| vCenter source | ✓ | ✓ | ✓ | ✓ |
| Avoids export overhead | ✗ | Partial | ✓ | ✓ |
| Windows driver injection | Manual | ✓ (virt-v2v) | ✓ (virt-v2v) | ✓ Built-in |
| Parallel migration | ✓ | ✓ | ✓ Excellent | ✓ |
| Setup complexity | Low | Medium | Medium-High | High |
| Per-virtual machine complexity | Low | Medium | Medium | Low (once set up) |
| Automation potential | Medium | High | High | Very High |
| Template reuse | ✓ Excellent | Limited | Limited | Limited |
| Image proliferation | ✗ High | ✓ Avoids | ✓ Avoids | ✓ Avoids |
{: caption="Migration method selection matrix" caption-side="bottom"}

General Guidance:
- Small migrations (<10 virtual machines), mostly single-disk: Method 1
- Medium migrations (10-50 virtual machines), mix of configurations: Method 2 or 3
- Large migrations (50+ virtual machines), efficiency critical: Method 3
- vCenter environment, automation-first approach: Method 4
- VCFaaS environment: Methods 1, 2, or 3

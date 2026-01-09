---

copyright:
  years: 2025
lastupdated: "2026-01-09"

keywords: VSI, File Storage, Block Storage, Encryption, Migration

subcollection: virtualization-solutions

---

{{site.data.keyword.attribute-definition-list}}

# Governance, Documentation, and Lessons Learned
{: #virt-sol-vpc-migration-design-governance}

This section provides some guidance on how to place some governance around the migration project.

## Change Management Integration
{: #virt-sol-vpc-migration-design-governance-change}

**Change Request Process**:
- Create change request for each wave
- Include: wave scope, VMs, applications, owners, cutover window
- Obtain approvals per organizational change process
- Link to detailed runbook

**Communication Plan**:
- T-7 days: Initial notification to application owners and users
- T-3 days: Reminder with specifics (downtime window, expected impact)
- T-1 day: Final reminder and confirmation of readiness
- T-0 (start of cutover): Maintenance window begins notification
- T+completion: Success notification and any follow-up actions
- T+24 hours: Post-migration status report

**Stakeholder Engagement**:
- Application owners: Responsible for application-specific validation
- Network team: Ensure Transit Gateway and VPC networking is ready
- Security team: Verify security group rules and encryption
- Operations team: Update monitoring, backup, and runbooks for VPC environment

## Documentation Requirements
{: #virt-sol-vpc-migration-design-governance-documentation}

**Architecture Documentation**:
- Network topology diagrams (VMware to VPC via Transit Gateway)
- VPC subnet design and IP addressing
- Security group architecture and rule sets
- Storage profile selections and rationale

**Runbook Templates**:

Create method-specific runbooks with exact commands and checklists. Example structure:

```markdown
# Migration Runbook: Method 3 (Live Network Transfer)

## Pre-Migration Checklist
- [ ] Transit Gateway connectivity verified
- [ ] Worker VSI provisioned (instance ID: _______)
- [ ] Target volumes created and attached
- [ ] Live ISO prepared and tested
- [ ] Source VM backed up/snapshotted

## Execution Steps

### 1. Configure Worker VSI Listener
**Person**: Migration Engineer
**Expected Duration**: 5 minutes
**Commands**:
    ```bash
    ssh worker-vsi
    nc -l 192.168.100.5 8080 | gunzip | dd of=/dev/vdb bs=16M status=progress
    ```
**Validation**: Listener process running, waiting for connection

### 2. Boot Source VM from ISO
**Person**: VMware Administrator  
**Expected Duration**: 5 minutes
**Steps**:
1. Attach ubuntu-live.iso to VM in vCenter
2. Reboot VM
3. Select "Try Ubuntu" option
4. Open terminal

**Validation**: Shell prompt accessible, can see disk devices

[...continue for each step...]

## Rollback Procedure
[Detailed steps for this wave]

## Post-Migration Validation
[Checklist specific to applications in this wave]
```

**Migration Documentation**:

Maintain a spreadsheet or database tracking:
- VM name
- VMware details (vCenter, cluster, datastore, IP, disks)
- VPC details (VSI ID, instance profile, storage profiles, new IP if re-IPed)
- Migration method used
- Wave assignment
- Migration status (not started, in progress, complete, rollback)
- Issues encountered and resolution
- Validation status

**Troubleshooting Guides**:

Document common issues and resolutions discovered during pilot and early waves:

| Issue | Symptoms | Resolution |
| ------- | ---------- | ------------ |
| Windows won't boot | INACCESSIBLE_BOOT_DEVICE | VirtIO SCSI driver not installed, retry with virt-v2v |
| Linux network not working | No IP on interface | Interface name changed, update config to new name |
| virt-v2v fails "unclean shutdown" | virt-v2v error message | Windows fast startup enabled, reboot Windows cleanly |
| VPC console shows "No bootable device" | Blank screen, no OS | GPT backup partition table issue, recreate with gdisk |
{: caption="Troubleshooting Guides" caption-side="bottom"}

## Lessons Learned Capture
{: #virt-sol-vpc-migration-design-governance-lessons}

After pilot wave and after completion of all waves, conduct structured retrospectives.

**Pilot Wave Retrospective** (T+7 days after pilot):

**What went well**:
- Which method worked best?
- Were time estimates accurate?
- Did rollback procedures work?

**What didn't go well**:
- Which issues were unexpected?
- Where did we waste time?
- What documentation was missing or unclear?

**Adjustments for subsequent waves**:
- Refine runbooks based on actual experience
- Update time estimates
- Add troubleshooting steps for new issues
- Adjust wave sizes if too large or too small

**Final Project Retrospective** (T+30 days after last wave):

**Overall metrics**:
- Total VMs migrated
- Total time invested
- Average time per VM by method
- Success rate (first-time boots, no rollbacks needed)
- Issues encountered and time to resolve

**Method effectiveness comparison**:
- For your specific environment, which method was most efficient?
- Would you use the same methods for future migrations?

**Recommendations for similar projects**:
- What would you do differently?
- What tools or automation would you build?
- What skills were most valuable?

**Artifacts to preserve**:
- Final runbooks (useful for future cloud migrations)
- Automation scripts (Terraform, Ansible, shell scripts)
- Troubleshooting guides
- Training materials for operations teams

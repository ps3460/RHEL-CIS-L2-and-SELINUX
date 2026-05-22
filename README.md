# RHEL 9 CIS Level 2 and SELinux Enforcing

Date: 2026-05-22

This repository is not maintained.

This repository contains generic Ansible automation for applying CIS Level 2 hardening to Red Hat Enterprise Linux 9 with OpenSCAP, then setting SELinux to enforcing mode and collecting basic evidence.

Review and test this automation before using it. CIS remediation can affect SSH access, sudo behavior, firewall rules, crypto policy, running services, package workflows, and application deployment.

## Contents

- `playbooks/bootstrap-ansible-user.yml`: optional bootstrap for a non-root automation user.
- `playbooks/rhel9-cis-l2.yml`: applies the RHEL 9 CIS Level 2 role.
- `playbooks/selinux-enforcing.yml`: sets and verifies SELinux enforcing mode.
- `roles/rhel9_cis_l2`: OpenSCAP CIS Level 2 scan, remediation, reboot, post-scan, and report fetch.
- `inventories/example/hosts.yml`: example inventory structure.

## Usage

Update `inventories/example/hosts.yml` for your target hosts and SSH settings.

Optional bootstrap:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/bootstrap-ansible-user.yml \
  -e rhel9_bootstrap_public_key_path="$HOME/.ssh/id_ed25519.pub"
```

Apply CIS Level 2:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/rhel9-cis-l2.yml
```

Set and verify SELinux enforcing:

```bash
ansible-playbook -i inventories/example/hosts.yml playbooks/selinux-enforcing.yml
```

## Variables

### CIS Level 2

- `rhel9_cis_l2_profile_candidates`: ordered OpenSCAP profile IDs to try.
- `rhel9_cis_l2_datastream_candidates`: ordered SCAP datastream paths to try.
- `rhel9_cis_l2_report_root`: report directory on the managed host.
- `rhel9_cis_l2_controller_report_root`: fetched report directory on the controller.
- `rhel9_cis_l2_remediate`: whether to run OpenSCAP remediation.
- `rhel9_cis_l2_reboot_after_remediation`: whether to reboot after remediation.
- `rhel9_cis_l2_preserve_ansible_user`: whether to restore automation SSH and sudo access after remediation.
- `rhel9_cis_l2_ansible_user`: automation user to preserve.
- `rhel9_cis_l2_public_key_path`: public key installed for the automation user when preservation is enabled.
- `rhel9_cis_l2_ansible_sudo_nopasswd`: whether the preserved sudo rule is passwordless.
- `rhel9_cis_l2_ssh_allow_users`: optional `AllowUsers` list written to the automation SSH drop-in.
- `rhel9_cis_l2_allowed_distributions`: allowed Ansible distribution names, defaulting to `RedHat`.
- `rhel9_cis_l2_allowed_host_regex`: optional managed-host guardrail regex. Leave empty to disable.

### SELinux Enforcing

- `selinux_enforcing_evidence_root`: fetched SELinux evidence directory on the controller.
- `selinux_enforcing_force_relabel`: whether to create `/.autorelabel` before the enforcing reboot.
- `selinux_enforcing_allowed_failed_services`: failed systemd services to ignore during verification.
- `selinux_enforcing_reboot_timeout`: reboot timeout in seconds.

### Bootstrap

- `rhel9_bootstrap_user`: automation user to create.
- `rhel9_bootstrap_public_key_path`: controller public key to install.
- `rhel9_bootstrap_sudo_nopasswd`: whether bootstrap sudo is passwordless.
- `rhel9_bootstrap_allowed_distributions`: allowed Ansible distribution names, defaulting to `RedHat`.

## Reports

The CIS role generates and fetches:

- pre-remediation ARF and HTML reports
- remediation ARF and HTML reports
- post-remediation ARF and HTML reports

The SELinux playbook writes:

- SELinux state evidence
- service state evidence
- AVC audit evidence

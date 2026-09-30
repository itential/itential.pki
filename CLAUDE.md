# CLAUDE.md -- itential.tls

This file provides guidance to Claude Code when working in this Ansible collection.

## What This Collection Is

`itential.tls` (v1.0.1) is an Ansible collection for TLS certificate lifecycle management.
It generates and deploys CA certificates and signed leaf certificates to target hosts using
`community.crypto`. All cryptographic operations run on the control node (`delegate_to: localhost`);
only the upload playbooks push files to remote hosts.

Two certificate workflows are supported:

- **Per-host** -- each inventory host gets a unique key and certificate with SANs derived
  from that host's identity (hostname, FQDN, IP).
- **Multi-domain** -- a single certificate covering all inventory hosts is generated and
  deployed identically to every host.

## Commands

### Install dependencies

```bash
ansible-galaxy collection install -r requirements.yml   # community.crypto >= 2.17.0
ansible-galaxy collection install . --force             # Install this collection locally
```

### Run the playbooks

```bash
# Per-host workflow (single command)
ansible-playbook -i <inventory> playbooks/tls_lifecycle.yml

# Per-host workflow (run in order)
ansible-playbook -i <inventory> playbooks/gen_ca_cert.yml
ansible-playbook -i <inventory> playbooks/gen_certs.yml
ansible-playbook -i <inventory> playbooks/upload_ca_cert.yml
ansible-playbook -i <inventory> playbooks/upload_certs.yml

# Multi-domain workflow (run in order)
ansible-playbook -i <inventory> playbooks/gen_ca_cert.yml
ansible-playbook -i <inventory> playbooks/gen_multi_domain_cert.yml
ansible-playbook -i <inventory> playbooks/upload_ca_cert.yml
ansible-playbook -i <inventory> playbooks/upload_multi_domain_cert.yml
```

## Repository Layout

```
playbooks/
  tls_lifecycle.yml             # End-to-end per-host workflow (CA gen + cert gen + upload)
  gen_ca_cert.yml               # Generate CA on localhost
  gen_certs.yml                 # Generate per-host certs on localhost
  gen_multi_domain_cert.yml     # Generate shared multi-SAN cert on localhost
  upload_ca_cert.yml            # Deploy CA cert to all hosts (become)
  upload_certs.yml              # Deploy per-host certs to each host (become)
  upload_multi_domain_cert.yml  # Deploy shared cert to all hosts (become)
roles/
  tls/
    defaults/main.yml           # All overridable variables
    tasks/
      validate_vars.yml         # Assert tls_pki_local_dir is defined and exists
      create_directory.yml      # Create one dir (SELinux-aware)
      create_cert_directories.yml  # Loop over cert/key dirs
      gen_ca_cert.yml           # 4096-bit RSA CA key + self-signed cert
      gen_certs.yml             # Per-host 2048-bit RSA key + CSR + cert + PEM
      gen_multi_domain_cert.yml # Shared 2048-bit RSA key + multi-SAN cert + PEM
      upload_ca_cert.yml        # Copy CA cert to remote (0444)
      upload_certs.yml          # Copy per-host key (0600) and cert (0444) to remote
      upload_multi_domain_cert.yml  # Copy shared key (0600) and cert (0444) to remote
meta/
  runtime.yml                   # Requires Ansible >= 2.15.0
galaxy.yml                      # Collection metadata and dependencies
```

## Key Variables

All variables live in `roles/tls/defaults/main.yml`.

| Variable | Default | Notes |
|---|---|---|
| `tls_pki_local_dir` | (undefined) | **Required.** Control-node path for all generated PKI files. |
| `tls_pki_ca_certs_dir` | `/etc/pki/ca-trust/source/anchors` | Remote CA cert directory |
| `tls_pki_certs_dir` | `/etc/pki/tls/certs` | Remote cert directory |
| `tls_pki_keys_dir` | `/etc/pki/tls/private` | Remote key directory |
| `tls_own_ca` | `true` | `true` = CA-signed; `false` = self-signed |
| `tls_ca_common_name` | `Itential CA` | CA certificate CN |
| `tls_csr_common_name` | `Itential` | Leaf cert CN |
| `tls_csr_country_name` | `US` | |
| `tls_csr_organization_name` | `Itential` | |
| `tls_csr_organizational_unit_name` | `IT` | |
| `tls_csr_state_or_province_name` | `Georgia` | |
| `tls_csr_locality_name` | `Atlanta` | |
| `tls_pki_ca_filename` | `ca` | Base name for CA files (`ca.key`, `ca.crt`) |
| `tls_pki_cert_filename` | `{{ inventory_hostname }}` | Base name for per-host files |
| `tls_pki_md_cert_filename` | `itential` | Base name for multi-domain files |
| `tls_additional_dns_sans` | `[]` | Extra DNS SANs appended to every cert |
| `tls_additional_ip_sans` | `[]` | Extra IP SANs appended to every cert |
| `tls_extended_key_usage` | `[serverAuth, clientAuth]` | Extended Key Usage applied to per-host and multi-domain leaf certs |

## Certificate Details

### CA certificate

- 4096-bit RSA key
- Basic constraint: `CA:TRUE`
- Key usage: `keyCertSign`, `cRLSign`
- Stored locally at `{{ tls_pki_local_dir }}/ca.key` and `ca.crt`

### Per-host leaf certificate

- 2048-bit RSA key
- Key usage: `digitalSignature`, `keyEncipherment`
- Extended key usage: `tls_extended_key_usage` (default `serverAuth`, `clientAuth`)
- SANs auto-built from: `inventory_hostname`, `ansible_fqdn`, `ansible_host`,
  `ansible_default_ipv4.address`, plus any `tls_additional_*_sans` entries
- Files: `<hostname>.key`, `<hostname>.csr`, `<hostname>.crt`, `<hostname>.pem`
- `.pem` is key + cert concatenated

### Multi-domain certificate

- 2048-bit RSA key
- Key usage: `digitalSignature`, `keyEncipherment`
- Extended key usage: `tls_extended_key_usage` (default `serverAuth`, `clientAuth`)
- SANs aggregated from ALL inventory hosts
- Validity: `+365d` (explicit)
- Files: `itential.key`, `itential.csr`, `itential.crt`, `itential.pem`

## Playbook Structure

Playbooks are thin wrappers that call `import_role: name: itential.tls.tls tasks_from: <task_file>`.

- `tls_lifecycle.yml` runs the full per-host workflow (CA gen, cert gen, CA upload, cert upload)
  as four sequential plays in one invocation.
- `gen_*` playbooks target `localhost`, `gather_facts: false` (except `gen_certs.yml`
  and `gen_multi_domain_cert.yml` which need facts from `all` first to build SANs).
- `upload_*` playbooks target `all`, `become: true`.

## Requirements

- Ansible >= 2.15.0
- `community.crypto >= 2.17.0`
- `tls_pki_local_dir` must exist on the control node before any playbook runs
- Upload playbooks require sudo on target hosts
- SELinux: automatically detected; `cert_t` context applied when active

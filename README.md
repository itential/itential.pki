# itential.tls Ansible Collection

> Generate and distribute TLS certificates across your infrastructure using a local PKI workflow.

---

## Table of Contents

- [Overview](#overview)
- [Installation](#installation)
- [Playbooks](#playbooks)
- [Variables Reference](#variables-reference)
  - [Required](#required)
  - [Remote Directories](#remote-directories)
  - [Certificate Subject (DN)](#certificate-subject-dn)
  - [CA Signing](#ca-signing)
  - [Subject Alternative Names](#subject-alternative-names)
  - [File Suffixes](#file-suffixes)
  - [CA File Variables](#ca-file-variables)
  - [Per-Host Certificate Variables](#per-host-certificate-variables)
  - [Multi-Domain Certificate Variables](#multi-domain-certificate-variables)
- [Certificate Configuration](#certificate-configuration)
- [Examples](#examples)
- [Notes & Caveats](#notes--caveats)

---

## Overview

This collection manages the full TLS certificate lifecycle:

1. Generate a local CA
2. Generate per-host or multi-domain certificates signed by that CA
3. Upload the CA cert and leaf certs to remote hosts

All certificate generation runs on the **Ansible control node**. Only the upload playbooks touch remote hosts.

## Installation

Clone the repo to your control node.
Build the itential-tls collection to create the tarball.
```bash
ansible-galaxy collection buildi
```
Install the collection. Make sure your collections path is set appropriately.
```bash
ansible-galaxy collection install itential-tls-<VERSION>.tar.gz
```

---

## Playbooks

| Playbook | Runs On | Description |
|---|---|---|
| `itential.tls.gen_ca_cert` | Local | Generate a CA private key and self-signed CA certificate |
| `itential.tls.gen_certs` | Local → Remote | Generate per-host key, CSR, and certificate signed by the CA |
| `itential.tls.gen_multi_domain_cert` | Local | Generate a single multi-domain cert covering all inventory hosts |
| `itential.tls.upload_ca_cert` | Remote | Upload the CA certificate to all target hosts |
| `itential.tls.upload_certs` | Remote | Upload per-host key and certificate to each target host |
| `itential.tls.upload_multi_domain_cert` | Remote | Upload the shared multi-domain key and certificate to all hosts |

---

## Variables Reference

### Required

| Variable | Default | Description |
|---|---|---|
| `tls_pki_local_dir` | *(none — required)* | Absolute path on the control node where generated keys, CSRs, and certs are stored. Must exist before running any playbook. |

---

### Remote Directories

Directories on remote hosts where files are installed. Created automatically if they do not exist.

| Variable | Default | Description |
|---|---|---|
| `tls_pki_ca_certs_dir` | `/etc/pki/ca-trust/source/anchors` | Where the CA certificate is installed on remote hosts |
| `tls_pki_certs_dir` | `/etc/pki/tls/certs` | Where leaf/host certificates are installed on remote hosts |
| `tls_pki_keys_dir` | `/etc/pki/tls/private` | Where private keys are installed on remote hosts |

---

### Certificate Subject (DN)

Populate the Distinguished Name fields in generated CSRs and certificates.

| Variable | Default | Description |
|---|---|---|
| `tls_csr_common_name` | `Itential` | Common Name (CN) |
| `tls_csr_country_name` | `US` | Two-letter ISO 3166-1 country code |
| `tls_csr_organization_name` | `Itential` | Organization (O) |
| `tls_csr_organizational_unit_name` | `IT` | Organizational Unit (OU) |
| `tls_csr_state_or_province_name` | `Georgia` | State or Province (ST) |
| `tls_csr_locality_name` | `Atlanta` | City or Locality (L) |
| `tls_ca_common_name` | `Itential CA` | Common Name used specifically for the CA certificate |

---

### CA Signing

| Variable | Default | Description |
|---|---|---|
| `tls_own_ca` | `true` | When `true`, certificates are signed by the local CA (`ownca` provider). When `false`, certificates are self-signed. |

---

### Subject Alternative Names

Additional SANs are appended to the automatically-discovered list built from `inventory_hostname`, `ansible_fqdn`, and `ansible_host`.

| Variable | Default | Description |
|---|---|---|
| `tls_additional_dns_sans` | `[]` | Extra DNS names to include in the SAN extension |
| `tls_additional_ip_sans` | `[]` | Extra IP addresses to include in the SAN extension |

---

### File Suffixes

| Variable | Default | Description |
|---|---|---|
| `tls_pki_key_suffix` | `.key` | Extension appended to key filenames |
| `tls_pki_cert_suffix` | `.crt` | Extension appended to certificate filenames |
| `tls_pki_csr_suffix` | `.csr` | Extension appended to CSR filenames |

---

### CA File Variables

Override `tls_pki_ca_filename` to rename all CA files at once. All other CA file variables are derived from it.

| Variable | Default | Description |
|---|---|---|
| `tls_pki_ca_filename` | `ca` | Base filename for all CA files |
| `tls_pki_ca_key_file_local` | `{{ tls_pki_local_dir }}/ca.key` | Local path to the CA private key |
| `tls_pki_ca_csr_file_local` | `{{ tls_pki_local_dir }}/ca.csr` | Local path to the CA CSR |
| `tls_pki_ca_cert_file_local` | `{{ tls_pki_local_dir }}/ca.crt` | Local path to the CA certificate |
| `tls_pki_ca_cert_file_dest` | `{{ tls_pki_ca_certs_dir }}/ca.crt` | Remote path where the CA cert is installed |

---

### Per-Host Certificate Variables

The cert filename defaults to `inventory_hostname` so each host gets a uniquely-named certificate.

| Variable | Default | Description |
|---|---|---|
| `tls_pki_cert_filename` | `{{ inventory_hostname }}` | Base filename for this host's certificate files |
| `tls_pki_key_file_local` | `{{ tls_pki_local_dir }}/<hostname>.key` | Local path to the host private key |
| `tls_pki_key_file_dest` | `{{ tls_pki_keys_dir }}/<hostname>.key` | Remote path for the host private key |
| `tls_pki_csr_file_local` | `{{ tls_pki_local_dir }}/<hostname>.csr` | Local path to the host CSR |
| `tls_pki_cert_file_local` | `{{ tls_pki_local_dir }}/<hostname>.crt` | Local path to the host certificate |
| `tls_pki_cert_file_dest` | `{{ tls_pki_certs_dir }}/<hostname>.crt` | Remote path for the host certificate |

---

### Multi-Domain Certificate Variables

A single certificate shared across all inventory hosts.

| Variable | Default | Description |
|---|---|---|
| `tls_pki_md_cert_filename` | `itential` | Base filename for the multi-domain certificate files |
| `tls_pki_md_key_file_local` | `{{ tls_pki_local_dir }}/itential.key` | Local path to the multi-domain private key |
| `tls_pki_md_key_file_dest` | `{{ tls_pki_keys_dir }}/itential.key` | Remote path for the multi-domain private key |
| `tls_pki_md_csr_file_local` | `{{ tls_pki_local_dir }}/itential.csr` | Local path to the multi-domain CSR |
| `tls_pki_md_cert_file_local` | `{{ tls_pki_local_dir }}/itential.crt` | Local path to the multi-domain certificate |
| `tls_pki_md_cert_file_dest` | `{{ tls_pki_certs_dir }}/itential.crt` | Remote path for the multi-domain certificate |

---

## Certificate Configuration

### Key Parameters

| Parameter | Value | Notes |
|---|---|---|
| CA key size | 4096-bit RSA | Strong root for signing |
| Leaf key size | 2048-bit RSA | Per-host and multi-domain certs |
| CA basic constraint | `CA:TRUE` | Marks certificate as a CA |
| CA key usage | `keyCertSign`, `cRLSign` | Required CA extensions |
| Leaf key usage | `digitalSignature`, `keyEncipherment` | Standard TLS leaf usage (multi-domain) |
| Leaf extended key usage | `serverAuth`, `clientAuth` | Allows use for TLS server and client auth |
| CA validity | openssl default (~30 days) | No explicit `not_after` set — override if needed |
| Leaf validity | `+365d` | Multi-domain certs. Per-host uses provider default. |

### Default Certificate Subject

```
CN = Itential
C  = US
O  = Itential
OU = IT
ST = Georgia
L  = Atlanta
```

### SAN Auto-Discovery

The collection automatically builds a SAN list per certificate from:

- `DNS:inventory_hostname` — if not a bare IP
- `IP:inventory_hostname` — if a bare IP
- `DNS:ansible_fqdn` — when the fact is available
- `IP:` or `DNS:ansible_host` — detected automatically
- All values in `tls_additional_dns_sans` and `tls_additional_ip_sans`

For multi-domain certs, this runs for **every host** in the inventory, producing a single cert valid for all nodes.

---

## Examples

Create a local directory to store the certificates.

```bash
$ mkdir -p ~/itential/pki
```

Add tls_pki_local_dir to your inventory with the directory.
### Minimal Inventory (Default Paths)

```yaml
all:
  vars:
    tls_pki_local_dir: ~/itential/pki
  hosts:
    node1.example.com:
      ansible_host: 10.0.0.11
    node2.example.com:
      ansible_host: 10.0.0.12
```

```bash
ansible-playbook itential.tls.gen_ca_cert     -i inventory.yml
ansible-playbook itential.tls.gen_certs       -i inventory.yml
ansible-playbook itential.tls.upload_ca_cert  -i inventory.yml
ansible-playbook itential.tls.upload_certs    -i inventory.yml
```

---

### Custom Remote Directories

Use this when the default `/etc/pki` paths don't exist or you want a custom layout. The collection creates the directories if they are missing.

```yaml
all:
  vars:
    tls_pki_local_dir:     ~/itential/pki
    tls_pki_ca_certs_dir:  <custom-ca-dir>
    tls_pki_certs_dir:     <custom-certs-dir>
    tls_pki_keys_dir:      <custom-keys-dir>
  hosts:
    node1.example.com:
      ansible_host: 10.0.0.11
    node2.example.com:
      ansible_host: 10.0.0.12
```

---

### Custom Certificate Subject

```yaml
all:
  vars:
    tls_pki_local_dir:                ~/itential/pki
    tls_csr_common_name:              MyApp
    tls_csr_country_name:             US
    tls_csr_organization_name:        Acme Corp
    tls_csr_organizational_unit_name: Engineering
    tls_csr_state_or_province_name:   California
    tls_csr_locality_name:            San Francisco
    tls_ca_common_name:               Acme Internal CA
```

---

### Additional SANs

```yaml
all:
  vars:
    tls_pki_local_dir: ~/itential/pki
    tls_additional_dns_sans:
      - app.internal.example.com
      - api.internal.example.com
    tls_additional_ip_sans:
      - 10.0.100.5
      - 10.0.100.6
```

---

### Self-Signed Certificates (No CA)

```yaml
all:
  vars:
    tls_pki_local_dir: ~/itential/pki
    tls_own_ca: false
```

> **Note:** Most clients will not trust self-signed certificates without a manual trust import.

---

### Multi-Domain Certificate Workflow

A single cert valid for all hosts in the inventory — useful for shared services or load-balanced deployments.

```bash
# 1. Generate the CA (once)
ansible-playbook itential.tls.gen_ca_cert              -i inventory.yml

# 2. Generate the multi-domain cert (covers all hosts)
ansible-playbook itential.tls.gen_multi_domain_cert    -i inventory.yml

# 3. Upload CA cert to all hosts
ansible-playbook itential.tls.upload_ca_cert           -i inventory.yml

# 4. Upload the shared multi-domain cert to all hosts
ansible-playbook itential.tls.upload_multi_domain_cert -i inventory.yml
```

Override `tls_pki_md_cert_filename` to rename the shared cert files:

```yaml
tls_pki_md_cert_filename: myapp-shared
# Produces: myapp-shared.key, myapp-shared.csr, myapp-shared.crt
```

---

## Notes & Caveats

- **`tls_pki_local_dir` must exist** on the control node before running any playbook. The collection validates this at startup and fails with a clear error if missing.
- **Remote directories are created automatically** by the upload playbooks (`tls_pki_ca_certs_dir`, `tls_pki_certs_dir`, `tls_pki_keys_dir`).
- **The CA private key stays local.** It is never uploaded to remote hosts.
- **CA certificate validity is not explicitly set** in `gen_ca_cert`. The OpenSSL default is ~30 days. Add `selfsigned_not_after: "+10y"` to the self-sign task if a longer lifetime is needed.
- **Multi-domain certificates expire after 365 days** (`ownca_not_after: +365d`).
- **Requires the `community.crypto` Ansible collection** to be installed on the control node.

---

*Copyright © 2025, Itential, Inc — GNU General Public License v3.0+*

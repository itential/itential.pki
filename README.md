# itential.tls

## Description
This collection contains a playbook that generated TLS certificates.

## Installation
Clone the repo to your control node.

Build the itential-tls collection to create the tarball.

```ansible-galaxy collection build```

Install the collection. Make sure your collections path is set appropriately.

```ansible-galaxy collection install itential-tls-<VERSION>.tar.gz```

## Usage
Create a local directory to store the certificates.

Example:

```bash
$ mkdir -p ~/itential/pki
```

Add `tls_pki_local_dir` to your inventory with the directory.

Example:

```yaml
all:
  vars:
    tls_pki_local_dir: ~/itential/pki
```

To generate a CA certificate:
```
ansible-playbook itential.tls.gen_ca_cert -i <INVENTORY>
```

To generate a server certificate for each host:
```
ansible-playbook itential.tls.gen_certs -i <INVENTORY>
```

To generate a single multi-domain certificate:
```
ansible-playbook itential.tls.gen_multi_domain_cert -i <INVENTORY>
```

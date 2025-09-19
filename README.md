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
To generate a CA certificate for each host:
```
ansible-playbook itential.tls.gen_ca_cert -i <INVENTORY>
```

To generate a certificate for each host:
```
ansible-playbook itential.tls.gen_certs -i <INVENTORY>
```

To generate a single multi-domain certificate for all hosts:
```
ansible-playbook itential.tls.gen_multi_domain_cert -i <INVENTORY>
```

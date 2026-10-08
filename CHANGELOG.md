# Changelog

## v1.0.0 (October 06, 2026)

* Add EKU serverAuth and clientAuth to gen_certs
* Add claude.md file, add playbook for entire lifecycle
* Add configurable Extended Key Usage support for generated certs
* Add instructions for setting the certs directory in the inventory
* Add inventory_hostname to gen_certs
* Add support for additional SANs
* Add task to validate vars
* Add tasks to generate pem files (key + cert)
* Added SELinux support and modified readme for corrections
* Added missing copyright
* Added playbook to upload certs
* Added tasks to create dirs exist incase of customization & README changes
* Change default ca file name to 'ca'
* Document tls_lifecycle.yml in CLAUDE.md
* Fix no-changed-when ansible-lint findings
* Fix upload playbooks
* Initial version
* Reduced repetitive code
* Refactor default var names
* Rename collection from itential.tls to itential.pki
* Rename role variables from tls_ to pki_ prefix
* Replace creation of local pki dir with assert
* Revert "Added tasks to create dirs exist incase of customization & README changes"
* Standarize default var names
* importing playbooks in sequence order instead of calling role
* remove creates arg which fails if the file already exists


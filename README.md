Ansible Role for Vault
=========

Example Requirements
--------------------

```yml
---
roles:
  - name: cert-generator
    src: git+https://github.com/moukail/ansible-role-cert-generator.git
    version: main
  - name: vault.podman
    src: git+https://github.com/moukail/ansible-role-vault.podman.git
    version: main

```

Example Playbook
----------------

```yaml
---
- name: Install Vault
  hosts: all
  become: true

  vars:
    cert_ca_passphrase: "" # REQUIRED: provide through Ansible Vault (minimum 16 chars)
    cert_ca_dir: /etc/pki/tls/
    cert_ca_certificate_path: /etc/pki/tls/ca.crt

    vault_port: 8200

  tasks:
    - name: Generate certificate
      ansible.builtin.include_role:
        name: cert-generator
      vars:
        cert_dir: /opt/vault/tls
        cert_certificate_path: /opt/vault/tls/server.crt
        cert_key_path: /opt/vault/tls/server.key

    - name: Run Vault with Podman
      ansible.builtin.include_role:
        name: vault.podman

  post_tasks:
    - name: Show Vault Keys
      ansible.builtin.debug:
        msg:
          - "Vault address: https://{{ ansible_facts['default_ipv4']['address'] }}:{{ vault_port }}"
          - "Vault token: {{ vault_token }}"
          - "Vault keys: {{ vault_keys }}"
      when: vault_token is defined and vault_keys is defined
Ansible Role: Generate cert-manager Certificate for Kubernetes
==============================================================

Ansible role to generate a certificate for
[cert-manager](https://github.com/jetstack/cert-manager) on Kubernetes.

Role Variables
--------------

```yaml
# Certificate name
kubernetes_cert_manager_cert_name:
# Namespace
kubernetes_cert_manager_cert_namespace: "default"

# cert-manager issuer to use
kubernetes_cert_manager_cert_issuer_ref:
# name: issuer_ref name to use
# kind: Issuer/ClusterIssuer

# Certificate common name
kubernetes_cert_manager_cert_common_name:
# Certificate DNS names (Subject Alternative Names)
kubernetes_cert_manager_cert_dns_names: []

# Secret name where the TLS cert will be stored
kubernetes_cert_manager_cert_secret_name:

# List of acme configs, dumps exactly as it is in the manifest as
# `spec/acme/config` (see example for more details)
kubernetes_cert_manager_cert_acme_configs: []
```

Dependencies
------------

Kubectl needs to be installed on the host targeted by the role.


Example Playbook
----------------

```yaml
- hosts: kube-master
  run_once: true
  vars:
    kubernetes_cert_manager_cert_issuer_ref:
      name: "letsencrypt-issuer"
      kind: "ClusterIssuer"

    kubernetes_cert_manager_cert_name: "example-com"
    kubernetes_cert_manager_cert_secret_name: example-tls
    kubernetes_cert_manager_cert_common_name: "example.com"
    kubernetes_cert_manager_cert_dns_names:
      - "www.example.com"
    kubernetes_cert_manager_cert_acme_configs:
      - http01:
          ingress: "example-ingress"
        domains:
          - example.com
          - www.example.com

  roles:
    - role: Anthony25.kubernetes-cert-manager-cert
```

Use `run_once` to run the role on only one available master in the cluster.

If the ingress is pushed by Ansible, its variables can be re-used. An example
with my [kubernetes-nextcloud
role](https://github.com/Anthony25/ansible-role-kubernetes-nextcloud):

```yaml
- hosts: kube-master
  run_once: true
  vars:
    kubernetes_nextcloud_ingress:
      name: "nextcloud-ingress"
      host: "nextcloud.example.com"
      annotations:
      tls:
        - secretName: "nextcloud-ingress-tls"
          hosts:
            - "nextcloud.example.com"

    kubernetes_cert_manager_cert_name: "nextcloud-example-com"
    kubernetes_cert_manager_cert_secret_name: |
      {{ kubernetes_nextcloud_ingress.tls[0].secretName }}
    kubernetes_cert_manager_cert_common_name: |
      {{ kubernetes_nextcloud_ingress.host }}
    kubernetes_cert_manager_cert_dns_names: |
      {{ kubernetes_nextcloud_ingress.tls[0].hosts }}
    kubernetes_cert_manager_cert_acme_configs:
      - http01:
          ingress: "{{ kubernetes_nextcloud_ingress.name }}"
        domains: "{{ kubernetes_nextcloud_ingress.tls[0].hosts }}"

  roles:
    - role: Anthony25.kubernetes-nextcloud
    - role: Anthony25.kubernetes-cert-manager-cert
      tags:
        - certs
  tags:
    - nextcloud
```

License
-------

Tool under the BSD license. Do not hesitate to report bugs, ask me some
questions or do some pull request if you want to!

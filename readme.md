Things to remember to do for creating the cluster correctly:
1. Install argoCD: https://argo-cd.readthedocs.io/en/stable/
2. For K3S make sure 
      "--write-kubeconfig-mode 600"
      "--disable servicelb"
      "--disable traefik"
      "--secrets-encryption"
      "--kube-apiserver-arg=admission-control-config-file=/etc/rancher/k3s/server/psa.yaml"
      "--kubelet-arg=pod-max-pids=100"
      "--kube-apiserver-arg=enable-admission-plugins=NodeRestriction"
      "--kubelet-arg=tls-cipher-suites=TLS_ECDHE_ECDSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_RSA_WITH_AES_256_GCM_SHA384,TLS_ECDHE_ECDSA_WITH_CHACHA20_POLY1305"
3. Create a PSA (See https://docs.k3s.io/security/hardening-guide for security guidance)
```
    apiVersion: apiserver.config.k8s.io/v1
    kind: AdmissionConfiguration
    plugins:
    - name: PodSecurity
      configuration:
        apiVersion: pod-security.admission.config.k8s.io/v1
        kind: PodSecurityConfiguration
        defaults:
          enforce: "restricted"
          enforce-version: "latest"
          warn: "restricted"
          warn-version: "latest"
        exemptions:
          usernames: []
          runtimeClasses: []
          namespaces: [kube-system, argocd]
```
# Wanted to make a homelab...thought why not learn about kubernetes. Winter 2025-2026 Project.

## WIP

General technologies:
- K3S running on nixos
- ArgoCD for gitops pipeline, with the goal of every part of the setup being declarative
- Kyverno for the policy engine and setting up sensible defaults for security (why doesn't kubernetes do this out of the box?!?)
  - Require pods run as non-root, non-root group, and with RuntimeDefault seccomdProfile with capabilities dropped
  - CPU and memory limits
  - Read only file system
  - Disable auto mount service token
  - No inter-namespace network communication (except to DNS)
- Grafana + Prometheus for logging and monitoring (not much use right now, but was cool to setup some persistent volume claim)
- A minecraft server as a "real" application to test with
- Headlamp for UI administration

As simple as the setup is/looks...it took a decent amount of failing to get working in the ~1 week I set the core of it up.

I'm not an expert and not going to pretend like there probably isn't some security holes here...but most of it won't be exposed to the outside world anyway.

---

Things to remember to do for creating the cluster correctly:
1. Install argoCD: https://argo-cd.readthedocs.io/en/stable/
2. For K3S make sure 
```
      "--write-kubeconfig-mode=600"
      "--disable=traefik"
      "--secrets-encryption"
      "--kube-apiserver-arg=admission-control-config-file=/etc/rancher/k3s/server/psa.yaml"
      "--kubelet-arg=pod-max-pids=2048"
      "--kube-apiserver-arg=enable-admission-plugins=NodeRestriction"
      "--kubelet-arg=authentication-token-webhook=true"
      "--kubelet-arg=authorization-mode=Webhook"
```
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
          namespaces: [kube-system]
```
4. For MC make sure these env are set:
- LOCAL_BACKUP_DIR: /mc-storage/server
- LOCAL_SERVER_DIR: /mc-storage/backups


To monitor: <br/>
`sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml port-forward svc/headlamp -n headlamp 8080:80`<br/>
`sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml port-forward -n monitoring deploy/monitoring-grafana 8081:3000`<br/>
`sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml port-forward svc/argocd-server -n argocd 8082:443`<br/>
`sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml create token headlamp -n headlamp`

To be aware of: 
- https://media.defense.gov/2022/Aug/29/2003066362/-1/-1/0/CTR_KUBERNETES_HARDENING_GUIDANCE_1.2_20220829.PDF
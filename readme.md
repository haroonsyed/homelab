# Wanted to make a homelab...thought why not learn about kubernetes. Winter 2025-2026 Project.

## WIP

## Motivation
I'm not necessarily against hosting things on the cloud, but I like hosting stuff myself more.
1. It's free
2. It's usually a good learning process (although applications/power outages can be annoying but I'm never hosting anything that important yet)
3. The consolidation of the whole internet into aws/azure/gcp is a little concerning. I like the idea of stuff still being alive outside of those.

Now usually when I hosted stuff in the past, I just run the app in a VM, port forward through my router and call it a day.
For the homelab setup I thought I would use proxmox...but I don't have an old computer with me to install it on as an OS. 

So when I saw kubernetes ran ontop of the host OS, I thought why not (despite hesitations around security...still need to look into rootless mode to be more sure).
I know most enterprises use it, so I'd be learning stuff for both work and hobby purposes...in hindsight I did not realize how much there was to know about the ecosystem and I have a bigger appreciation for what devops teams do now (not pretending to be an expert or anything).

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
0. Install k3s (Includes reinstall instructions if you need to nuke existing install...I was playing around with cilium for a bit)
```
sudo pkill -9 k3s
sudo pkill -9 containerd-shim
sudo systemctl stop k3s
for m in $(mount | grep -E 'k3s|containerd' | awk '{print $3}'); 
do sudo umount -l "$m" 
done
sudo rm -rf /run/k3s
sudo rm -rf /run/containerd
sudo rm -rf /var/lib/rancher/k3s
sudo rm -rf /etc/rancher/k3s
sudo rm -rf /run/k3s
sudo rm -rf /run/containerd
sudo rm -rf /var/lib/cni
sudo rm -rf /var/lib/cilium
sudo rm -rf /etc/cni/net.d/*
sudo ip link delete cilium_host
sudo ip link delete cilium_net
sudo ip link delete cilium_vxlan
sudo ip link delete flannel.1
sudo iptables-save | grep -iv cilium | sudo iptables-restore
sudo ip6tables-save | grep -iv cilium | sudo ip6tables-restore
sudo nixos-rebuild switch -I nixos-config=/home/haroonsyed/.config/nixos/configuration-pc.nix --upgrade
1. For K3S make sure 
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
2. Install argoCD and bootstrap from repo
```
sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml create namespace argocd
sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
sudo kubectl --kubeconfig /etc/rancher/k3s/k3s.yaml apply -f manifest.yaml
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
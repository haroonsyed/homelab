Things to remember to do for creating the cluster correctly:
1. Install argoCD: https://argo-cd.readthedocs.io/en/stable/
2. For K3S make sure 
  - "--write-kubeconfig-mode 600"
  - "--disable servicelb"
  - "--disable traefik"
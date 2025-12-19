KUBECONFIG=~/.kube/k3s.yaml

kubectl -n databases create secret generic pg-miniflux-user \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=miniflux_user \
  --from-literal=password='REPLACE_WITH_STRONG_PASSWORD'

kubectl -n databases label secret pg-miniflux-user cnpg.io/reload=true --overwrite

kubectl -n databases port-forward svc/pg-rw 5432:5432

kubectl -n argocd port-forward svc/argocd-server 8080:80 --address 0.0.0.0

https://github.com/cloudnative-pg/cloudnative-pg/issues/3756

kubectl -n argocd create secret generic sops-age \
  --from-file=keys.txt=./age.key

For secrets use
`sops --encrypt --in-place projects/secrets/miniflux-db.yaml`
`sops edit projects/secrets/miniflux-db.yaml`

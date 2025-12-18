Create a secret for the Miniflux user password

kubectl -n databases create secret generic pg-miniflux-user \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=miniflux_user \
  --from-literal=password='REPLACE_WITH_STRONG_PASSWORD'

kubectl -n databases label secret pg-miniflux-user cnpg.io/reload=true --overwrite

kubectl -n databases port-forward svc/pg-rw 5432:5432

kubectl -n argocd port-forward svc/argocd-server 8080:80 --address 0.0.0.0

https://github.com/cloudnative-pg/cloudnative-pg/issues/3756

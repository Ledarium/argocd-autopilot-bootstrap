Create a secret for the Miniflux user password

kubectl -n databases create secret generic pg-miniflux-user \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=miniflux_user \
  --from-literal=password='REPLACE_WITH_STRONG_PASSWORD'

kubectl -n databases label secret pg-miniflux-user cnpg.io/reload=true --overwrite

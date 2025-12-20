KUBECONFIG=~/.kube/k3s.yaml

kubectl -n databases create secret generic pg-miniflux-user \
  --type=kubernetes.io/basic-auth \
  --from-literal=username=miniflux_user \
  --from-literal=password='REPLACE_WITH_STRONG_PASSWORD'

kubectl -n databases label secret pg-miniflux-user cnpg.io/reload=true --overwrite

kubectl -n databases port-forward svc/pg-rw 5432:5432

kubectl -n argocd port-forward svc/argocd-server 8080:80 --address 0.0.0.0

https://github.com/cloudnative-pg/cloudnative-pg/issues/3756

## TODO

setup https://argo-cd.readthedocs.io/en/stable/user-guide/multiple_sources/

separate namespaces

## Sealing secrets

Download cluster private key
```
kubeseal --fetch-cert \
  --controller-name sealed-secrets \
  --controller-namespace sealed-secrets \
  > sealed-secrets.pub.pem
```

Create environments/testing/secrets/pg-miniflux-user.yaml DO NOT COMMIT

```
kubeseal \
  --cert sealed-secrets.pub.pem \
  --format yaml \
  --namespace testing \
  --name pg-miniflux-user \ < environments/testing/secrets/pg-miniflux-user.yaml > environments/testing/secrets/pg-miniflux-user.sealed.yaml
```

Commit .sealed.yaml and remove the other.

If you need to reapply do `kubectl apply -f environments/testing/secrets/pg-miniflux-user.sealed.yaml`

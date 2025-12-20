## Commands

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

Pin all versions

## Get secret

`kubectl get secret db-user-pass -o jsonpath='{.data.password}' | base64 --decode`

## Sealing secrets

Download cluster private key
```
kubeseal --fetch-cert \
  --controller-name sealed-secrets \
  --controller-namespace sealed-secrets \
  > sealed-secrets.pub.pem
```

Create environments/testing/pg-cluster/pg-forgejo-user.yaml DO NOT COMMIT
Example secret:
```
apiVersion: bitnami.com/v1alpha1
kind: Secret
metadata:
  name: <replace>
  namespace: testing
type: Opaque
stringData:
  password: <redacted>
  username: <redacted>
```


```
kubeseal \
  --cert sealed-secrets.pub.pem \
  --format yaml \
  --namespace testing \
  --name pg-forgejo-user \ < environments/testing/pg-cluster/pg-forgejo-user.yaml > environments/testing/pg-cluster/pg-forgejo-user.sealed.yaml
```

Commit .sealed.yaml and remove the other.

If you need to reapply do `kubectl apply -f environments/testing/secrets/pg-cluster-user.sealed.yaml`

### Alternatively (test)

echo -n bar | kubectl create secret generic mysecret --dry-run=client --from-file=foo=/dev/stdin -o json \
  | kubeseal > mysealedsecret.json

## Test postgres from pod

`kubectl -n default run -it --rm psqltest --image=postgres:17 --   bash -lc 'getent hosts pg-r.testing.svc.cluster.local && echo OK'`
All commands and output from this session will be recorded in container logs, including credentials and sensitive information passed through the command prompt.
If you don't see a command prompt, try pressing enter.
warning: couldn't attach to pod/psqltest, falling back to streaming logs: Internal error occurred: unable to upgrade connection: container psqltest not found in pod psqltest_default
10.43.26.219    pg-r.testing.svc.cluster.local
OK
pod "psqltest" deleted from default namespace

## Un/reinstall ArgoCD

kubectl delete -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
rm /var/lib/argocd-autopilot/bootstrapped
if need to reinstall:
systemctl restart argocd-autopilot-bootstrap


Creation of a selead secret

```
echo -n bar | kubectl create secret generic mysecret --dry-run --from-file=foo=/dev/stdin -o yaml >mysecret.yaml
cat mysecret.yaml 
kubeseal -o yaml <mysecret.yaml >mysealedsecret.yaml 
```

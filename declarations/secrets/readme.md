Run the following to add the database passwords as secrets

```bash
kubectl create secret generic db-passwords
  --from-literal=db-password='password'
  --from-literal=db-root-password='password'
```

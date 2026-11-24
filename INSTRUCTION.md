# Task 7: Configuration management

## 1. Apply the changes

Apply the manifests in this order (the namespace first, then the ConfigMap and the Secret,
and the Deployment last because it uses them):

```
kubectl apply -f .infrastructure/namespace.yml
kubectl apply -f .infrastructure/configMap.yml
kubectl apply -f .infrastructure/secret.yml
kubectl apply -f .infrastructure/deployment.yml
```

- `configMap.yml` creates the ConfigMap `todoapp-config` with `PYTHONUNBUFFERED`.
- `secret.yml` creates the Secret `todoapp-secret` with `SECRET_KEY`.
- `deployment.yml` reads both through `valueFrom` (`configMapKeyRef` and `secretKeyRef`).
- `src/todolist/settings.py` reads the key from the environment:
  `SECRET_KEY = os.environ.get("SECRET_KEY", "insecure-local-dev-key")`.
  The fallback value is used only for a local run without Kubernetes.

## 2. Validate the changes

Check that the resources exist and the pod is running:

```
kubectl get pods,configmap,secret -n todoapp
```

Expected result: the ConfigMap `todoapp-config`, the Secret `todoapp-secret` and a pod `1/1 Running`.

Check that the variables are set in the container (use the exact pod name from the previous command):

```
kubectl exec <pod-name> -n todoapp -- env | grep -e PYTHONUNBUFFERED -e SECRET_KEY
```

Expected result:

```
PYTHONUNBUFFERED=1
SECRET_KEY=change-me-demo-secret-key-1234567890-abcdefghijklmnopqrstuvwxyz
```

Check the values stored in the resources:

```
kubectl describe configmap todoapp-config -n todoapp
kubectl get secret todoapp-secret -n todoapp -o jsonpath="{.data.SECRET_KEY}"
```

The Secret value is stored base64-encoded (`stringData` in the manifest is encoded by Kubernetes).

## 3. Notes

- The value in `secret.yml` is a demo value. The key that was hardcoded in `settings.py`
  was public in Git history, so it must not be reused. In a real project a Secret manifest
  with a real key is not committed to Git.
- The image `ikulyk404/todoapp:3.0.0` is prebuilt with the old `settings.py`. To make the
  running application use the key from the Secret, the image has to be rebuilt from the
  changed code.
  
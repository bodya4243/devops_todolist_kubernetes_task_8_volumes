# Validation instructions

## Deploy the resources

From the repository root, deploy the complete stack:

```bash
./bootstrap.sh
```

Wait for the application rollout and confirm that its pod is running:

```bash
kubectl rollout status deployment/todoapp -n todoapp
kubectl get pods -n todoapp -l app=todoapp
```

The pod should show `Running` and `READY` as `1/1`. The application is exposed by
the `todoapp-nodeport` service; obtain its port with:

```bash
kubectl get service todoapp-nodeport -n todoapp
curl --fail http://localhost:30007/api/health
```

The health request should return `Health OK`.

## Validate persistent storage

Confirm that the PV and claim are bound, then check the mount inside the running
application container:

```bash
kubectl get pv todo-pv
kubectl get pvc todo-pvc -n todoapp
kubectl exec -n todoapp deployment/todoapp -- sh -c 'mount | grep " /app/data "'
```

Both PV and PVC should report `Bound`, and the last command should show the
`/app/data` mount.

## Validate ConfigMap files

The ConfigMap is projected read-only at `/app/configs`, with one file per data key.
Verify the expected key and its contents:

```bash
kubectl exec -n todoapp deployment/todoapp -- sh -c 'ls -l /app/configs && cat /app/configs/PYTHONUNBUFFERED'
```

The listing should include `PYTHONUNBUFFERED`; its value should be `1`. The
permissions shown by `ls -l` should not include write permission for the container
user.

## Validate Secret files

The Secret is projected read-only at `/app/secrets`. Confirm its file exists and
matches the Secret value without printing the secret itself:

```bash
kubectl exec -n todoapp deployment/todoapp -- test -f /app/secrets/SECRET_KEY
kubectl exec -n todoapp deployment/todoapp -- sh -c 'test ! -w /app/secrets/SECRET_KEY'
```

Both commands should exit successfully.

Stable Deployment (backend-deployment.yaml)
```bash
labels:
  app: backend
  version: stable
```
Canary Deployment (backend-canary.yaml)
```bash
labels:
  app: backend
  version: canary
```
Backend Service (backend-service.yaml)
```bash
selector:
  app: backend
```

To scale:
```bash
kubectl scale deployment backend-canary --replicas=2
kubectl scale deployment backend --replicas=3
```

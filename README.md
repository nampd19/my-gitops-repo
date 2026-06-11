# My GitOps Repo — ArgoCD Testing

## Cấu trúc thư mục

```
my-gitops-repo/
├── applications/
│   └── guestbook/
│       ├── base/                        # Kustomize base chung
│       │   ├── deployment.yaml          # Deployment + Service
│       │   └── kustomization.yaml
│       ├── overlays/
│       │   ├── dev/                     # Override cho dev (2 replicas, namespace: guestbook-dev)
│       │   │   ├── deployment-patch.yaml
│       │   │   └── kustomization.yaml
│       │   └── prod/                    # Override cho prod (3 replicas, namespace: guestbook-prod)
│       │       ├── deployment-patch.yaml
│       │       └── kustomization.yaml
│       ├── argocd-app-dev.yaml          # ArgoCD Application CRD cho dev
│       └── argocd-app-prod.yaml         # ArgoCD Application CRD cho prod
└── infrastructure/
    └── argocd/
        └── argocd-install.yaml          # Ghi chú cài đặt ArgoCD
```

## Các bước test ArgoCD

### 1. Cài ArgoCD lên cluster

```bash
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

### 2. Truy cập ArgoCD UI

```bash
# Lấy password admin
kubectl -n argocd get secret argocd-initial-admin-secret \
  -o jsonpath="{.data.password}" | base64 -d

# Port-forward UI
kubectl port-forward svc/argocd-server -n argocd 8080:443
# Truy cập: https://localhost:8080  (user: admin)
```

### 3. Đăng ký repo này với ArgoCD (nếu private)

```bash
argocd repo add https://github.com/nampd19/my-gitops-repo.git \
  --username <user> --password <token>
```

### 4. Tạo ArgoCD Applications

```bash
# Dev environment
kubectl apply -f applications/guestbook/argocd-app-dev.yaml

# Prod environment
kubectl apply -f applications/guestbook/argocd-app-prod.yaml
```

### 5. Kiểm tra kết quả

```bash
kubectl get pods -n guestbook-dev
kubectl get pods -n guestbook-prod

# Xem qua ArgoCD CLI
argocd app list
argocd app get guestbook-dev
argocd app get guestbook-prod
```

### 6. Test Kustomize build local (không cần cluster)

```bash
# Kiểm tra output của dev overlay
kubectl kustomize applications/guestbook/overlays/dev

# Kiểm tra output của prod overlay
kubectl kustomize applications/guestbook/overlays/prod
```

## Lưu ý

- **`repoURL`** trong `argocd-app-*.yaml` — thay bằng URL repo thực tế của bạn nếu khác.
- **`syncPolicy.automated`** — ArgoCD sẽ tự động sync khi có commit mới.
- **`CreateNamespace=true`** — ArgoCD tự tạo namespace nếu chưa có.

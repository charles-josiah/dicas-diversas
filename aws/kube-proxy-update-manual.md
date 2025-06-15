# 🛠️ Manual: Atualização do kube-proxy com imagem custom no ECR

## ✅ 1. Baixar a imagem do kube-proxy desejada

Exemplo (imagem pública da AWS para EKS 1.28):
```bash
docker pull public.ecr.aws/eks-distro/kubernetes/kube-proxy:v1.28.15-eks-1-28-51
```

## ✅ 2. Taguear para o seu ECR

Substitua o `<seu-repo>`:
```bash
docker tag public.ecr.aws/eks-distro/kubernetes/kube-proxy:v1.28.15-eks-1-28-51 \
422025336571.dkr.ecr.us-west-2.amazonaws.com/eks-img:kube-proxy-v1.28.15
```

## ✅ 3. Fazer login no ECR

```bash
aws ecr get-login-password --region us-west-2 | docker login --username AWS --password-stdin 422025336571.dkr.ecr.us-west-2.amazonaws.com
```

## ✅ 4. Subir a imagem para o seu ECR

```bash
docker push 422025336571.dkr.ecr.us-west-2.amazonaws.com/eks-img:kube-proxy-v1.28.15
```

---

## ✅ 5. Atualizar o DaemonSet do kube-proxy

```bash
kubectl -n kube-system edit daemonset kube-proxy
```

➡️ Edite a linha da imagem:
```yaml
image: 422025336571.dkr.ecr.us-west-2.amazonaws.com/eks-img:kube-proxy-v1.28.15
```

Salve e saia (`:wq` no `vi`).

---

## ✅ 6. Verificar o rollout

```bash
kubectl -n kube-system rollout status daemonset kube-proxy
```

---

## ✅ 7. Validar pods atualizados

### Ver os pods em execução:
```bash
kubectl get pods -n kube-system -l k8s-app=kube-proxy -o wide
```

### Checar imagem de cada pod:
```bash
kubectl -n kube-system get pods -l k8s-app=kube-proxy -o jsonpath="{range .items[*]}{.metadata.name}: {.spec.containers[0].image}{'\\n'}{end}"
```

### Ver diretamente a imagem no DaemonSet:
```bash
kubectl -n kube-system describe daemonset kube-proxy | grep Image
```

---

## ✅ 8. Diagnóstico e eventos

### Logs dos pods do kube-proxy:
```bash
kubectl -n kube-system logs -l k8s-app=kube-proxy
```

### Eventos recentes no namespace kube-system:
```bash
kubectl get events -n kube-system --sort-by='.lastTimestamp'
```

---

## ✅ Comandos úteis (referência rápida):

```bash
kubectl -n kube-system edit daemonset kube-proxy
kubectl get pods -n kube-system -l k8s-app=kube-proxy -o wide
kubectl -n kube-system logs -l k8s-app=kube-proxy
kubectl get events -n kube-system --sort-by='.lastTimestamp'
kubectl -n kube-system describe daemonset kube-proxy | grep Image
kubectl -n kube-system get pods -l k8s-app=kube-proxy -o jsonpath="{range .items[*]}{.metadata.name}: {.spec.containers[0].image}{'\\n'}{end}"
kubectl get daemonset kube-proxy -n kube-system -o jsonpath='{.spec.template.spec.containers[0].image}'
```

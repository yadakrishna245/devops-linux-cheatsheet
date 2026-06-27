# Day 9 — Kubernetes ☸️

## LinkedIn Post:

```
☸️ [Day 9/15] DevOps Cheat Sheet: Kubernetes

kubectl commands I run 10+ times daily.

No fluff. Just the ones that matter in production:

━━━━━━━━━━━━━━━━━

# Where am I?
kubectl config get-contexts
kubectl config use-context <context-name>
kubectl cluster-info
kubectl get nodes -o wide

# Pods — the daily bread
kubectl get pods -A                          # All namespaces
kubectl get pods -o wide                     # With node info
kubectl describe pod <pod-name>              # WHY is it failing?
kubectl logs <pod-name> -f --tail=100        # Follow logs
kubectl exec -it <pod-name> -- /bin/sh       # Get inside

# Deployments
kubectl set image deployment/nginx nginx=nginx:1.25
kubectl rollout status deployment/nginx
kubectl rollout undo deployment/nginx        # Instant rollback!
kubectl scale deployment/nginx --replicas=5

# Troubleshooting
kubectl get events --sort-by='.lastTimestamp'
kubectl top pods                             # Resource usage
kubectl top nodes
kubectl get pods --field-selector=status.phase=Failed

━━━━━━━━━━━━━━━━━

# Quick YAML generation (don't write from scratch!)
kubectl run test --image=busybox --dry-run=client -o yaml > pod.yaml
kubectl create deployment web --image=nginx --dry-run=client -o yaml > deploy.yaml

━━━━━━━━━━━━━━━━━

💡 Pro tip: `kubectl rollout undo` is your panic button.

Bad deployment? One command. Instant rollback.
No downtime. No drama. That's the power of K8s.

♻️ Repost for your Kubernetes community
#Kubernetes #K8s #DevOps #CloudEngineering #SRE #kubectl
```

## First Comment:

```
🔗 Full repo: https://github.com/yadakrishna245/devops-linux-cheatsheet
⭐ Star it — covers beginner to advanced kubectl!

Tomorrow: AWS CLI — one-liners that save hours ☁️
```

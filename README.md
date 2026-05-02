# 🔐 Kubernetes Security Hardening

Déploiement et sécurisation d'une application web sur Kubernetes avec les bonnes pratiques de sécurité.

## 🏗️ Architecture
Cluster Minikube avec 4 nodes (1 control-plane + 3 workers)

## 🔐 Fonctionnalités de Sécurité

- **Secrets** : Stockage sécurisé des credentials
- **Resource Limits** : Protection contre les attaques DoS (CPU: 200m, RAM: 128Mi)
- **SecurityContext** : Suppression des capabilities dangereuses (NET_RAW, SYS_ADMIN...)
- **RBAC** : dev-user peut uniquement lire les pods (get/list), pas créer ni supprimer
- **Network Policy** : Isolation du trafic réseau entrant/sortant

## 📁 Structure
- secret.yaml — Secrets Kubernetes
- deployment.yaml — Déploiement sécurisé nginx
- role.yaml — Rôle RBAC lecture seule
- rolebinding.yaml — Association rôle utilisateur
- netpol.yaml — Network Policy

## 🚀 Déploiement
kubectl apply -f secret.yaml
kubectl apply -f deployment.yaml
kubectl apply -f role.yaml
kubectl apply -f rolebinding.yaml
kubectl apply -f netpol.yaml

## 🧪 Tests RBAC
kubectl auth can-i delete pods --as=dev-user  # no
kubectl auth can-i get pods --as=dev-user     # yes

## 🛠️ Stack
Kubernetes v1.35 | Minikube | kubectl | nginx:alpine

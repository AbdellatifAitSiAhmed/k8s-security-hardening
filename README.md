# 🔐 Kubernetes Security Hardening

> Déploiement et sécurisation d'une application web sur Kubernetes avec les meilleures pratiques de sécurité : Secrets, RBAC, Network Policy, SecurityContext, Resource Limits.

**Filière** : RSSP — Réseaux, Systèmes et Services Programmables
**Établissement** : ENSA Marrakech | Université Cadi Ayyad  
**Année** : 2025/2026  
**Réalisé par** : AIT SI AHMED Abdellatif

---

## 🏗️ Architecture

```
Cluster Minikube — 4 nodes
┌────────────────────────────────────────┐
│  control-plane (minikube)              │
├────────────────────────────────────────┤
│  worker-1   worker-2   worker-3        │
└────────────────────────────────────────┘
          │
    namespace: secure-app
          │
    ┌─────▼──────┐
    │   nginx    │  ← deployment.yaml (SecurityContext + Limits)
    │  :alpine   │
    └─────┬──────┘
          │
  ┌───────┼────────┐
  │       │        │
secret  rbac   netpol
.yaml  role+   .yaml
       binding
       +sa
```

---

## 🔐 Fonctionnalités de sécurité

| Mécanisme | Fichier | Détail |
|---|---|---|
| **Secrets** | `secret.yaml` | Credentials encodés base64, montés en variable d'env |
| **Resource Limits** | `deployment.yaml` | CPU: 200m max / RAM: 128Mi max — protection DoS |
| **SecurityContext** | `deployment.yaml` | Drop NET_RAW, SYS_ADMIN — runAsNonRoot: true |
| **RBAC** | `role.yaml` + `rolebinding.yaml` + `serviceaccount.yaml` | dev-user : get/list pods uniquement |
| **Network Policy** | `netpol.yaml` | Isolation complète trafic entrant/sortant |

---

## 📁 Structure du projet

```
k8s-security-hardening/
├── namespace.yaml        # Namespace isolé : secure-app
├── secret.yaml           # Secrets Kubernetes (base64)
├── serviceaccount.yaml   # ServiceAccount dev-user
├── role.yaml             # Rôle RBAC lecture seule
├── rolebinding.yaml      # Association role ↔ dev-user
├── deployment.yaml       # Déploiement nginx sécurisé
├── netpol.yaml           # Network Policy isolation réseau
├── .gitignore
├── LICENSE
└── README.md
```

---

## 🚀 Prérequis & Installation

```bash
# Prérequis
minikube v1.35+
kubectl v1.35+
```

```bash
# Démarrer le cluster 4 nodes
minikube start --nodes 4 --driver=docker

# Vérifier les nodes
kubectl get nodes
```

---

## ▶️ Déploiement

```bash
# 1. Créer le namespace
kubectl apply -f namespace.yaml

# 2. Créer les secrets
kubectl apply -f secret.yaml -n secure-app

# 3. Créer le ServiceAccount
kubectl apply -f serviceaccount.yaml -n secure-app

# 4. Appliquer le RBAC
kubectl apply -f role.yaml -n secure-app
kubectl apply -f rolebinding.yaml -n secure-app

# 5. Déployer l'application
kubectl apply -f deployment.yaml -n secure-app

# 6. Appliquer la Network Policy
kubectl apply -f netpol.yaml -n secure-app

# Vérifier que tout tourne
kubectl get all -n secure-app
```

---

## 🧪 Tests de validation

### Test RBAC
```bash
# dev-user NE PEUT PAS supprimer les pods
kubectl auth can-i delete pods --as=system:serviceaccount:secure-app:dev-user -n secure-app
# → no ✅

# dev-user PEUT lister les pods
kubectl auth can-i get pods --as=system:serviceaccount:secure-app:dev-user -n secure-app
# → yes ✅

# dev-user NE PEUT PAS créer des pods
kubectl auth can-i create pods --as=system:serviceaccount:secure-app:dev-user -n secure-app
# → no ✅
```

### Test SecurityContext
```bash
# Vérifier les capabilities supprimées
kubectl get pod -n secure-app -o jsonpath='{.items[0].spec.containers[0].securityContext}'
```

### Test Resource Limits
```bash
kubectl describe pod -n secure-app | grep -A4 "Limits"
# CPU: 200m | Memory: 128Mi ✅
```

---

## ✅ Résultats

| Test | Résultat |
|---|---|
| RBAC delete pods | ❌ Refusé (attendu) |
| RBAC get pods | ✅ Autorisé |
| RBAC create pods | ❌ Refusé (attendu) |
| SecurityContext NET_RAW | ✅ Supprimé |
| SecurityContext SYS_ADMIN | ✅ Supprimé |
| Resource Limit CPU 200m | ✅ Appliqué |
| Resource Limit RAM 128Mi | ✅ Appliqué |
| Network Policy isolation | ✅ Active |

---

## 🛠️ Stack technique

![Kubernetes](https://img.shields.io/badge/Kubernetes-v1.35-326CE5?logo=kubernetes&logoColor=white)
![Minikube](https://img.shields.io/badge/Minikube-4nodes-F5A800?logo=kubernetes&logoColor=white)
![nginx](https://img.shields.io/badge/nginx-alpine-009639?logo=nginx&logoColor=white)

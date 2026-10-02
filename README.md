# GitOps Kubernetes Deployment with Argo CD & kOps on AWS

Ce projet met en œuvre une architecture **GitOps complète** pour automatiser le déploiement d'une application sur un cluster **Kubernetes hébergé sur AWS** et provisionné avec **kOps**.

**Argo CD** assure la synchronisation entre l'état souhaité, défini dans un dépôt Git, et l'état réel du cluster Kubernetes. Toute modification des manifestes Kubernetes versionnés dans Git peut ainsi être détectée et appliquée automatiquement au cluster.

---

## 🎯 Objectifs du Projet

Ce projet permet de mettre en pratique :

* ☁️ Provisionnement d'un cluster Kubernetes sur AWS avec **kOps**
* ☸️ Administration d'un cluster Kubernetes
* 🔄 Mise en œuvre d'une approche **GitOps**
* 🚀 Déploiement continu avec **Argo CD**
* 📦 Gestion des applications via des manifestes Kubernetes
* 🌐 Exposition des applications avec **Ingress / Load Balancer**
* 🗂️ Utilisation de **GitHub comme source de vérité**
* 🔍 Synchronisation automatique entre Git et Kubernetes

---

# 🏗️ Architecture du Projet

L'architecture repose sur quatre composants principaux.

### 1. GitHub — Source de vérité

Le dépôt GitHub contient les manifestes Kubernetes nécessaires au déploiement de l'application :

```text
deployment.yaml
service.yaml
ingress.yaml
argocd-application.yaml
```

Git constitue la **source de vérité déclarative** du système.

Toute modification de l'infrastructure applicative ou de la configuration Kubernetes est versionnée dans Git.

---

### 2. Argo CD — GitOps Controller

**Argo CD** surveille le dépôt Git et compare en permanence :

```text
État souhaité
     ↓
GitHub
     ↓
Argo CD
     ↓
État réel du cluster
```

Lorsqu'une différence (*drift*) est détectée, Argo CD peut synchroniser le cluster avec l'état défini dans Git.

Argo CD permet notamment de :

* surveiller les dépôts Git ;
* détecter les modifications ;
* synchroniser les manifestes Kubernetes ;
* visualiser l'état des ressources ;
* détecter les différences entre Git et Kubernetes ;
* effectuer des synchronisations manuelles ou automatiques ;
* suivre l'état des applications déployées.

---

### 3. Kubernetes Cluster — AWS / kOps

Le cluster Kubernetes est provisionné et administré avec **kOps**.

Architecture simplifiée :

```text
                    AWS
                     │
                     ▼
                  kOps
                     │
          ┌──────────┴──────────┐
          │                     │
          ▼                     ▼
 Control Plane              Worker Nodes
          │                 ┌─────────────┐
          │                 │ Worker 1    │
          │                 │ Worker 2    │
          │                 └──────┬──────┘
          │                        │
          ▼                        ▼
 Kubernetes API               Application Pods
 etcd
 Scheduler
 Controller Manager
```

Le **Control Plane** assure la gestion du cluster tandis que les **Worker Nodes** exécutent les workloads applicatifs.

---

### 4. Ingress / AWS Load Balancer

L'application peut être exposée vers Internet via un **Ingress Controller** et un **Load Balancer AWS**.

Flux utilisateur :

```text
Utilisateur
     │
     ▼
AWS Load Balancer
     │
     ▼
Ingress Controller
     │
     ▼
Kubernetes Service
     │
     ▼
Application Pods
```

Cette architecture permet de séparer clairement :

* l'accès externe ;
* le routage HTTP/HTTPS ;
* les Services Kubernetes ;
* les Pods applicatifs.

---

# 🔄 GitOps Workflow

Le fonctionnement du projet suit le modèle GitOps suivant :

```text
       Developer
           │
           │ git push
           ▼
       GitHub Repository
           │
           │ Repository Monitoring
           ▼
        Argo CD
           │
           │ Synchronization
           ▼
   Kubernetes Cluster
           │
           ▼
      Application Pods
           │
           ▼
      End Users
```

### Étape 1 — Modification du code ou des manifestes

Le développeur modifie par exemple :

```text
deployment.yaml
service.yaml
ingress.yaml
```

Puis pousse les changements :

```bash
git add .
git commit -m "Update application deployment"
git push origin main
```

---

### Étape 2 — Détection par Argo CD

Argo CD surveille le dépôt Git.

Lorsqu'une modification est détectée, Argo CD compare :

```text
Git Repository
       VS
Kubernetes Cluster
```

---

### Étape 3 — Synchronisation

Si l'Application Argo CD est configurée avec l'auto-sync, Argo CD applique l'état souhaité au cluster.

```text
Git
 │
 │ Desired State
 ▼
Argo CD
 │
 │ Sync
 ▼
Kubernetes
```

---

### Étape 4 — Déploiement

Kubernetes crée ou met à jour les ressources nécessaires :

```text
Deployment
    │
    ▼
ReplicaSet
    │
    ▼
Pods
```

Les Pods sont distribués sur les Worker Nodes selon les règles définies par Kubernetes.

---

### Étape 5 — Accès utilisateur

L'application devient accessible via le chemin réseau configuré :

```text
Internet
   │
   ▼
AWS Load Balancer
   │
   ▼
Ingress Controller
   │
   ▼
Service
   │
   ▼
Pods
```

---

# 📂 Structure du Repository

Une structure possible du projet :

```text
ar-deploy/
│
├── Architecture.png
├── ArgoDC.png
├── App-running.png
│
├── deployment.yaml
├── service.yaml
├── ingress.yaml
├── argocd-application.yaml
│
├── guide_de_deploiement_et_resolution_de_problemes.md
│
└── README.md
```

### Description

| Fichier                                              | Description                         |
| ---------------------------------------------------- | ----------------------------------- |
| `Architecture.png`                                   | Architecture globale du projet      |
| `ArgoDC.png`                                         | Capture de l'interface Argo CD      |
| `App-running.png`                                    | Capture de l'application déployée   |
| `deployment.yaml`                                    | Définition du Deployment Kubernetes |
| `service.yaml`                                       | Service Kubernetes                  |
| `ingress.yaml`                                       | Configuration de l'Ingress          |
| `argocd-application.yaml`                            | Ressource Application Argo CD       |
| `guide_de_deploiement_et_resolution_de_problemes.md` | Guide détaillé                      |
| `README.md`                                          | Documentation principale            |

---

# 🛠️ Prérequis

Avant de commencer, installer et configurer :

* **AWS CLI**
* **kOps**
* **kubectl**
* **Argo CD CLI** *(optionnel)*
* **Git**
* Un compte AWS
* Un bucket S3 pour le state store kOps
* Un domaine Route 53 *(recommandé)*

Exemple :

```bash
aws configure
```

Vérifier les outils :

```bash
aws --version
kops version
kubectl version --client
argocd version --client
git --version
```

---

# ☁️ 1. Provisionnement du Cluster avec kOps

## Configuration du State Store

kOps utilise un bucket S3 pour stocker l'état de configuration du cluster.

```bash
export KOPS_STATE_STORE=s3://votre-bucket-kops-state
```

Définir ensuite le nom du cluster :

```bash
export CLUSTER_NAME=mon-cluster.k8s.local
```

---

## Création du Cluster

Exemple :

```bash
kops create cluster \
  --name=${CLUSTER_NAME} \
  --state=${KOPS_STATE_STORE} \
  --zones=us-east-1a \
  --node-count=2 \
  --node-size=t3.medium \
  --control-plane-size=t3.medium
```

Vérifier la configuration :

```bash
kops validate cluster
```

Afficher les informations du cluster :

```bash
kops get cluster
```

---

## Appliquer la configuration

```bash
kops update cluster \
  --name ${CLUSTER_NAME} \
  --yes \
  --admin
```

Après le déploiement, vérifier les Nodes :

```bash
kubectl get nodes
```

Exemple :

```text
NAME          STATUS   ROLES
control       Ready    control-plane
worker-1      Ready    <none>
worker-2      Ready    <none>
```

---

# ☸️ 2. Vérification du Cluster Kubernetes

Vérifier les Nodes :

```bash
kubectl get nodes -o wide
```

Vérifier les Pods :

```bash
kubectl get pods -A
```

Vérifier les Services :

```bash
kubectl get svc -A
```

Vérifier les Ingress :

```bash
kubectl get ingress -A
```

---

# 🚀 3. Installation d'Argo CD

Créer le namespace :

```bash
kubectl create namespace argocd
```

Installer Argo CD :

```bash
kubectl apply \
  -n argocd \
  -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
```

Vérifier les Pods :

```bash
kubectl get pods -n argocd
```

Attendre que les composants soient dans l'état :

```text
Running
```

---

# 🔐 4. Accès à Argo CD

Vérifier les Services :

```bash
kubectl get svc -n argocd
```

Selon l'architecture choisie, Argo CD peut être exposé via :

* LoadBalancer ;
* Ingress ;
* Port Forward.

Pour un test local :

```bash
kubectl port-forward svc/argocd-server \
  -n argocd 8080:443
```

L'interface peut alors être accessible localement sur :

```text
https://localhost:8080
```

---

# 🔗 5. Connexion du Repository GitHub à Argo CD

Le manifeste `argocd-application.yaml` définit la relation entre :

```text
GitHub Repository
       │
       ▼
    Argo CD
       │
       ▼
Kubernetes Cluster
```

Exemple :

```yaml
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
  name: ar-deploy
  namespace: argocd
spec:
  project: default

  source:
    repoURL: https://github.com/boubacardjibrilla/ar-deploy.git
    targetRevision: HEAD
    path: .

  destination:
    server: https://kubernetes.default.svc
    namespace: default

  syncPolicy:
    automated:
      prune: true
      selfHeal: true
```

Appliquer le manifeste :

```bash
kubectl apply -f argocd-application.yaml
```

Vérifier l'Application :

```bash
kubectl get applications -n argocd
```

---

# 🔄 6. Synchronisation GitOps

Après une modification :

```bash
git add .
git commit -m "Update Kubernetes application"
git push origin main
```

Argo CD détecte la modification.

Pour vérifier :

```bash
kubectl get applications -n argocd
```

Avec le CLI Argo CD :

```bash
argocd app list
```

Afficher les détails :

```bash
argocd app get ar-deploy
```

Effectuer une synchronisation manuelle :

```bash
argocd app sync ar-deploy
```

---

# 📊 7. Vérification du Déploiement

Vérifier le Deployment :

```bash
kubectl get deployment
```

Vérifier les Pods :

```bash
kubectl get pods -o wide
```

Vérifier le Service :

```bash
kubectl get svc
```

Vérifier l'Ingress :

```bash
kubectl get ingress
```

Pour analyser une ressource :

```bash
kubectl describe deployment <deployment-name>
```

Afficher les logs :

```bash
kubectl logs <pod-name>
```

---

# 🌐 8. Accès à l'Application

Le chemin d'accès dépend de la configuration réseau.

Architecture :

```text
                    Internet
                       │
                       ▼
               AWS Load Balancer
                       │
                       ▼
                Ingress Controller
                       │
                       ▼
                Kubernetes Service
                       │
             ┌─────────┴─────────┐
             ▼                   ▼
          Pod 1                Pod 2
```

Vérifier l'adresse externe :

```bash
kubectl get ingress
```

ou :

```bash
kubectl get svc
```

selon la méthode d'exposition utilisée.

---

# 🔍 9. Vérification du GitOps

Une démonstration importante consiste à modifier volontairement le nombre de replicas.

Par exemple :

```yaml
spec:
  replicas: 3
```

Puis :

```bash
git add deployment.yaml
git commit -m "Scale application to 3 replicas"
git push origin main
```

Argo CD détecte la modification et synchronise le cluster.

Vérifier :

```bash
kubectl get pods
```

Le résultat doit refléter l'état déclaré dans Git.

Cette démonstration montre le principe fondamental du GitOps :

```text
Git = Desired State

        ↓

     Argo CD

        ↓

Kubernetes = Actual State
```

---

# 🧠 Concepts DevOps / GitOps Démontrés

Ce projet permet de démontrer plusieurs compétences importantes :

| Domaine                 | Technologies                  |
| ----------------------- | ----------------------------- |
| Cloud                   | AWS                           |
| Infrastructure          | kOps                          |
| Container Orchestration | Kubernetes                    |
| GitOps                  | Argo CD                       |
| Version Control         | Git / GitHub                  |
| Networking              | Kubernetes Services / Ingress |
| Load Balancing          | AWS Load Balancer             |
| Configuration           | YAML                          |
| Automation              | Argo CD                       |
| Deployment              | Kubernetes                    |
| Infrastructure State    | S3 / kOps                     |

---

# 📸 Captures d'Écran

### Architecture

```text
Architecture.png
```

### Argo CD

```text
ArgoDC.png
```

### Application déployée

```text
App-running.png
```

Ces captures permettent de documenter visuellement :

* l'architecture AWS/Kubernetes ;
* l'état de l'application dans Argo CD ;
* le résultat final du déploiement.

---

# 📖 Guide de Déploiement et Dépannage

Pour les procédures détaillées, consulter :

```text
guide_de_deploiement_et_resolution_de_problemes.md
```

Ce document peut notamment couvrir :

* création du cluster kOps ;
* configuration AWS ;
* installation d'Argo CD ;
* configuration du repository Git ;
* problèmes d'Ingress ;
* problèmes de Load Balancer ;
* erreurs de synchronisation Argo CD ;
* problèmes de Pods ;
* diagnostic Kubernetes ;
* vérification réseau.

---

# 🔐 Bonnes Pratiques

Pour un environnement réel :

* Ne pas exposer inutilement l'API Kubernetes sur Internet.
* Restreindre les Security Groups AWS.
* Utiliser HTTPS pour les accès externes.
* Utiliser un certificat TLS valide avec un domaine.
* Protéger les credentials Git.
* Utiliser des repositories privés lorsque nécessaire.
* Éviter de stocker des secrets directement dans Git.
* Utiliser Kubernetes Secrets ou une solution de gestion des secrets.
* Utiliser des comptes IAM avec le principe du moindre privilège.
* Surveiller les ressources AWS afin d'éviter les coûts inutiles.

---

# 🚀 Évolutions Possibles

Ce projet peut être étendu progressivement vers une architecture DevOps/GitOps plus complète :

```text
GitHub
   │
   ▼
Argo CD
   │
   ▼
Kubernetes / kOps
   │
   ├── Ingress Nginx
   ├── Helm
   ├── Secrets Management
   ├── Prometheus
   ├── Grafana
   └── Monitoring
```

Évolutions possibles :

* 🔐 HTTPS avec TLS / Cert-Manager
* 🌐 Ingress Nginx
* 📦 Helm Charts
* 🔄 Argo CD avec plusieurs environnements
* 🧪 CI avec Jenkins ou GitHub Actions
* 📊 Prometheus & Grafana
* 🔐 HashiCorp Vault / AWS Secrets Manager
* 🏗️ Terraform
* ☁️ Migration vers Amazon EKS
* 🌍 Gestion Dev / Staging / Production
* 🔁 Stratégies de déploiement avancées

---

## 👨‍💻 Auteur

**Boubacar Djibrilla**

Projet réalisé dans le cadre de la mise en pratique des technologies :

**AWS · Kubernetes · kOps · Argo CD · GitOps · Docker · Linux · DevOps**

---

⭐ Si ce projet vous a été utile, n'hésitez pas à donner une étoile au repository.

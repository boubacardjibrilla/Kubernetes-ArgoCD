# Guide Complet : Déploiement d'une Application Kubernetes avec Helm, ArgoCD et AWS (kOps)

Ce guide regroupe toutes les étapes nécessaires pour installer **Helm** et **ArgoCD**, déployer une application bancaire via GitOps, et configurer le réseau AWS (Security Groups et Load Balancer) pour corriger les erreurs de connexion.

---

## 1. Installation de Helm

Exécutez les commandes suivantes pour installer le gestionnaire de paquets Helm 3 :

```bash
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3
chmod 700 get_helm.sh
./get_helm.sh
helm version
```

---

## 2. Installation d'ArgoCD via Helm

Ajoutez le dépôt officiel ArgoCD, créez un espace de nom dédié (`namespace`) et lancez l'installation :

```bash
# Ajout et mise à jour du dépôt Helm Argo
helm repo add argo https://argoproj.github.io/argo-helm
helm repo update

# Création du namespace dédié et installation
kubectl create namespace argocd
helm install argocd argo/argo-cd --namespace argocd

# Vérification du déploiement
kubectl get all -n argocd
```

---

## 3. Exposition du Serveur ArgoCD

Pour accéder à l'interface graphique d'ArgoCD depuis votre navigateur, modifiez le service pour lui attribuer un Load Balancer AWS :

```bash
# Modification du service ArgoCD en type LoadBalancer
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'

# Installation de jq pour extraire les données JSON
yum install jq -y

# Récupération et affichage du nom DNS du Load Balancer
export ARGOCD_SERVER=$(kubectl get svc argocd-server -n argocd -o json | jq -r '.status.loadBalancer.ingress[0].hostname')
echo "URL ArgoCD : http://$ARGOCD_SERVER"
```

---

## 4. Récupération des Identifiants ArgoCD

Exécutez la commande ci-dessous pour récupérer le mot de passe initial généré par ArgoCD :

```bash
# Décodage du mot de passe administrateur
export ARGO_PWD=$(kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d)

echo ""
echo "=== Identifiants ArgoCD ==="
echo "Nom d'utilisateur : admin"
echo "Mot de passe       : $ARGO_PWD"
```

---

## 5. Configuration de l'Application dans ArgoCD

1. Ouvrez l'adresse de l'ELB dans votre navigateur (`http://$ARGOCD_SERVER`).
2. Connectez-vous avec l'identifiant **`admin`** et le mot de passe récupéré à l'étape précédente.
3. Cliquez sur **+ New App** et saisissez la configuration suivante :
   * **Application Name :** `hdfcbank` (ou `bankapp`)
   * **Project Name :** `default`
   * **Sync Policy :** `Automatic`
   * **Repository URL :** `https://github.com/ReyazShaik/ar-deploy.git`
   * **Path :** `./`
   * **Cluster URL :** `https://kubernetes.default.svc`
   * **Namespace :** `default`
4. Cliquez sur **Create**.

---

## 6. Vérification du Déploiement et Diagnostic Réseau

### 6.1. Vérification des ressources Kubernetes
Une fois l'application synchronisée, vérifiez que les pods et services fonctionnent correctement :

```bash
# Vérification des pods
kubectl get po

# Récupération de l'adresse du Load Balancer et du NodePort
kubectl get svc
```

*Exemple d'affichage :*
`mobilebankingservice LoadBalancer ... a2a0843c1e3fd4cd1af9cf61cba940ef-1651595843.us-east-1.elb.amazonaws.com 80:31371/TCP`

---

### 6.2. Résolution de l'erreur `ERR_CONNECTION_REFUSED` (Configuration AWS)

Si vous ne parvenez pas à accéder à l'application via l'URL du Load Balancer, le flux est probablement bloqué au niveau du **Security Group AWS** des nœuds Workers.

#### Correction via la Console AWS EC2 :
1. Connectez-vous à la console **AWS EC2** > **Security Groups**.
2. Sélectionnez le groupe de sécurité de vos nœuds Workers : **`nodes.djibril.k8s.local`** (`sg-0196bcd0b4c00e42e`).
3. Dans l'onglet **Inbound rules** (Règles d'entrée), cliquez sur **Edit inbound rules**.
4. Ajoutez une règle :
   * **Type :** Custom TCP
   * **Port Range :** Le port NodePort de votre service (ex: `31371` ou la plage `30000-32767`).
   * **Source :** `0.0.0.0/0` (ou le Security Group du Load Balancer `sg-01f021e2b20c167c1`).
5. Cliquez sur **Save rules**.

#### Correction via AWS CLI (alternative) :
```bash
aws ec2 authorize-security-group-ingress \
  --group-id sg-0196bcd0b4c00e42e \
  --protocol tcp \
  --port 31371 \
  --cidr 0.0.0.0/0
```

---

## 7. Accès Final à l'Application

Une fois la règle de sécurité enregistrée et le bilan de santé (*Health Check*) validé côté AWS :

1. Ouvrez votre navigateur.
2. Saisissez l'adresse **HTTP** (et non HTTPS) de votre service :
   `http://a2a0843c1e3fd4cd1af9cf61cba940ef-1651595843.us-east-1.elb.amazonaws.com`
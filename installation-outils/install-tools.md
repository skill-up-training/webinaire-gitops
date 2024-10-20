# Pour faire ce lab on va utiliser  2 serveurs

# Serveur 1 pour installer Rancher
sudo apt update && sudo apt upgrade -y
sudo apt install docker.io -y

sudo systemctl start docker
sudo systemctl enable docker
sudo docker run -d --restart=unless-stopped -p 80:80 -p 443:443 --privileged rancher/rancher:latest

# Pour se connecter à Rancher 
- Admin access to Rancher UI
docker logs  CONTAINER_ID  2>&1 | grep "Bootstrap Password:"
Recuperer l'ID du container qui tourne

# Serveur 2 installer Kubernetes et ArgoCD

### Étape 1 : Installer ArgoCD dans Kubernetes

1. **Installer ArgoCD dans le namespace `argocd`** :

   Tout d'abord, créez un namespace dédié pour ArgoCD, puis installez les composants ArgoCD dans ce namespace.

   ```bash
   kubectl create namespace argocd
   ```

2. **Installer les manifests d'ArgoCD** à partir du dépôt officiel :

   Utilisez le fichier YAML fourni par ArgoCD pour installer tous les composants nécessaires :

   ```bash
   kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
   ```

   Cela installera ArgoCD dans le namespace `argocd`. Vous pouvez vérifier l'état des pods avec la commande suivante :

   ```bash
   kubectl get pods -n argocd
   ```

### Étape 2 : Exposer ArgoCD avec un Service NodePort

Par défaut, ArgoCD crée un **Service de type ClusterIP**, ce qui signifie que l'interface n'est pas accessible depuis l'extérieur du cluster. Vous devez le changer en **NodePort** pour accéder à l'interface web ArgoCD via l'IP d'un de vos nœuds.

1. **Modifier le Service `argocd-server` en NodePort** :

   Vous pouvez modifier le type de Service `argocd-server` à l'aide de la commande `kubectl patch` :

   ```bash
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort"}}'
   ```

   Si vous souhaitez spécifier un NodePort particulier, vous pouvez le faire en ajoutant le port exact, par exemple :

   ```bash
   kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "NodePort", "ports": [{"port": 443, "targetPort": 8080, "nodePort": 32000}]}}'
   ```

   Cela exposera ArgoCD sur le port `32000` du nœud.

2. **Vérifier le Service NodePort** :

   Utilisez cette commande pour voir les détails du Service ArgoCD :

   ```bash
   kubectl get svc -n argocd
   ```

   Vous verrez que le service `argocd-server` est maintenant de type `NodePort` avec un port externe.

3. **Accéder à l'interface Web d'ArgoCD** :

   Une fois que le Service est configuré en NodePort, vous pouvez accéder à l'interface web d'ArgoCD via l'IP du nœud et le NodePort.

   Si l'IP du nœud est `192.168.1.100` et le NodePort est `32000`, vous pouvez accéder à l'interface ArgoCD en visitant :

   ```
   https://192.168.1.100:32000
   ```

   **Remarque** : Comme ArgoCD utilise HTTPS avec un certificat auto-signé, vous devrez probablement accepter le certificat non valide dans votre navigateur.

### Étape 3 : Récupérer le mot de passe d'ArgoCD

Par défaut, le mot de passe pour accéder à l'interface ArgoCD est le nom du Pod `argocd-server` qui a été généré.

1. Pour obtenir le mot de passe (qui est stocké dans un secret Kubernetes), exécutez la commande suivante :

   ```bash
   kubectl -n argocd get secret argocd-initial-admin-secret -o jsonpath="{.data.password}" | base64 -d
   ```

   Le nom d'utilisateur par défaut est `admin`.
# Créer une application dans argoCD en renseignant le projet 2
https://github.com/skill-up-training/webinaire-gitops-projet2.git


# Démonstration

## Objectif
Montrer les commandes de base sur un cluster Kubernetes.

## Etapes

#### 1. Vérifier l'état du cluster
```bash
kubectl get ns

kubectl get po -A

kubectl get nodes
kubectl top nodes

# Montrer les ressources d'API disponibles
kubectl api-resources
```

#### 2. Mettre en place du monitoring dans un shell séparé
```bash
# Monitoring
watch -d "kubectl get po,services,nodes"
# Ou
k9s
```

#### 3. Créer son espace dédié
```bash
kubectl create ns jpi -o=yaml --dry-run=client > jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Passe le contexte par défaut sur le namespace jpi
kubectl config set-context --current --namespace jpi

# Vérification
kubectl config get-contexts 
```

#### 4. Déployer un conteneur

```bash
kubectl run nginx --labels "app=nginx,project=formation-kubernetes" --image nginx --port 80 --dry-run=client -o=yaml >> jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre l'état dans le monitoring

kubectl get po -o wide
kubectl logs -f nginx
kubectl exec -it nginx -- bash

# Tester le Nginx via un port forward
kubectl port-forward pod/nginx 8080:80
curl http://127.0.0.1:8080
```

#### 4. Déployer plusieurs replicas du même pod

```bash

kubectl create deployment apache --image httpd:2.4.65 --port 80 --replicas=4 --dry-run=client -o=yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre l'état dans le monitoring

# Modifier l'image en faisant une typo "httppd:2.4.66
vim ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre la rolling upgrade qui reste bloqué

# Corriger la typo et changer l'image en 2.4.66
vim ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre la rolling upgrade qui fini en succès
```

#### 5. Déployer un pod avec une commande customisée

```bash
# On regarde le yaml avant de lancer la commande
kubectl run shell --image busybox --command sh --dry-run=client -o=yaml

# On crée le pod et on lance un shell dedans
kubectl run shell --rm -it --image busybox --  ls -ltr
# Quitter le shell
```

#### 6. Mettre à jour un pod

```bash
# Modifier le nom du conteneur nginx dans le pod Nginx
vim ./jpi.yaml
kubectl apply -f ./jpi.yaml

# La modification échoue, elle nécessite une recréation
kubectl delete po nginx
kubectl apply -f ./jpi.yaml

# Faire la même choses avec le déploiement apache
vim ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Ca marche! Le déploiement recrée le pod automatiquement.
```

#### 7. Nettoyage

```bash
kubectl delete -f ./jpi.yaml

# Suivre l'état dans le monitoring
``` 
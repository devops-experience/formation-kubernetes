# Démonstration

## Objectif
Montrer comment provisionner et gérer des volumes persistants.

## Etapes

#### 1. Vérifier l'état du cluster
```bash
kubectl get ns

kubectl get po -A

kubectl get nodes
kubectl top nodes
```

#### 2. Mettre en place du monitoring dans un shell séparé
```bash
# Monitoring
watch -d -n 1 "kubectl get po,pv,pvc,nodes -o wide"
# Ou
k9s
```

#### 3. Créer son espace dédié
```bash
kubectl create ns jpi -o=yaml --dry-run=client >> jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Passe le contexte par défaut sur le namespace jpi
kubectl config set-context --current --namespace jpi

# Vérification
kubectl config get-contexts 
```

#### 4. Explorer les StorageClass du cluster

```bash
# Lister les StorageClass
kubectl get storageclass

# Ouvrir la StorageClass par défaut
# Expliquer les champs provisioner, reclaimPolicy et volumeBindingMode 
kubectl edit storageclass standard-rwo

# Montrer les CSI drivers installés dans le cluster
# Préciser que ces pods sont uniquement la partie agent
# Les controlleurs sont installés sur le control plane géré par GCP
kubectl get ds -n kube-system | grep csi

# Faire le lien entre les CSI et les provisioners
```

#### 5. Créer un PVC et le rattacher a un Pod

```bash
# Créer un PVC
cat ./pvc.yaml >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Le PVC a été crée mais doit être affecté a un pod pour être provisionné
kubectl get pvc

# Créer un déploiement utilisant le PVC
cat ./app.yaml >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre sur le monitoring
# le provisionnement du PVC
# la création du pod

# Montrer le volume de 1 Gi monté dans le pod
k exec -it deploy/postgres -- lsblk

# Récupérer l'id du PV du PVC
PV=$(kubectl get pvc postgres-db -o jsonpath='{.spec.volumeName}')
kubectl describe pv $PV

# Montrer qu'a partir du volumeHandle, on arrive a retrouver le 
# disque dans GCP
# => https://console.cloud.google.com/compute/disks?project=quantum-theme-484321-i2
```

#### 6. Supprimer le PVC et nettoyer

```bash
kubectl delete -f ./jpi.yaml

# Suivre l'état dans le monitoring

# Le PV existe t'il toujours?
kubectl get pv

# Dû a la reclaimPolicy de la storageClass
kubectl get storageclass standard-rwo -o jsonpath='{.reclaimPolicy}'

# Vérifier l'état du disque dans GCP
# => https://console.cloud.google.com/compute/disks?project=quantum-theme-484321-i2
```

# Démonstration

## Objectif
Montrer comment configurer des applications dans Kubernetes.

## Etapes

#### 1. Créer son espace dédié
```bash
kubectl create ns jpi -o=yaml --dry-run=client > jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Passe le contexte par défaut sur le namespace jpi
kubectl config set-context --current --namespace jpi

# Vérification
kubectl config get-contexts 
```

#### 2. Lancer un monitoring
```bash
# Monitoring
watch -d "kubectl get deployment,po,cm,secrets"
```

#### 3. Créer des fichiers de configuration et les utiliser
```bash
# Créer un configmap
kubectl create configmap app-conf --from-literal=APP_MODE=demo --from-literal=APP_TIMEOUT=30 --from-file=app.conf=./app.conf --dry-run=client -o=yaml >> jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Utiliser certains champs du configmap dans un pod en envVar et monter le fichier dans un volume du pod
vim ./pod-cm.yaml
cat ./pod-cm.yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Tester le pod
kubectl logs demo-cm
```

#### 4. Créer des secrets et les utiliser
```bash
# Créer un secret 
kubectl create secret generic postgres --from-file=POSTGRES_PASSWORD=./postgres_password -o=yaml --dry-run=client >> jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Utiliser certains champs du secret dans un deploiement en envVar et monter le fichier dans un volume du pod
vim ./pod-secret.yaml
cat ./pod-secret.yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Tester le pod
kubectl logs -f deployment/postgres

# Attention, ne se met pas a jour en cas de nouveaux replicas
kubectl port-forward deployment/postgres 5432
psql -h 127.0.0.1 -d demo -U demo -W 
```

#### 5. Nettoyage

```bash
kubectl delete -f ./jpi.yaml

# Suivre l'état dans le monitoring
``` 

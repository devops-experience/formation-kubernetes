kubectl delete ns jpi# Démonstration

## Objectif
Montrer comment définir les ressources et les healthchecks des pods et leurs conséquences.

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
watch -d -n 1 "kubectl get deployment,po,services -o wide; helm list"
# Ou
k9s
```

#### 3. Créer son espace dédié
```bash
kubectl create ns jpi

# Passe le contexte par défaut sur le namespace jpi
kubectl config set-context --current --namespace jpi

# Vérification
kubectl config get-contexts 
```

#### 4. Déployer une application avec Helm

```bash
helm list -A

# Ajouter le repository de l'application
# => https://github.com/stefanprodan/podinfo
helm repo add podinfo https://stefanprodan.github.io/podinfo
helm repo update

# Consulter la liste des charts du repo
helm search repo podinfo

# Consulter le values du chart
helm show-values podinfo/podinfo

# Installation du frontend
helm upgrade --install --wait frontend podinfo/podinfo \
--namespace jpi \
--version 6.9.4 \
--set replicaCount=2 \
--set backend=http://backend-podinfo:9898/echo

# Des tests sont parfois intégrés au Helm chart et peuvent être utilisés
# pour valider le déploiement
helm test frontend --namespace jpi

# Tester l'ui
kubectl -n jpi port-forward deploy/frontend-podinfo 8080:9898
# => http://127.0.0.1:8080/

# regarder la console chromium
# Erreur car pas de service backend

# Installation du backend
helm upgrade --install --wait backend podinfo/podinfo \
--namespace jpi \
--version 6.9.4 \
--set redis.enabled=true

# Retester l'ui
kubectl -n jpi port-forward deploy/frontend-podinfo 8080:9898
# => http://127.0.0.1:8080/

# Le compteur ping s'incrémente
# Ca marche!
```

#### 5. Mettre à jour une application avec Helm

```bash
# Stocker la conf dans un fichier values

# Récupérer le chart pour l'analyser
helm fetch podinfo/podinfo --version 6.10.0
tar zxvf podinfo-6.10.0.tgz          
vim podinfo/values.yaml

# 1 fichier values par release
# Plus pratique que passer par des --set
# => IaC
# 1 changement dans backend: replicaCount: 2
vim backend.yaml
vim frontend.yaml

# Liste des versions disponibles du chart
helm search repo podinfo --versions

# Install d'un plugin de diff
helm plugin install https://github.com/databus23/helm-diff

# Vérifier ce que va faire l'upgrade
helm diff upgrade --install frontend podinfo/podinfo \
--namespace jpi \
--version 6.10.0 \
--values ./frontend.yaml

# Pas de modifications importantes

# Mettre à jour la release frontend
helm upgrade --install --wait frontend podinfo/podinfo \
--namespace jpi \
--version 6.10.0 \
--values ./frontend.yaml

# Pareil pour le backend

# Diff
helm diff upgrade --install backend podinfo/podinfo \
--namespace jpi \
--version 6.10.0 \
--values ./backend.yaml

# Upgrade
helm upgrade --install --wait backend podinfo/podinfo \
--namespace jpi \
--version 6.10.0 \
--values ./backend.yaml

# Retester l'ui
kubectl -n jpi port-forward deploy/frontend-podinfo 8080:9898
# => http://127.0.0.1:8080/

# Le compteur ping s'incrémente
# La version affichée est bien la v6.10.0
# Ca marche!
```

#### 6. Supprimer une application avec Helm

```bash
helm uninstall backend frontend
```

#### 7. Nettoyage

```bash
kubectl delete ns jpi

# Suivre l'état dans le monitoring
``` 
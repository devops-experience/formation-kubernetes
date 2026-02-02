# Démonstration

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
watch -d -n 1 "kubectl get po,services,nodes -o wide"
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

#### 4. Déployer des conteneurs aux ressources limitées

```bash
kubectl run nginx --image nginx --port 80 --dry-run=client -o=yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Expliquer le warning du mutator 
# Warning: autopilot-default-resources-mutator:Autopilot updated Pod jpi/nginx: defaulted unspecified 'cpu' resource for containers [nginx] (see http://g.co/gke/autopilot-defaults).


# Revue du pod qui va se faire OOMKill
vim ./stress.yaml

# Application
cat ./stress.yaml >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre l'état dans le monitoring et via cette commande
# Les métriques ne sont collectés que toutes les 15 secondes
# Echantillonage, donc pas en temps réel
# Pas de persistence
kubectl top pod

# Affichage de l'état détaillé du pod
# Regarder en particulier le champ "State" et "Last State"
kubectl describe po stress
kubectl get events
```

#### 5. Déployer des conteneurs avec des healthchecks liveness

```bash
# Revue du pod qui va ne plus être healthy
vim ./liveness.yaml

# Application
cat ./liveness.yaml >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre l'état dans le monitoring
# Le pod doit redémarré au bout de ~50 secondes
# Calcul:
# 30 secondes (suppression du fichier) +
# 15 secondes (failuretreshold * periodSeconds) +
# 5 (terminationGracePeriod)

# Suivre également les évenements du Pod liveness
# SIGKILL parce que pas de forward de signaux par Bash
kubectl get events \                             
  --watch \
  --field-selector involvedObject.kind=Pod,involvedObject.name=liveness

# Champ liveness et exitCode (137 == SIGKILL)
kubectl describe po liveness
```

#### 6. Déployer des conteneurs avec des healthchecks readiness

```bash
# Revue du déploiement et de son service
# On va charger le pod pour qu'il n'arrive plus a répondre
# Le healhcheck va alors échouer et l'enlever des endpoints
vim ./readiness.yaml

# Les précédents Pods doivent être passé en CrashLoopbackOff
# Cela veut dire qu'ils on atteint le nombre de redémarrage maximum sur une fenêtre de temps.

# Application
cat ./readiness.yaml >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Suivre l'état dans le monitoring

# Suivre également les évenements
kubectl get events -w

# Monitorer les endpoints du service
# (les pods sur lequel le service redirige le traffic)
# Quand le pod devient unhealthy son endpoint disparait
# il n'est plus routable
# A noter que le top n'est pas pratique car pas temps réel (échantillonage)
watch -d "kubectl top pods -l app=readiness; kubectl get po,endpoints -l app=readiness -o wide"

# Commencer a stresser l'app
# S'amuser a arrêter la boucle et redémarrer
# et observer le comportement des pods
kubectl run --rm netshoot -it --image nicolaka/netshoot -- sh

while true
do
  curl http://readiness:5000 &
  sleep 1;
done
```

#### 7. Nettoyage

```bash
kubectl delete -f ./jpi.yaml

# Suivre l'état dans le monitoring
``` 

# Travaux pratiques

## Objectif
Comprendre comment configurer correctement un Pod.

## Documentation (rappel)
- Référence rapide: https://kubernetes.io/docs/reference/kubectl/quick-reference/#viewing-and-finding-resources
- Documentation API: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.33/#podspec-v1-core

## Etapes

#### 1. Récupérer le kubeconfig du cluster
- Le lien vous a été envoyé par le formateur sur Teams.
- Vous devez avoir au moins un client installé sur votre machine
- Pour "kubectl", la variable KUBECONFIG doit pointer sur le kubeconfig qui vous a été transmis:
```bash
$env:KUBECONFIG = "<CHEMIN VERS VOTRE KUBECONFIG>"
kubectl.exe cluster-info
```

#### 2. Explorer le cluster
- Explorer le contenu du cluster
```bash
kubectl.exe get ns

kubectl.exe get po -A

kubectl.exe get nodes
kubectl.exe top nodes
```

#### 3. Créer son espace dédié
- Créer un Namespace avec vos initiales
```bash
$NS = "$VOS_INITIALES"

# Créer le Namespace
kubectl.exe create ns $NS -o=yaml --dry-run=client > tp4.yaml
echo "---" >> ./tp4.yaml
kubectl.exe apply -f ./tp4.yaml

# Passe le contexte par défaut sur votre namespace
kubectl.exe config set-context --current --namespace $NS

# Vérification
kubectl.exe config get-contexts
```

#### 4. Mettre en place du monitoring dans un shell séparé
- Mettre en place le monitoring
```bash
# Sous Linux
watch -d "kubectl get po,deployment,service -o wide"

# Dans Powershell
while ($true) {
    Clear-Host
    kubectl get po,deployment,service -o wide
    Start-Sleep -Seconds 2
}

# ou
k9s
# ou
lens
# ou
headlamp
```

#### 5. Déployer des conteneurs aux ressources limitées

- Récupérer le dépôt Git
```bash
# Ou télécharger directement l'archive depuis github si vous n'avez pas git
git clone "https://github.com/devops-experience/formation-kubernetes.git"
cd "formation-kubernetes/4-garantir-et-maitriser-mes-conteneurs"
```

- Déployer un conteneur sans ressources définies
```bash
kubectl.exe run nginx --image nginx --port 80 --dry-run=client -o=yaml >> tp4.yaml
echo "---" >> ./tp4.yaml
kubectl.exe apply -f ./tp4.yaml
```

- Vérifier le manifeste réellement appliqué dans le cluster
```bash
kubectl.exe get po nginx -o yaml
```
 
- Des ressources par défaut ont été définies automatiquement par un composant du cluster

- Lire le fichier `stress.yaml`
```bash
notepad.exe stress.yaml
```

- Appliquer le contenu du fichier

```bash
# Ajouter le fichier dans votre yaml à appliquer
gc ./stress.yaml | ac ./tp4.yaml
echo "---" >> ./tp4.yaml
# Appliquer les nouveaux objets
kubectl.exe apply -f ./tp4.yaml
```

- Suivre l'état du pod dans le monitoring jusqu'à l'OOMKill

- Investiguer le pod
```bash
# Etat
# Regarder en particulier le champ "State" et "Last State"
kubectl.exe describe po stress
```

#### 6. Déployer des conteneurs avec des healthchecks liveness
- Lire le fichier `liveness.yaml`
```bash
notepad.exe liveness.yaml
```

- Appliquer le contenu du fichier

```bash
# Ajouter le fichier dans votre yaml à appliquer
gc ./liveness.yaml | ac ./tp4.yaml
echo "---" >> ./tp4.yaml
# Appliquer les nouveaux objets
kubectl.exe apply -f ./tp4.yaml
```

- Suivre l'état du pod jusqu'au redémarrage, au bout de 5 minutes , on passe a l'état "crashloopbackoff"

- Investiguer le pod
```bash
# Etat
# Regarder en particulier les champs "State" et "Last State" et events
kubectl.exe describe po liveness
```

- On peut aussi voir les événements séparemment

```bash
kubectl.exe get events \                             
  --watch \
  --field-selector involvedObject.kind=Pod,involvedObject.name=liveness
```

#### 7. Déployer des conteneurs avec des healtchecks readiness
- Lire le fichier `readiness.yaml`
```bash
notepad.exe readiness.yaml
```

- Appliquer le contenu du fichier

```bash
# Ajouter le fichier dans votre yaml à appliquer
gc ./readiness.yaml | ac ./tp4.yaml
echo "---" >> ./tp4.yaml
# Appliquer les nouveaux objets
kubectl.exe apply -f ./tp4.yaml
```

- Ouvrir un autre shell et lancer un monitoring
```bash
# Linux
watch -d "kubectl get po,endpoints -l app=readiness -o wide"

# OU

# Windows
# Dans Powershell
while ($true) {
    Clear-Host
    kubectl.exe kubectl get po,endpoints -l app=readiness -o wide
    Start-Sleep -Seconds 1
}
```

- Saturer le pod de requêtes
```bash
kubectl.exe run --rm netshoot -it --image nicolaka/netshoot -- sh
# Dans le shell du pod
while true
do
  curl http://readiness:5000 &
  sleep 2;
done
```

- Regarder les pods devenir "Unhealthy" et être retirer des endpoints du service

- Investiguer les erreurs de healthchecks dans les évenements
```
kubectl events -w
```

#### 8. Nettoyage
- Nettoyer le cluster des objets crées
```bash
kubectl.exe delete -f ./tp4.yaml

# Suivre l'état dans le monitoring
```

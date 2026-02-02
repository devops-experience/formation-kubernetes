# Travaux pratiques

## Objectif
Gérer une application avec Helm.

## Documentation (rappel)
- Référence rapide: https://kubernetes.io/docs/reference/kubectl/quick-reference/#viewing-and-finding-resources
- Documentation API: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.33/#podspec-v1-core

## Etapes

#### 1. Créer son espace dédié
- Créer un Namespace avec vos initiales
```bash
$NS = "$VOS_INITIALES"

# Créer le Namespace
kubectl.exe create ns $NS

# Passe le contexte par défaut sur votre namespace
kubectl.exe config set-context --current --namespace $NS

# Vérification
kubectl.exe config get-contexts
```

#### 2. Mettre en place du monitoring dans un shell séparé
- Mettre en place le monitoring
```bash
# Sous Linux
watch -d -n 1 "kubectl get deployment,po,services -o wide; helm list"

# Dans Powershell
while ($true) {
    Clear-Host
    kubectl get po,deployment,service -o wide
    helm list
    Start-Sleep -Seconds 2
}

# ou
k9s
# ou
lens
# ou
headlamp
```

#### 5. Déployer une application avec Helm

- Récupérer le dépôt Git
```bash
# Ou télécharger directement l'archive depuis github si vous n'avez pas git
git clone "https://github.com/devops-experience/formation-kubernetes.git"
cd "formation-kubernetes/6-installer-applications-tierces"
```

- Ajouter le dépôt Helm podinfo (https://github.com/stefanprodan/podinfo)
```bash
helm.exe repo add podinfo https://stefanprodan.github.io/podinfo
helm.exe repo update
```

- Installer le frontend et suivre le déploiement sur le monitoring
```bash
helm.exe upgrade --install --wait frontend podinfo/podinfo \
--version 6.9.4 \
--set replicaCount=2 \
--set backend=http://backend-podinfo:9898/echo
```
 
- Lancer les tests intégrés au chart
```bash
helm.exe test frontend --namespace jpi
```

- Déployer le backend et suivre le déploiement sur le monitoring
```bash
helm.exe upgrade --install --wait backend podinfo/podinfo \
--version 6.9.4 \
--set redis.enabled=true
```

- Tester l'ui (Cliquer sur le bouton ping, vérifier la version affichée)

```bash
kubectl.exe port-forward deploy/frontend-podinfo 8080:9898
```

#### 6. Mettre à jour une application avec Helm

- Analyser le fichier de configuration du chart
```bash
helm.exe fetch podinfo/podinfo --version 6.10.0 --untar
notepad.exe podinfo/values.yaml
```

- On a crée deux versions de ce fichier pour les 2 releases à déployer.
```bash
notepad.exe backend.yaml
notepad.exe frontend.yaml
```

- On cherche la dernière version du chart.
```bash
helm.exe search repo podinfo --versions
```

- On installe le plugin de diff de Helm qui permet d'afficher un diff entre la version déployée et celle que l'on souhaite appliquer (très pratique et rassurant).
```bash
helm.exe plugin install https://github.com/databus23/helm-diff
```

- Mettre à jour le frontend et suivre le déploiement sur le monitoring
```bash
helm.exe diff upgrade --install frontend podinfo/podinfo \
--version 6.10.0 \
--values ./frontend.yaml

# Diff correct on maj

helm.exe upgrade --install --wait frontend podinfo/podinfo \
--version 6.10.0 \
--values ./frontend.yaml
```

- Mettre à jour le backend et suivre le déploiement sur le monitoring
```bash
helm.exe diff upgrade --install backend podinfo/podinfo \
--version 6.10.0 \
--values ./backend.yaml

# Diff correct on maj

helm.exe upgrade --install --wait backend podinfo/podinfo \
--version 6.10.0 \
--values ./backend.yaml
```

- On teste l'application
```bash
kubectl.exe port-forward deploy/frontend-podinfo 8080:9898
```

- Ouvrir un navigateur a l'url http://127.0.0.1:8080
  - Vérifier la version affichée
  - Vérifier que le bouton ping incrémente bien le compteur

#### 7. Supprimer l'application déployée avec Helm
- Supprimer l'application
```bash
helm.exe uninstall backend frontend
```

#### 8. A toi de pratiquer!
- Installer 'kube-view' (https://github.com/benc-uk/kubeview/tree/main/deploy/helm), un visualisateur graphique de cluster Kubernetes en respectant la conf suivante:
  - version du chart a installer: 2.0.5
  - kubeview doit être en mode "singleNamespace"
  - kubeview doit être installé dans votre namespace nominatif
  - Le service de Kubeview doit être de type ClusterIP
  - Le nombre de replicas kubeview doit être égal à 2
- Tester votre installation 'kube-view' avec la commande "kubectl.exe port-forward".

#### 9. Nettoyage

- Appliquer le contenu du fichier

```bash
kubectl delete ns $NS
```
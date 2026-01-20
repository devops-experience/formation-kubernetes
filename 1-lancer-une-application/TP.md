# Démonstration

## Objectif
Faire vos premiers pas dans Kubernetes.

## Documentation (rappel)
Référence rapide: https://kubernetes.io/docs/reference/kubectl/quick-reference/#viewing-and-finding-resources
Documentation API: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.33/#podspec-v1-core

## Etapes

#### 1. Récupérer le kubeconfig du cluster
Le lien vous a été envoyé par le formateur.

#### 2. Explorer le cluster
- Explorer le contenu du cluster
```bash
kubectl get ns

kubectl get po -A

kubectl get nodes
kubectl top nodes

kubectl api-resources
```

#### 3. Créer son espace dédié
- Créer un Namespace avec vos initiales
```bash
NS="$VOS_INITIALES"

# Créer le Namespace
kubectl create ns $VOS_INITIALES -o=yaml --dry-run=client > tp1.yaml
echo "---" >> ./tp1.yaml
kubectl apply -f ./tp1.yaml

# Passe le contexte par défaut sur votre namespace
kubectl config set-context --current --namespace $NS

# Vérification
kubectl config get-contexts 
```

#### 4. Mettre en place du monitoring dans un shell séparé
- Mettre en place le monitoring
```bash
# Monitoring
watch -d "kubectl get po,deployment -o wide"
# ou 
k9s
# ou
lens
# ou
headlamp
```

#### 5. Déployer mon premier conteneur
- Créer un pod Nginx
```bash
kubectl run nginx --image nginx --port 80 --dry-run=client -o=yaml >> tp1.yaml
echo "---" >> ./tp1.yaml
kubectl apply -f ./tp1.yaml
```

- Suivre l'état dans le monitoring

- Investiguer le pod crée
```bash
# Etat
kubectl describe po nginx
# Logs
kubectl logs -f nginx
# Shell
kubectl exec -it nginx -- bash
```

- Tester le Nginx via un port forward
```bash
# Port-forward sur localhost:8080
kubectl port-forward pod/nginx 8080:80
curl http://127.0.0.1:8080
```

#### 6. Déployer plusieurs réplicas d'un même pod
- Créer un déploiement appellé "webserver" déployant 3 réplicas de Nginx (image: nginx:latest) exposant le port 80
```bash
kubectl create deployment webserver --image nginx:latest --port 80 --replicas=3 --dry-run=client -o=yaml >> ./tp1.yaml
echo "---" >> ./tp1.yaml
kubectl apply -f ./tp1.yaml
```

- Suivre l'état dans le monitoring

#### 7. A toi de pratiquer!
- Créer un Pod echo avec les caractéristiques suivantes:
    - image: hashicorp/http-echo:1.0.0
    - nom du conteneur: echo
    - nom du pod: hashi-echo
    - port exposé: 5678
- Tester ce que le port 5678 (via shell ou port-forward)
- Créer un déploiement avec 1 réplica en partant des specifications du Pod que vous venez de créer
- Modifier le nombre de réplicas à 4

#### 8. Nettoyage
- Nettoyer le cluster des objets crées
```bash
kubectl delete -f ./tp1.yaml

# Suivre l'état dans le monitoring
``` 
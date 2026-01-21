# Démonstration

## Objectif
Configurer des conteneurs dans Kubernetes.

## Documentation (rappel)
- Référence rapide: https://kubernetes.io/docs/reference/kubectl/quick-reference/#viewing-and-finding-resources
- Documentation API: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.35/#podspec-v1-core

## Etapes

#### 1. Créer son espace dédié
- Créer un Namespace avec vos initiales
```bash
$NS = "$VOS_INITIALES"

# Créer le Namespace
kubectl.exe create ns $NS -o=yaml --dry-run=client > tp2.yaml
echo "---" >> ./tp2.yaml
kubectl.exe apply -f ./tp2.yaml

# Passe le contexte par défaut sur votre namespace
kubectl.exe config set-context --current --namespace $NS

# Vérification
kubectl.exe config get-contexts 
```

#### 2. Mettre en place du monitoring dans un shell séparé
- Mettre en place le monitoring
```bash
# Sous Linux
watch -d "kubectl get po,deployment,cm,secret -o wide"
# ou 
k9s
# ou
lens
# ou
headlamp
```

#### 3. Créer une configuration et utilise la
- Créer une ConfigMap
```bash
kubectl.exe create configmap app-conf --from-literal=APP_MODE=demo --from-literal=APP_TIMEOUT=30 --from-file=app.conf=./app.conf --dry-run -o=yaml >> tp2.yaml
echo "---" >> ./tp2.yaml
kubectl.exe apply -f ./tp2.yaml
```
- Utiliser certains champs du configmap dans un pod en envVar et monter le fichier dans un volume du pod
```bash
vim ./pod-cm.yaml
cat ./pod-cm.yaml >> ./tp2.yaml
echo "---" >> ./tp2.yaml
kubectl.exe apply -f ./tp2.yaml
```

- Tester le pod
```bash
kubectl.exe logs demo-cm
```

#### 4. Créer un secret et utilise le
- Créer un secret 
```bash
kubectl.exe create secret generic postgres --from-file=POSTGRES_PASSWORD=./postgres_password -o=yaml --dry-run=client >> tp2.yaml
echo "---" >> ./tp2.yaml
kubectl.exe apply -f ./tp2.yaml
```

- Utiliser certains champs du secret dans un deploiement en envVar et monter le fichier dans un volume du pod
```bash
vim ./pod-secret.yaml
cat ./pod-secret.yaml >> ./tp2.yaml
echo "---" >> ./tp2.yaml
kubectl.exe apply -f ./tp2.yaml
```

- Tester le pod
```bash
kubectl.exe logs -f deployment/postgres

# Attention, ne se met pas a jour en cas de nouveaux replicas
kubectl.exe port-forward deployment/postgres 5432

# Utiliser le mot de passe du fichier postgres_password
psql -h 127.0.0.1 -d demo -U demo -W 
```

#### 5. A toi de pratiquer!
- Créer 2 ConfigMaps
    - `atdp-vars` contenant:
        - APP_ENV=ppr
        - APP_LOG_LEVEL=info
    - `atdp-conf` contenant le fichier "atdp/app.conf"
- Créer un secret `atdp-secret` contenant le fichier `atdp/secret` avec comme clé `SECRET1`.
- Créer un pod à partir du fichier "atdp/pod-pratique.yaml" dans le répertoire du TP en lui rajoutant:
    - Les variables d'environnements APP_ENV et APP_LOG_LEVEL montées depuis la configmap `atdp-vars`
    - Le fichier `app.conf` monté en tant que volume depuis la configmap `atdp-conf`.
    - Le secret `atdp-secret` monté en tant que volume dans le répertoire `/run/secret/atdp`
- Vérifier les logs de l'application, les variables d'environnement et les fichiers montés
- Modifier la valeur de `APP_ENV` dans la configmap `atdp-vars` en `dev`.
- Vérifier dans le pod si la variable d'environnement a été mise à jour.
- Recréer le pod.
- Vérifier à nouveau la valeur de la variable `APP_ENV`.

#### 6. Nettoyage

```bash
kubectl.exe delete -f ./tp2.yaml

# Suivre l'état dans le monitoring
``` 

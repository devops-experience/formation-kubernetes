# Démonstration

## Objectif
Rendre une application HTTP routable depuis l'extérieur du cluster.

## Documentation (rappel)
- Référence rapide: https://kubernetes.io/docs/reference/kubectl/quick-reference/#viewing-and-finding-resources
- Documentation API: https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.35/#podspec-v1-core

## Etapes

#### 1. Créer son espace dédié
- Créer un Namespace avec vos initiales
```bash
$NS = "$VOS_INITIALES"

# Créer le Namespace
kubectl.exe create ns $NS -o=yaml --dry-run=client > tp3.yaml
echo "---" >> ./tp3.yaml
kubectl.exe apply -f ./tp3.yaml

# Passe le contexte par défaut sur votre namespace
kubectl.exe config set-context --current --namespace $NS

# Vérification
kubectl.exe config get-contexts 
```

#### 2. Lancer un monitoring
- Lancer un monitoring dans un shell dédié ou dans une interface graphique
```bash
# Monitoring
watch -d "kubectl get deployment,service,ingress -o wide"
# ou 
k9s
# ou
lens
# ou
headlamp
```

#### 3. Déployer une application et l'exposer
- Déployer l'application whoami qui affiche des informations minimales sur l'OS du pod
```bash
kubectl.exe create deployment --replicas=3 whoami --image traefik/whoami:v1.11.0 --dry-run=client -o=yaml >> ./tp3.yaml
echo "---" >> ./tp3.yaml
kubectl.exe apply -f ./tp3.yaml
```

- Exposer le service
```bash
kubectl.exe expose deployment whoami --port=8080 --target-port=80 --name whoami --dry-run=client -o=yaml >> ./tp3.yaml
echo "---" >> ./tp3.yaml
kubectl.exe apply -f ./tp3.yaml
```

- Tester le service
```bash
# Ne pas tester avec port-forward qui n'utilise le service que pour sélectionner un pod
# Lancer d'abord cette commande
kubectl.exe run -it --rm checker --image=curlimages/curl:latest --command -- sh 
# Dans le shell, lancer cette commande 
while true; do curl --silent http://whoami:8080 | grep "IP: 10";sleep 1;done
# Puis arrêter le shell
exit
```

#### 4. Exposer l'application à l'extérieur du cluster
- Investiguer l'ingress controleur Nginx
```bash
kubectl.exe get po,service -n ingress-nginx -o wide
# Une entrée A *.onati.devops-experience.com redirige sur le LB du Nginx ingress controleur
kubectl.exe run -it --rm --image alpine -- sh
apk add bind-tools
dig test.onati.devops-experience.com
exit
```
- Exposer l'application via le reverse-proxy du cluster
!!!!! Utiliser vos initiales pour le nom
```bash
NOM="$VOS_INITIALES"
kubectl.exe create ingress whoami --class=nginx --rule="$NOM.onati.devops-experience.com/=whoami:8080" --dry-run=client -o=yaml >> ./tp3.yaml
echo "---" >> ./tp3.yaml
kubectl.exe apply -f ./tp3.yaml
```

- Tester le service
```bash
kubectl.exe run -it --rm --image=curlimages/curl:latest -- sh
curl --silent http://$NOM.onati.devops-experience.com

while true
do
    # On voit bien le load balancing en round robin
    curl --silent http:/$NOM.onati.devops-experience.com | grep "IP: 10"
    sleep 1
done
exit
```

#### 5. A toi de pratiquer!
- Créer 2 ConfigMaps:
    - `atdp-nginx1` contenant une entrée:
        - `index.html` = "version1"
    - `atdp-nginx2` contenant une entrée:
        - `index.html` = "version2"
- Créer 2 Deployments:
    - `nginx-1`:
        - image: nginx:latest
        - selector et labels:
            - app=nginx
        - la configmap `atdp-nginx1` montée sur le chemin `/usr/share/nginx/html/index.html`
        - replicas: 2
    - `nginx-2`:
        - image: nginx:latest
        - selector et labels:
            - app=nginx
        - la configmap `atdp-nginx2` montée sur le chemin `/usr/share/nginx/html/index.html`
        - replicas: 4
- Créer un Service de type `ClusterIP` avec comme selector:
    - app=nginx
- Créer un Ingress exposant le service précedemment crée sur une url: http://$NOM.nginx.onati.devops-experience.com
- Curler 12 fois cette url.
    - Que voyez-vous?

#### 6. Nettoyage
- Nettoyer votre espace
```bash
kubectl.exe delete -f ./tp3.yaml

# Suivre l'état dans le monitoring
```

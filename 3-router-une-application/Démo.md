# Démonstration

## Objectif
Montrer comment router du trafic entrant vers une application dans un cluster Kubernetes.

## Etapes

#### 1. Créer son espace dédié
```bash
kubectl create ns jpi -o=yaml --dry-run=client > ./jpi.yaml
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
watch -d "kubectl get service,ingress"
```

#### 3. Déployer une application et l'exposer
```bash
# Déployer
kubectl create deployment --replicas=3 whoami --image traefik/whoami:v1.11.0 --dry-run=client -o=yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Exposer le service
kubectl expose deployment whoami --port=8080 --target-port=80 --name whoami --dry-run=client -o=yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Tester
# Ne pas tester avec port-forward qui n'utilise le service que pour sélectionner un pod
kubectl run -it --rm checker --image=curlimages/curl:latest --command -- sh 
while true; do curl --silent http://whoami:8080 | grep "IP: 10";sleep 1;done
```

#### 4. Exposer l'application à l'extérieur du cluster
```bash
# Montrer le reverse-proxy du cluster
kubectl get po,service -n ingress-nginx

# Exposer l'application via le reverse-proxy du cluster
kubectl create ingress whoami --class=nginx --rule="whoami.onati.devops-experience.com/=whoami:8080" --dry-run=client -o=yaml >> ./jpi.yaml
echo "---" >> ./jpi.yaml
kubectl apply -f ./jpi.yaml

# Tester le service
curl --silent http://whoami.onati.devops-experience.com

while true
do
    curl --silent http://whoami.onati.devops-experience.com | grep "IP: 10"
    sleep 1
done
```

#### 5. Nettoyage

```bash
kubectl delete -f ./jpi.yaml

# Suivre l'état dans le monitoring
```
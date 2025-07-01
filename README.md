# TP KUBERNETES PARTIE 2
## I - Installation ArgoCD
kubectl create namespace argocd
kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml
argocd admin initial-password -n argocd
kubectl port-forward --address 0.0.0.0 svc/argocd-server -n argocd 8080:443
kubectl patch svc argocd-server -n argocd -p '{"spec": {"type": "LoadBalancer"}}'
https://192.168.1.139:8080/settings/clusters

## II – Déploiement de l’application ArgoCD
Installation d’un Ingress Controller via ces commandes suivantes :
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.10.1/deploy/static/provider/cloud/deploy.yaml
kubectl -n ingress-nginx get svc

## III - Création d’une application ArgoCD
Création du fichier ingress.yaml :
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
name: hello-ingress
annotations:
nginx.ingress.kubernetes.io/rewrite-target: /
spec:
rules:
- host: hello.local
http:
paths:
- path: /
pathType: Prefix
backend:
service:
name: hello-kubernetes
port:
number: 80
deployment :

Ajout d’une entrée dans le fichier /etc/hosts :
<IP-ingress> hello.local -> 10.100.204.44 hello.local


Création du fichier hello-argo.yaml :
apiVersion: argoproj.io/v1alpha1
kind: Application
metadata:
name: hello-kubernetes
namespace: argocd
spec:
project: default
source:
repoURL: https://github.com/paulbouwer/hello-kubernetes
path: .
targetRevision: HEAD
destination:
server: https://kubernetes.default.svc
namespace: default
syncPolicy:
automated: {} # Pour sync auto (à désactiver pour test manuel)
ingress:
enabled: true

## IV - Déploiement de l’application ArgoCD
Déploiement de l’application via cette commande : kubectl apply -f hello-argo.yaml
URL d’accès : http://a13517d9cabcc42a2911de6da23f02c1-977322706.us-east-1.elb.amazonaws.com/
Test de scalabilité de Kubernetes :
Pour vérifier si cela fonctionne, il faut porter une attention particulière au pod qui change après chaque rechargement de la page Web :

## V - Test de résilience Argo CD
Test n°1 : Suppression manuelle d’un Pod (auto-heal)
Argo CD détecte le pod manquant et Kubernetes recrée automatiquement le pod

Test n°2 : Suppression manuelle d’un Service (sync automatique)
Nous voyons que le service SVC est recréé automatiquement

Test n°3 : Suppression manuelle d’un Service (sync manuel)

Affichage d’ArgoCD après synchronisation manuelle :

## VI - Déploiement d'une image depuis un Repo Privé
On commence par cloner le repository
On se connecte ensuite à notre compte Docker Hub
On va ensuite créer l'image privée C poussée sur Docker Hub :
On fait la création du secret Kubernetes regcred pour autoriser le cluster à tirer l’image privée

## VII - Service MESH avec ISTIO
On télécharge l'application

Déploiment de l'application
On applique le manifeste Bookinfo et configure Gateway + VirtualService
On a ici la preuve que l’application Bookinfo fonctionne via l’URL du LoadBalancer avec ces différentes versions et varations.
ISTIO permet de contrôler les requêtes circulant entre les micro-services et peut aussi chiffrer les requêtes.

## VIII - Illustration HPA
On a déployé un HPA pour php-apache, qui scale automatiquement nos pods en réponse à la charge CPU. Puis on a atteint jusqu’à 9 réplicas, ce qui prouve que l’autoscaling fonctionne parfaitement.
Déploiement initial :
Mise en place du HPA :
Scaling des pods sous charge :

## IX - Application onlineboutique via ArgoCD
L’application ArgoCD est créée et synchronisée :
Ingress configurés :
hello-kubernetes → aws.cyrion automobile.fr onlineboutique → ELB AWS
Preuve des hôtes et ports configurés :

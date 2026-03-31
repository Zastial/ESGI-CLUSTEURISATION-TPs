# TP — Setup Minikube

Author: Alexandre CAROL

## Questions

### Partie 2.2

| Question | Réponse |
|----------|---------|
| Quel est le nom du contexte courant ? | Minikube. |
| Combien de nœuds voyez-vous ? | Il n'y a qu'un seul nœud car c'est un cluster local. |

### Partie 3.3

| Question | Réponse |
|----------|---------|
| Quelle image exacte est utilisée ? | nginx:1.27. |
| Quel évènement confirme le téléchargement et le démarrage ? | Les évènements Pulled, Created et Started confirment que l'image a été téléchargée et le conteneur démarré. |

### Partie 4.2

| Question | Réponse |
|----------|---------|
| À quoi sert un Service dans Kubernetes ? | Un Service permet d'accéder aux pods de manière stable car il garde la même adresse même si les pods changent. |
| Pourquoi le Service n'expose-t-il pas une IP publique en local ? | Parce que ClusterIP est interne au cluster, il faut utiliser minikube service pour y accéder. |

### Partie 5.2

| Question | Réponse |
|----------|---------|
| Que se passe-t-il au niveau des pods lors d'un changement d'image ? | Un nouveau pod est créé avec la nouvelle image et l'ancien est arrêté progressivement. |
| Qu'est-ce qu'un "rollout" et pourquoi est-ce utile ? | C'est la mise à jour progressive d'une application, c'est utile pour éviter les coupures et pouvoir revenir en arrière si ça se passe mal. |

---

## Compte-rendu d'exécution

### Commandes de vérification

```bash
# Vérifier le cluster
kubectl config current-context
kubectl cluster-info
kubectl get nodes

# Créer le namespace
kubectl create namespace tp-minikube

# Appliquer le Deployment
kubectl apply -f deployment.yaml -n tp-minikube
kubectl get deployments -n tp-minikube
kubectl get pods -n tp-minikube

# Appliquer le Service
kubectl apply -f service.yaml -n tp-minikube
kubectl get svc -n tp-minikube

# Accéder au service
minikube service web-nginx-svc -n tp-minikube --url
```

**Exposition du Service :** Via `ClusterIP` interne (par défaut), puis accès via `minikube service web-nginx-svc -n tp-minikube --url` qui expose un proxy local.

**Difficultés rencontrées :**
1. *Service non accessible en local* – Le `ClusterIP` n'expose pas d'IP publique. **Solution :** Utiliser `minikube service` pour créer un tunnel automatique vers le pod.
2. *Image nginx non téléchargée au démarrage du pod* – Délai de pull d'image. **Solution :** Attendre avec `kubectl rollout status` ou vérifier les événements via `kubectl describe pod`.


# TP Déploiement d’une image depuis un registre privé sur le cluster Kubernetes

Author: Alexandre CAROL

# Livrables

`docker images | grep tp-docker-hub-app`

![docker images | grep tp-docker-hub-app](assets/image.png)

`kubectl get pods -o wide`

![kubectl get pods -o wide](assets/image-1.png)

Acces web via port-forward et page affichee.

![port-forward](assets/image-2.png)

![port-forward-result](assets/image-3.png)

### Réponses courtes :

- **3 optimisations mises en place dans le Dockerfile et pourquoi :**
Utilisation d'un build multi-stage, copie seulement des fichiers utiles et activation de `NODE_ENV=production` pour avoir une image plus légère.

- **À quoi sert imagePullSecrets et comment vous l'avez configuré :** imagePullSecrets permet a Kubernetes de pull une image privée et il est configureé avec le secret `regcred` créé avec `kubectl create secret docker-registry`.

## Bonus

### Double replicas :
![replicas-2](assets/image-4.png)


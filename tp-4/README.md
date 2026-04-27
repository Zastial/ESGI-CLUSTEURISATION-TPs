Questions

Question 1 - Différence entre Secret et ConfigMap
 ConfigMap sert à stocker la configuration non sensible (host, port, options). Secret sert à stocker les informations sensibles comme les mots de passe et les clés.

Question 2 - Pourquoi MySQL a 1 replica et l'application 2
 MySQL garde des données, donc plusieurs replicas sans réplication propre peuvent causer des conflits. L'application web est stateless, donc on peut la dupliquer plus facilement.

Question 3 - Que se passe-t-il si on supprime le PVC
 Si on supprime le PVC puis on recrée le pod MySQL, les données sont perdues. Cela montre que le PVC est indispensable pour la persistance.

Question 4 - Rôle de podAntiAffinity
 podAntiAffinity évite de placer des pods similaires sur le même noeud. Sans cette règle, une panne d'un seul noeud peut faire tomber plusieurs replicas.

Question 5 - Différence entre ClusterIP, NodePort et LoadBalancer
 ClusterIP est accessible seulement dans le cluster. NodePort ouvre un port sur chaque noeud, et LoadBalancer expose le service vers l'extérieur avec une IP publique.

Question 6 - Pourquoi imagePullPolicy Never en local
 En local, l'image est souvent déjà présente dans Minikube et n'existe pas dans un registre distant. En production, on récupère normalement les images depuis un registre, donc Never n'est pas adapté.

Question 7 - Différence entre HPA et VPA
 HPA change le nombre de pods selon la charge. VPA change les ressources CPU/RAM d un pod sans changer le nombre de pods.

Question 8 - Pourquoi le scale down est plus lent
 Le scale up est rapide pour absorber un pic de trafic. Le scale down est volontairement plus lent pour éviter les changements trop fréquents.

Question 9 - Pourquoi StatefulSet est mieux pour une base répliquée
 StatefulSet donne une identité stable et un volume dédié à chaque pod, ce qui est important pour une base. Deployment est surtout adapté aux applications stateless.
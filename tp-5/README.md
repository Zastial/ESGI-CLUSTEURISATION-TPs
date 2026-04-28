# TP-5 Kubernetes

Ce TP déploie une application de démo sur 3 environnements (dev, staging, prod) avec isolation par namespace, RBAC, quotas, Pod Security Standards et NetworkPolicy.

## Arborescence

```
tp-5/
	k8s/
		base/
			deployment.yaml
			service.yaml
			kustomization.yaml
		overlays/
			dev/
				namespace-dev.yaml
				rbac.yaml
				kustomization.yaml
			staging/
				namespace-staging.yaml
				rbac.yaml
				kustomization.yaml
			prod/
				namespace-prod.yaml
				rbac.yaml
				kustomization.yaml
		quotas.yaml
		pss-labels.yaml
		networkpolicy.yaml
```

## Pré-requis

- kubectl configuré sur ton cluster (ex: minikube)
- contexte kubectl valide

## Installation (ordre recommandé)

Depuis le dossier `tp-5`:

```bash
# 1) Créer/mettre à jour les namespaces
kubectl apply -f k8s/overlays/dev/namespace-dev.yaml
kubectl apply -f k8s/overlays/staging/namespace-staging.yaml
kubectl apply -f k8s/overlays/prod/namespace-prod.yaml

# 2) Appliquer les labels Pod Security Standards
kubectl apply -f k8s/pss-labels.yaml

# 3) Appliquer quotas/limites (namespace app-dev)
kubectl apply -f k8s/quotas.yaml

# 4) Appliquer RBAC par environnement
kubectl apply -f k8s/overlays/dev/rbac.yaml
kubectl apply -f k8s/overlays/staging/rbac.yaml
kubectl apply -f k8s/overlays/prod/rbac.yaml

# 5) Appliquer la policy réseau prod (default deny)
kubectl apply -f k8s/networkpolicy.yaml

# 6) Déployer l'application via kustomize overlays
kubectl apply -k k8s/overlays/dev
kubectl apply -k k8s/overlays/staging
kubectl apply -k k8s/overlays/prod
```

## Personas (contexte kubeconfig)

```bash
kubectl config set-credentials platform-admin --token=fake-token-admin
kubectl config set-credentials dev-user --token=fake-token-dev
kubectl config set-credentials qa-user --token=fake-token-qa
kubectl config set-credentials prod-deployer --token=fake-token-prod

kubectl config set-context ctx-platform-admin --cluster=minikube --user=platform-admin --namespace=app-prod
kubectl config set-context ctx-dev-user --cluster=minikube --user=dev-user --namespace=app-dev
kubectl config set-context ctx-qa-user --cluster=minikube --user=qa-user --namespace=app-staging
kubectl config set-context ctx-prod-deployer --cluster=minikube --user=prod-deployer --namespace=app-prod

kubectl config set-context dev-user@app-cluster --cluster=minikube --user=dev-user --namespace=app-dev
kubectl config set-context qa-user@app-cluster --cluster=minikube --user=qa-user --namespace=app-staging
```

## Vérifications

```bash
# Namespaces
kubectl get ns app-dev app-staging app-prod

# Déploiements
kubectl get deploy -n app-dev
kubectl get deploy -n app-staging
kubectl get deploy -n app-prod

# RBAC
kubectl --context dev-user@app-cluster auth can-i create deployments -n app-dev
kubectl --context dev-user@app-cluster auth can-i get pods -n app-prod

# Quota (doit refuser en app-dev si dépassement CPU requests)
kubectl run quota-test --image=nginx -n app-dev --requests='cpu=2'
```

## Résumé RBAC actuel

| Sujet | Namespace | Permissions principales |
|---|---|---|
| dev-user | app-dev | lecture + debug + create/update/patch/delete sur pods, services, configmaps, deployments, ingresses |
| dev-user | app-staging | lecture + debug + create/update/patch/delete sur pods, services, configmaps, deployments, ingresses |
| prod-deployer (ServiceAccount) | app-prod | lecture pods/pods logs + update/patch deployments + create/update services |

## Notes sécurité

- Aucun accès explicite aux secrets dans les rôles dev/staging/prod.
- PSS baseline sur dev/staging, restricted sur prod.
- NetworkPolicy de type default deny dans app-prod.

## Mémo

- [ ] séparation env (namespaces, labels)
	- Définir `dev`, `staging`, `prod` dès le départ.
	- Standardiser les labels (`environment`, `owner`, `app`).

- [ ] RBAC (qui fait quoi)
	- Mapper les rôles par persona (dev, qa, ops, deployer).
	- Appliquer le moindre privilège et valider avec `kubectl auth can-i`.

- [ ] secrets (stratégie + limites)
	- Éviter `get/list/watch` sur `secrets` pour les profils dev.
	- Utiliser un secret manager et une rotation planifiée.

- [ ] quotas/limits
	- Ajouter `ResourceQuota` par namespace.
	- Ajouter `LimitRange` pour requests/limits par défaut.

- [ ] PSS
	- `baseline` en dev/staging, `restricted` en prod.
	- Vérifier `securityContext` sur tous les workloads.

- [ ] network policies
	- Poser une règle `default-deny` au minimum en prod.
	- Ouvrir explicitement uniquement les flux nécessaires.

- [ ] stratégie de release + rollback
	- Promouvoir dev -> staging -> prod.
	- Prévoir `rollout status` + `rollout undo` documentés.

- [ ] politique images (registry privé, scan, signature)
	- Registry privé obligatoire en prod.
	- Scan de vulnérabilités en CI, signature des images, tags immuables.
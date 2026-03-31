# TP Docker - Optimisation d'Image

Author: Alexandre CAROL

## Questions

### Partie 2.4
| Question | Réponse |
|----------|---------|
| Poids de `tp-docker:classic` ? | **1.62 GB** |
| Qu'est-ce qui gonfle l'image ? | **node_modules (eslint inclus), npm cache, image node:20** |

### Partie 3.1
| Question | Réponse |
|----------|---------|
| Pourquoi `node_modules` dans `.dockerignore` ? | **Éviter de copier les binaires compilés** |

### Partie 3.5
| Question | Réponse |
|----------|---------|
| Écart classic vs multistage ? | **~1.3 GB (80% de réduction)** |
| Qu'est-ce qui a été supprimé ? | **devDependencies (eslint), npm cache, binaires inutiles** |
| Réduit la surface d'attaque ? | **Oui, moins de dépendances = moins de vulnérabilités** |

### Partie 4.1
| Question | Réponse |
|----------|---------|
| Plus gros layers ? | **RUN npm ci** |
| Cache Docker avec package*.json ? | **Si le code change mais pas les dépendances, ce layer est réutilisé sans réinstaller** |

## 📊 Compte-rendu : Multi-stage vs Classic

**Poids:** Classic = 1.62 GB → Multi-stage = ~0.32 GB (**80% de réduction**)

**Pourquoi le multi-stage est plus léger :**
1. **Suppression des devDependencies** – Le stage final copie seulement `node_modules` de production (sans eslint, testing tools, etc.)
2. **Pas de npm cache ni de binaires compilés** – Seuls les sources Node compilées et le code applicatif sont copiés, éliminant les couches inutilisables

**Amélioration en production :** Ajouter une validation pour interdire les images > 500 MB en CI/CD (scanner de vulnérabilités + contrôle du poids)

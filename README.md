# TP CI/CD - Pipeline d'Intégration et Déploiement Continu

Ce dépôt contient le projet pratique pour le TP CI/CD. L'objectif est de mettre en place une chaîne complète de CI/CD automatisée (GitHub Actions) pour compiler, tester, scanner, et déployer une application Go sur un cluster Kubernetes local (Minikube) en utilisant ArgoCD (GitOps).

## Architecture du Projet

*   `/app` : Code source de l'application web en Go et ses tests.
*   `.github/workflows` : Configurations des pipelines d'intégration continue (GitHub Actions).

## Fonctionnalités Réalisées

### Étape 1 : Architecture Applicative & Packaging (En cours)

#### 1. Application Web en Go
Une micro-application en Go a été créée dans le dossier `/app`. Elle expose les routes suivantes :
- `GET /` : Page d'accueil avec un message de bienvenue.
- `GET /health` : Endpoint de vérification de l'état de santé de l'application (`status: OK`).

Pour lancer l'application localement (nécessite Go installé) :
```bash
cd app
go run main.go
```

Pour exécuter les tests :
```bash
cd app
go test -v
```

#### 2. Conteneurisation (Docker)
Un [Dockerfile](file:///c:/Users/nicoc/code/tp-ci-cd/app/Dockerfile) multi-stage optimisé et sécurisé a été mis en place dans le dossier `/app` :
- **Multi-stage Build** : Le binaire est compilé dans une image `golang:1.21-alpine` puis transféré dans une image finale `alpine:3.19.1` très légère (taille minimale de l'image).
- **Sécurité (Non-root)** : Un utilisateur et un groupe système non-privilégiés (`appuser:appgroup`) sont créés et utilisés via l'instruction `USER` pour éviter toute exécution en root.
- **Optimisation** : Désactivation de CGO (`CGO_ENABLED=0`) et suppression des tables de symboles/informations de debug (`-ldflags="-w -s"`) pour réduire encore la taille du binaire.

#### 3. Configuration de Déploiement (Helm Chart)
Un Helm Chart a été configuré dans le dossier `/chart` pour orchestrer le déploiement sur Kubernetes :
- `Chart.yaml` : Métadonnées du chart.
- `values.yaml` : Fichier de configuration contenant les variables comme l'image du conteneur (`repository`, `tag`), le nombre de réplicas, les ressources (CPU/RAM), et les ports d'écoute.
- `templates/` : Définitions Kubernetes dynamiques pour le `Deployment` (avec configuration de liveness/readiness probes sur `/health`, contextes de sécurité non-root et restriction de privilèges) et le `Service` (exposant le port).
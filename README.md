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
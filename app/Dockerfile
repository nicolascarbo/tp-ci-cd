# --- Étape 1 : Build de l'application ---
FROM golang:1.21-alpine AS builder

# Répertoire de travail
WORKDIR /build

# Copie des fichiers de dépendances et téléchargement
COPY go.mod ./
RUN go mod download

# Copie du reste du code source
COPY . .

# Compilation statique de l'application (désactivation de CGO pour un binaire autonome)
# -ldflags="-w -s" permet de réduire la taille du binaire en supprimant les infos de debug
RUN CGO_ENABLED=0 GOOS=linux go build -ldflags="-w -s" -o app .

# --- Étape 2 : Image d'exécution finale sécurisée ---
FROM alpine:3.19.1

# Installation de certificats de sécurité basiques si nécessaire
RUN apk --no-cache add ca-certificates

# Création d'un groupe et d'un utilisateur non-privilégié (non-root)
RUN addgroup -S appgroup && adduser -S appuser -G appgroup

# Définition du répertoire de travail pour l'utilisateur non-root
WORKDIR /home/appuser

# Copie du binaire compilé depuis l'étape builder
COPY --from=builder /build/app .

# Changement de propriétaire pour s'assurer que l'utilisateur non-root peut exécuter le binaire
RUN chown appuser:appgroup app

# Utilisation de l'utilisateur non-root
USER appuser

# Exposition du port d'écoute de l'application
EXPOSE 8080

# Commande de démarrage
ENTRYPOINT ["./app"]

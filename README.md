Guide complet Docker et Docker Compose
1. Commandes de base Docker
Connexion au registre Docker
docker login -u <username>


Permet de se connecter à Docker Hub ou un registre privé.

Après la commande, il demande le mot de passe.

Construire une image
docker build -t <nom_image>:<tag> .


Le . indique que le Dockerfile se trouve dans le dossier courant.

Exemple :

docker build -t monapp:1.0 .

Exécuter un conteneur
docker run -it <nom_image> /bin/bash


-it = interactif + terminal.

Pour exécuter en arrière-plan (détaché) :

docker run -d --name mon_conteneur <nom_image>

Push et Pull
docker push <nom_image>:<tag>
docker pull <nom_image>:<tag>


push = envoyer une image vers un registre.

pull = récupérer une image depuis un registre.

Lister et gérer les images
docker images        # Liste des images locales
docker rmi <id_image>  # Supprimer une image

2. Dockerfile et multi-stage build
Dockerfile classique

Il doit être dans le même dossier où vous lancez le docker build.

Exemple simple :

FROM ubuntu:22.04
RUN apt-get update && apt-get install -y python3
COPY . /app
WORKDIR /app
CMD ["python3", "app.py"]

Multi-stage build

Permet de réduire la taille des images.

Exemple :

# Stage 1: builder
FROM golang:1.20 AS builder
WORKDIR /app
COPY . .
RUN go build -o myapp

# Stage 2: final
FROM alpine:latest
WORKDIR /app
COPY --from=builder /app/myapp .
CMD ["./myapp"]


Avantage : vous ne gardez que ce qui est nécessaire, pas tout l’environnement de build.

3. Avantages d’utiliser des images « minimalistes »

Scratch : image vide, utile pour Go, C, etc.

Alpine : image très légère (≈5 MB).

Avantages :

Sécurité renforcée

Temps de téléchargement réduit

Moins d’espace disque

4. Gestion des volumes
Créer et utiliser un volume
docker volume create monvolume
docker volume ls
docker volume inspect monvolume

Monter un volume
docker run -d --mount source=monvolume,target=/app nginx:latest


--mount permet de lier un volume Docker à un chemin dans le conteneur.

Supprimer un volume

Il faut arrêter le conteneur qui utilise ce volume :

docker stop mon_conteneur
docker rm mon_conteneur
docker volume rm monvolume

Différence -v vs --mount

-v monvolume:/app → syntaxe plus ancienne.

--mount source=monvolume,target=/app → syntaxe plus explicite et flexible.

5. Gestion des réseaux
Liste des réseaux
docker network ls

Créer un réseau isolé
docker network create monreseau

Supprimer un réseau
docker network rm monreseau

Lancer un conteneur sur un réseau spécifique
docker run -d --name monconteneur --network monreseau nginx:latest

Exécuter une commande dans un conteneur
docker exec -it monconteneur /bin/bash


Le réseau par défaut est bridge, généralement recommandé pour des conteneurs simples.

6. Différence Docker vs Docker Compose
Docker CLI  Docker Compose
Lance un conteneur unique   Lance plusieurs conteneurs définis dans un fichier YAML
Commandes comme docker run  Commandes docker-compose up/down
Configuration inline    Configuration centralisée dans docker-compose.yml
Commandes Docker Compose principales
docker compose up          # Démarre les services
docker compose down        # Arrête et supprime les conteneurs
docker compose build       # Rebuild les images
docker compose ps          # Liste les conteneurs du projet
docker compose logs        # Voir les logs
docker compose stop/start  # Stop/start les services
docker compose exec <service> <commande>  # Exécuter commande dans un service
docker compose down -v     # Supprime volumes associés

7. Docker Buildx pour multi-arch
Installer buildx
sudo apt install docker-buildx

Lister les builders
docker builder ls

Créer un builder multi-architecture
docker buildx create --name multiarch --platform linux/amd64,linux/arm64 --driver docker-container --bootstrap --use

Construire pour plusieurs architectures
docker buildx build --platform linux/amd64,linux/arm64 -t monapp:latest .

8. Commandes supplémentaires utiles
docker ps                  # Liste conteneurs en cours
docker ps -a               # Tous les conteneurs
docker inspect <id>        # Info détaillée conteneur ou image
docker rm <id>             # Supprimer conteneur
docker rmi <id>            # Supprimer image
docker logs <id>           # Logs conteneur
docker stop <id>           # Arrêter conteneur
docker restart <id>        # Redémarrer conteneur
docker system prune -a     # Nettoyer images, conteneurs, volumes inutilisés

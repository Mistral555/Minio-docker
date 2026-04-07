# MinIO Docker

Déploiement de [MinIO](https://min.io/) avec Docker Compose, incluant un reverse proxy Nginx et une persistance des données via volumes.

MinIO est un serveur de stockage objet compatible avec l'API Amazon S3, idéal pour du self-hosting.

---

## Architecture

```
Internet / Navigateur
        │
        ▼
   [Nginx :80]         ← reverse proxy (point d'entrée unique)
   /          \
  /            \
[MinIO API]  [MinIO Console]
  :9000          :9001
        │
        ▼
  [Volume Docker]      ← données persistantes
```

| Service | Rôle | Port interne |
|---------|------|-------------|
| `minio` | Serveur de stockage objet | 9000 (API), 9001 (Console) |
| `nginx` | Reverse proxy | 80 → Console, 9000 → API |

---

## Structure du projet

```
minio-docker/
├── docker-compose.yml
├── .env                  # variables d'environnement (non commité)
├── .env.example          # modèle de configuration
├── .gitignore
├── nginx/
│   └── nginx.conf        # configuration du reverse proxy
└── README.md
```

---

## Lancement

### Prérequis

- [Docker](https://docs.docker.com/get-docker/) >= 20.x
- [Docker Compose](https://docs.docker.com/compose/) >= 2.x

### Installation

**1. Cloner le dépôt**
```bash
git clone <url-du-repo>
cd minio-docker
```

**2. Configurer les variables d'environnement**
```bash
cp .env.example .env
```
Puis éditer `.env` avec tes propres valeurs :
```env
MINIO_ROOT_USER=admin
MINIO_ROOT_PASSWORD=password123
```

**3. Démarrer les services**
```bash
docker compose up -d
```

**4. Vérifier que tout tourne**
```bash
docker compose ps
```

---

## Accès

| Interface | URL |
|-----------|-----|
| Console Web MinIO | http://localhost |
| API S3 | http://localhost:9000 |

Identifiants : ceux définis dans ton `.env`.

---

## Commandes utiles

```bash
# Démarrer les services
docker compose up -d

# Voir les logs en temps réel
docker compose logs -f

# Voir les logs d'un service spécifique
docker compose logs -f minio

# Vérifier l'état des healthchecks
docker ps

# Arrêter les services
docker compose down

# Arrêter et supprimer les volumes (⚠️ supprime les données)
docker compose down -v
```

---

## Persistance des données

Les données MinIO sont stockées dans un volume Docker nommé `minio_data`. Ce volume persiste même si les conteneurs sont supprimés.

```bash
# Lister les volumes Docker
docker volume ls

# Inspecter le volume
docker volume inspect minio_data
```

---

## Healthcheck

Le service `minio` expose un endpoint de santé natif :

```
GET http://localhost:9000/minio/health/live
```

Docker vérifie cet endpoint toutes les 10 secondes. Le service `nginx` ne démarre que lorsque MinIO est déclaré **healthy**, évitant ainsi les erreurs au démarrage.

---

## Sécurité

- Les credentials sont stockés dans `.env`, jamais dans le code
- Le fichier `.env` est exclu du dépôt Git via `.gitignore`
- Les ports de MinIO sont exposés uniquement en interne (via `expose`), Nginx est le seul point d'entrée public
- En production, il est recommandé d'utiliser HTTPS avec des certificats TLS

---

## Ressources

- [Documentation officielle MinIO](https://min.io/docs/)
- [Image Docker MinIO](https://quay.io/repository/minio/minio)
- [Documentation Nginx](https://nginx.org/en/docs/)
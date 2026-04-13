# 🎬 Media Server
 
Un serveur multimédia personnel pour télécharger et regarder des films en streaming, depuis ton navigateur, sans abonnement.
 
> Projet réalisé dans le cadre d'un hackathon DevOps — découverte de Docker et de l'auto-hébergement.
 
---
 
## C'est quoi ce projet ?
 
L'idée : reproduire ton propre Netflix à la maison, en utilisant trois outils open source qui fonctionnent ensemble :
 
| Outil | Rôle | Analogie |
|-------|------|----------|
|  **Jackett** | Cherche des films disponibles en torrent | Google, mais pour les torrents |
|  **Deluge** | Télécharge les films via torrent | Un client torrent avec interface web |
|  **Jellyfin** | Affiche et lit les films en streaming | Ton Netflix perso |
 
### Comment ils s'enchaînent ?
 
```
1. Tu cherches un film sur Jackett
        ↓
2. Jackett te donne un fichier .torrent
        ↓
3. Tu le donnes à Deluge, qui télécharge le film
        ↓
4. Le film apparaît automatiquement sur Jellyfin
        ↓
5. Tu le regardes depuis ton navigateur 
```
 
---
 
## Ce dont tu as besoin
 
- Un ordinateur sous Linux (ou une VM)
- **Docker** — pour faire tourner les applications dans des conteneurs
- **Docker Compose** — pour les lancer tous ensemble avec une seule commande
 
### Installer Docker
 
```bash
sudo apt update
sudo apt install docker.io -y
```
 
### Installer Docker Compose
 
```bash
sudo curl -L "https://github.com/docker/compose/releases/download/v2.24.0/docker-compose-$(uname -s)-$(uname -m)" -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```
 
### Vérifier que tout est installé
 
```bash
docker --version
docker-compose --version
```
 
---
 
## Lancer le projet
 
```bash
# 1. Récupérer le projet
git clone https://github.com/<ton-pseudo>/media-server.git
cd media-server
 
# 2. Lancer tous les services
sudo docker-compose up -d
 
# 3. Vérifier que tout tourne
sudo docker-compose ps
```
 
Tu devrais voir les 3 services avec le statut `Up`. 
 
---
 
## Accéder aux interfaces
 
Ouvre ton navigateur et va sur ces adresses (remplace `localhost` par l'IP de ta machine si tu es sur une VM) :
 
| Interface | Adresse | Mot de passe par défaut |
|-----------|---------|--------------------------|
|  Jellyfin | http://localhost:8096 | À créer au premier lancement |
|  Deluge | http://localhost:8112 | `deluge` |
|  Jackett | http://localhost:9117 | Aucun |
 
> **Sur une VM ?** Trouve ton IP avec `ip addr show` et cherche une ligne `inet 192.168.x.x`.
 
---
 
## Premiers pas
 
### 1. Configurer Jellyfin
Au premier lancement, un assistant te guide :
- Crée un compte administrateur (note bien ton mot de passe !)
- Ajoute une médiathèque : type **Films**, dossier `/media`
 
### 2. Ajouter un indexeur sur Jackett
- Clique sur **Add Indexer**
- Recherche **Internet Archive** (films du domaine public, gratuits et légaux)
- Clique sur **+** pour l'ajouter
 
### 3. Télécharger ton premier film
- Recherche un film dans Jackett
- Récupère le fichier `.torrent`
- Ajoute-le dans Deluge (bouton **+** → **Fichier**)
- Attends la fin du téléchargement
- Le film apparaît tout seul dans Jellyfin 
 
---
 
## Organisation des fichiers
 
Tout est rangé dans le dossier `media-server/` :
 
```
media-server/
├── docker-compose.yml     ← le fichier qui orchestre tout
│
├── jellyfin/
│   ├── config/            ← préférences de Jellyfin
│   ├── cache/             ← cache (miniatures, etc.)
│   └── media/             ← tes films sont ici
│
├── deluge/
│   └── config/            ← préférences de Deluge
│
└── jackett/
    ├── data/              ← préférences de Jackett
    └── blackhole/         ← dossier surveillé par Deluge
```
 
> Les films téléchargés par Deluge atterrissent dans `jellyfin/media/` — c'est pourquoi Jellyfin les voit automatiquement.
 
---
 
## Commandes utiles
 
```bash
# Démarrer les services
sudo docker-compose up -d
 
# Arrêter les services (sans supprimer les données)
sudo docker-compose down
 
# Voir les logs d'un service
sudo docker-compose logs jellyfin
 
# Redémarrer un seul service
sudo docker-compose restart deluge
```
 
---
 
## Mes données sont-elles perdues si je supprime les conteneurs ?
 
**Non.** Tous les fichiers (films, configuration, préférences) sont stockés sur ta machine dans le dossier `media-server/`. Les conteneurs Docker ne sont que des "boîtes" qui font tourner les applications — supprimer la boîte ne supprime pas ton contenu.
 
C'est ce qu'on appelle des **volumes persistants** en Docker. 
 
---
 
## Ce que j'ai appris
 
Ce projet m'a permis de découvrir :
 
-  **Docker & Docker Compose** — lancer plusieurs applications isolées
-  **Les volumes** — faire persister des données entre redémarrages
-  **La communication entre conteneurs** — partager un dossier entre deux services
-  **Le self-hosting** — héberger ses propres services plutôt que dépendre d'un tiers

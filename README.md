# Media Server

Un serveur pour télécharger des films en torrent et les regarder depuis le navigateur.
Pas d'abonnement, tout tourne chez toi.

> Projet fait pendant un hackathon DevOps, pour apprendre Docker et voir comment
> héberger ses propres applis.

---

## C'est quoi ?

J'avais envie de comprendre comment fonctionne un truc comme Netflix, mais en version
"fait maison". L'idée c'est de brancher trois outils ensemble :

| Outil | Rôle |
|-------|------|
| **Jackett** | Pour chercher des films disponibles en torrent |
| **Deluge** | Pour télécharger ces films |
| **Jellyfin** | Pour les regarder depuis le navigateur |

Une fois lancés, ça marche comme ça :
Tu cherches un film sur Jackett
→ Jackett te donne un lien torrent
→ Tu le mets dans Deluge, il télécharge
→ Le film apparaît tout seul dans Jellyfin
→ Tu regardes

---

## Ce qu'il faut avoir

- Un ordi sous Linux (ou une VM)
- Docker installé
- Docker Compose installé

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

### Vérifier que c'est bon

```bash
docker --version
docker-compose --version
```

---

## Lancer le projet

```bash
git clone https://github.com/<ton-pseudo>/media-server.git
cd media-server
sudo docker-compose up -d
sudo docker-compose ps
```

Si les 3 services affichent `Up`, c'est bon.

---

## Accéder aux interfaces

Une fois lancé, chaque outil a sa propre page web :

| Interface | Adresse | Mot de passe par défaut |
|-----------|---------|--------------------------|
| Jellyfin | http://localhost:8096 | À créer au premier lancement |
| Deluge | http://localhost:8112 | `deluge` |
| Jackett | http://localhost:9117 | Aucun |

> Si tu es sur une VM, remplace `localhost` par l'IP de la machine.
> Tu la trouves avec `ip addr show` (cherche une ligne `inet 192.168.x.x`).

---

## Premiers pas

### 1. Jellyfin

Au premier lancement il y a un assistant de configuration :
- Crée un compte admin (note le mot de passe quelque part)
- Ajoute une médiathèque : type **Films**, dossier `/media`

### 2. Jackett

- Clique sur **Add Indexer**
- Cherche **Internet Archive** (films du domaine public, gratuits)
- Clique sur **+**

### 3. Télécharger un film

- Cherche dans Jackett
- Récupère le `.torrent`
- Mets-le dans Deluge (bouton **+** → **Fichier**)
- Attends la fin du téléchargement
- Le film apparaît dans Jellyfin tout seul

---

## Les fichiers

```
media-server/
  docker-compose.yml     <- le fichier principal
  jellyfin/
    config/
    cache/
    media/               <- les films finissent ici
  deluge/
    config/
  jackett/
    data/
    blackhole/           <- Deluge surveille ce dossier
```

---

## Commandes utiles

```bash
# Démarrer
sudo docker-compose up -d

# Arrêter (sans perdre les données)
sudo docker-compose down

# Voir les logs d'un service
sudo docker-compose logs jellyfin

# Redémarrer un service
sudo docker-compose restart deluge
```

---

## Est-ce que je perds mes films si je supprime les conteneurs ?

Non. Les films et les configs sont sur ta machine, dans `media-server/`.
Docker ne fait que "faire tourner" les applis, comme une boîte. Supprimer la boîte
ne touche pas à ce qu'il y a dedans.

C'est ce qu'on appelle les volumes en Docker.

---

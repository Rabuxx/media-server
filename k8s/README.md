# 🎬 Media Server — Kubernetes

Un serveur multimédia personnel pour télécharger et regarder des films en streaming, déployé sur Kubernetes avec k3s.

> Projet réalisé dans le cadre d'un hackathon DevOps — découverte de Kubernetes et de l'auto-hébergement.

---

## C'est quoi ce projet ?

L'idée : reproduire ton propre Netflix à la maison, en utilisant trois outils open source orchestrés avec Kubernetes :

| Outil | Rôle | Analogie |
|-------|------|----------|
| **Jackett** | Cherche des films disponibles en torrent | Google, mais pour les torrents |
| **Deluge** | Télécharge les films via torrent | Un client torrent avec interface web |
| **Jellyfin** | Affiche et lit les films en streaming | Ton Netflix perso |

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
- **k3s** — la version légère de Kubernetes, parfaite pour une machine locale

### Installer k3s

```bash
curl -sfL https://get.k3s.io | sh -

# Vérifier que le nœud est prêt
sudo kubectl get nodes
```

Tu devrais voir ton nœud avec le statut `Ready`. 

---

## Structure du projet

```
media-server/
├── jellyfin/
│   ├── config/        ← préférences de Jellyfin
│   ├── cache/         ← cache (miniatures, etc.)
│   └── media/         ← tes films sont ici (partagé avec Deluge)
│
├── deluge/
│   └── config/        ← préférences de Deluge
│
├── jackett/
│   ├── data/          ← préférences de Jackett
│   └── blackhole/     ← fichiers .torrent (partagé avec Deluge)
│
└── k8s/
    ├── jellyfin.yaml  ← PV + PVC + Deployment + Service
    ├── deluge.yaml    ← PV + PVC + Deployment + Service
    └── jackett.yaml   ← PV + PVC + Deployment + Service
```

> `jellyfin/media/` est partagé entre Jellyfin et Deluge — quand Deluge télécharge un film, Jellyfin le voit automatiquement. De même, `jackett/blackhole/` est partagé entre Jackett et Deluge pour le transfert automatique des fichiers `.torrent`.

---

## C'est quoi les objets Kubernetes ?

Chaque fichier `.yaml` contient 4 types d'objets :

| Objet | Rôle |
|-------|------|
| **PersistentVolume (PV)** | Déclare le stockage disponible sur la machine hôte |
| **PersistentVolumeClaim (PVC)** | Réclame une partie de ce stockage pour un service |
| **Deployment** | Gère le conteneur, son image et ses volumes |
| **Service** | Expose le conteneur sur un port accessible depuis l'extérieur |

```
Ta machine (hostPath)
      ↓
PersistentVolume      ← "voilà le disque disponible"
      ↓
PersistentVolumeClaim ← "je réclame X Go de ce disque"
      ↓
Deployment            ← "je monte ce PVC dans mon conteneur"
      ↓
Service (NodePort)    ← "j'expose ce conteneur sur le port 3xxxx"
```

---

## Déployer

> Applique Jackett **avant** Deluge — Deluge utilise le PVC `jackett-blackhole` créé par le manifest Jackett.

```bash
cd k8s/

sudo kubectl apply -f jellyfin.yaml
sudo kubectl apply -f jackett.yaml
sudo kubectl apply -f deluge.yaml

# Vérifier que tout tourne
sudo kubectl get pods
```

Tu devrais voir les 3 pods avec le statut `Running`. 

---

## Accéder aux interfaces

> Remplace `<IP>` par l'IP de ta machine (`ip addr show`).

| Interface | Adresse | Identifiants par défaut |
|-----------|---------|--------------------------|
| Jellyfin | http://\<IP\>:30096 | À créer au premier lancement |
| Deluge | http://\<IP\>:30112 | `deluge` / `deluge` |
| Jackett | http://\<IP\>:30117 | Aucun |

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
- Recherche un film dans Jackett et récupère le fichier `.torrent`
- Ajoute-le dans Deluge (bouton **+** → **Fichier**)
- Attends la fin du téléchargement
- Le film apparaît tout seul dans Jellyfin 

---

## Commandes utiles

```bash
# État des pods
sudo kubectl get pods

# État des volumes
sudo kubectl get pvc

# Logs d'un pod
sudo kubectl logs <nom-du-pod>

# Détails et erreurs d'un pod
sudo kubectl describe pod <nom-du-pod>

# Supprimer un service
sudo kubectl delete -f jellyfin.yaml
```

> Les données (films, config) sont stockées dans les dossiers locaux — supprimer les pods ne les efface pas.

---

## Ce que j'ai appris

- **Kubernetes / k3s** — orchestrer des conteneurs avec des manifests YAML
- **PersistentVolumes & PVC** — déclarer et réclamer du stockage persistant
- **Volumes partagés** — faire communiquer deux services via un dossier commun
- **NodePort** — exposer un service Kubernetes sur un port accessible depuis l'extérieur
- **Self-hosting** — héberger ses propres services sans dépendre d'un abonnement

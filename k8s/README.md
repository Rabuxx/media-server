# Media Server — Kubernetes

Un serveur pour télécharger des films en torrent et les regarder depuis le navigateur.
Déployé sur Kubernetes avec k3s.

> Projet fait pendant un hackathon DevOps, pour apprendre Kubernetes et voir comment
> orchestrer plusieurs services sur une machine locale.

---

## C'est quoi ?

Trois outils qui fonctionnent ensemble :

| Outil | Rôle |
|-------|------|
| **Jackett** | Chercher des films disponibles en torrent |
| **Deluge** | Télécharger ces films |
| **Jellyfin** | Les regarder depuis le navigateur |

---

## Ce qu'il faut avoir

- Un ordi sous Linux (ou une VM)
- **k3s** — une version allégée de Kubernetes, plus simple à installer sur une seule machine

### Installer k3s

```bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

Si tu vois ton nœud avec le statut `Ready`, c'est bon.

---

## Comment c'est organisé

```
media-server/
  jellyfin/
    config/        <- préférences de Jellyfin
    cache/
    media/         <- les films sont ici (Deluge dépose aussi ici)
  deluge/
    config/
  jackett/
    data/
    blackhole/     <- Deluge surveille ce dossier pour les .torrent
  k8s/
    jellyfin.yaml  <- config Kubernetes pour Jellyfin
    deluge.yaml    <- config Kubernetes pour Deluge
    jackett.yaml   <- config Kubernetes pour Jackett
```

`jellyfin/media/` est partagé entre Jellyfin et Deluge — quand Deluge finit un téléchargement,
Jellyfin voit le film automatiquement.

---

## C'est quoi les objets Kubernetes ?

Chaque fichier `.yaml` contient 4 types d'objets :

| Objet | Rôle |
|-------|------|
| **PersistentVolume (PV)** | Déclare un dossier de la machine comme espace de stockage |
| **PersistentVolumeClaim (PVC)** | Demande à utiliser cet espace pour un service |
| **Deployment** | Lance le conteneur avec ses volumes |
| **Service** | Rend le conteneur accessible depuis le navigateur |

En gros ça suit cette logique :

```
Dossier sur ta machine
    -> PersistentVolume   : "ce dossier est disponible"
    -> PersistentVolumeClaim : "je veux utiliser ce dossier"
    -> Deployment         : "je branche ce dossier dans mon conteneur"
    -> Service            : "j'expose ce conteneur sur un port"
```

---

## Déployer

Applique Jackett avant Deluge — Deluge a besoin d'un volume créé par Jackett.

```bash
cd k8s/
sudo kubectl apply -f jellyfin.yaml
sudo kubectl apply -f jackett.yaml
sudo kubectl apply -f deluge.yaml
sudo kubectl get pods
```

Si les 3 pods affichent `Running`, c'est bon.

---

## Accéder aux interfaces

Remplace `<IP>` par l'IP de ta machine (commande : `ip addr show`).

| Interface | Adresse | Identifiants par défaut |
|-----------|---------|--------------------------|
| Jellyfin | http://\<IP\>:30096 | À créer au premier lancement |
| Deluge | http://\<IP\>:30112 | `deluge` / `deluge` |
| Jackett | http://\<IP\>:30117 | Aucun |

---

## Premiers pas

### 1. Jellyfin

Au premier lancement il y a un assistant :
- Crée un compte admin (note le mot de passe)
- Ajoute une médiathèque : type **Films**, dossier `/media`

### 2. Jackett

- Clique sur **Add Indexer**
- Cherche **Internet Archive** (films gratuits et légaux)
- Clique sur **+**

### 3. Télécharger un film

- Cherche dans Jackett, récupère le `.torrent`
- Mets-le dans Deluge (bouton **+** → **Fichier**)
- Attends la fin du téléchargement
- Le film apparaît dans Jellyfin tout seul

---

## Commandes utiles

```bash
# État des pods
sudo kubectl get pods

# État des volumes
sudo kubectl get pvc

# Logs d'un pod
sudo kubectl logs <nom-du-pod>

# Déboguer un pod qui ne démarre pas
sudo kubectl describe pod <nom-du-pod>

# Supprimer un service
sudo kubectl delete -f jellyfin.yaml
```

Les films et les configs sont stockés en local — supprimer les pods ne les efface pas.


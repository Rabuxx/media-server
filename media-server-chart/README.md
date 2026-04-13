# Media Server — Helm Chart

Un serveur pour télécharger des films en torrent et les regarder depuis le navigateur.
Tout est déployé sur Kubernetes via Helm, en une seule commande.

> Projet fait pendant un hackathon DevOps, pour apprendre Helm et comprendre comment
> orchestrer plusieurs services avec Kubernetes.

---

## C'est quoi ?

Même idée que la version Docker Compose, mais cette fois déployée sur Kubernetes.
Trois outils qui fonctionnent ensemble :

| Outil | Rôle |
|-------|------|
| **Jackett** | Chercher des films disponibles en torrent |
| **Deluge** | Télécharger ces films |
| **Jellyfin** | Les regarder depuis le navigateur |

---

## C'est quoi Helm ?

Kubernetes demande un fichier de config pour chaque chose : chaque service, chaque volume, chaque règle réseau... Ça fait vite beaucoup de fichiers à gérer.

Helm règle ce problème. Tu mets toutes tes valeurs dans un seul fichier (`values.yaml`), et Helm génère tout le reste automatiquement à partir de templates.

### Comment c'est organisé

```
media-server-chart/
  Chart.yaml          <- nom et version du chart
  values.yaml         <- toute la config (ports, images, volumes...)
  templates/
    deployment.yaml   <- template pour déployer les 3 services
    service.yaml      <- template pour exposer les 3 services
    pv.yaml           <- template pour les volumes
    pvc.yaml          <- template pour les demandes de volumes
```

Au lieu d'écrire trois fois la même chose pour chaque service, les templates utilisent une boucle qui tourne sur ce qui est défini dans `values.yaml` :

```yaml
{{- range .Values.services }}
  name: {{ .name }}
  image: {{ .image }}
{{- end }}
```

---

## Ce qu'il faut avoir

- Un ordi sous Linux (ou une VM)
- **k3s** — une version allégée de Kubernetes
- **Helm** — pour déployer le chart

### Installer k3s

```bash
curl -sfL https://get.k3s.io | sh -
sudo kubectl get nodes
```

### Installer Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash
helm version
```

### Donner accès à la config k3s

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

---

## Déployer

```bash
git clone https://github.com/Rabuxx/media-server.git
cd media-server
helm install media-server media-server-chart
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

## Changer la config

Tout est dans `values.yaml`. Tu peux modifier les ports, les images Docker ou les tailles
de volumes sans toucher aux templates.

```yaml
services:
  - name: jellyfin
    image: jellyfin/jellyfin
    port: 8096
    nodePort: 30096
```

Après une modif, pour mettre à jour :

```bash
helm upgrade media-server media-server-chart
```

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
# Voir ce qui est déployé
helm list

# Mettre à jour après une modif
helm upgrade media-server media-server-chart

# Désinstaller (sans perdre les données)
helm uninstall media-server

# État des pods
sudo kubectl get pods

# Logs d'un pod
sudo kubectl logs <nom-du-pod>

# Déboguer un pod qui ne démarre pas
sudo kubectl describe pod <nom-du-pod>
```

Les films et les configs sont stockés en local — désinstaller le chart ne les efface pas.

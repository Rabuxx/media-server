# 🎬 Media Server — Helm Chart

Un serveur multimédia personnel déployé sur Kubernetes via un chart Helm, pour télécharger et regarder des films en streaming depuis ton navigateur.

> Projet réalisé dans le cadre d'un hackathon DevOps — découverte de Helm et de l'orchestration Kubernetes.

---

## C'est quoi ce projet ?

L'idée : reproduire ton propre Netflix à la maison avec trois outils open source, déployés en une seule commande grâce à Helm :

| Outil | Rôle | Analogie |
|-------|------|----------|
| **Jackett** | Cherche des films disponibles en torrent | Google, mais pour les torrents |
| **Deluge** | Télécharge les films via torrent | Un client torrent avec interface web |
| **Jellyfin** | Affiche et lit les films en streaming | Ton Netflix perso |

---

## C'est quoi Helm ?

Helm est le **gestionnaire de paquets de Kubernetes**.

Sans Helm, tu dois écrire et appliquer un fichier YAML par service, par volume, par service réseau... Ça devient vite des dizaines de fichiers à maintenir.

Avec Helm, tu définis **une seule fois** tes valeurs dans `values.yaml`, et des **templates** génèrent automatiquement tous les objets Kubernetes.

```
Analogie : si Kubernetes c'est un OS, Helm c'est apt ou npm.
```

### Structure du chart

```
media-server-chart/
├── Chart.yaml          ← carte d'identité du chart (nom, version)
├── values.yaml         ← toutes les valeurs configurables (images, ports, volumes...)
└── templates/
    ├── deployment.yaml ← template des Deployments (boucle sur les 3 services)
    ├── service.yaml    ← template des Services (boucle sur les 3 services)
    ├── pv.yaml         ← template des PersistentVolumes
    └── pvc.yaml        ← template des PersistentVolumeClaims
```

Au lieu de dupliquer la configuration pour chaque service, les templates utilisent une boucle `range` pour itérer sur les services et volumes définis dans `values.yaml` :

```yaml
{{- range .Values.services }}
  name: {{ .name }}
  image: {{ .image }}
  ...
{{- end }}
```

---

## Ce dont tu as besoin

- Un ordinateur sous Linux (ou une VM)
- **k3s** — la version légère de Kubernetes
- **Helm** — le gestionnaire de paquets Kubernetes

### Installer k3s

```bash
curl -sfL https://get.k3s.io | sh -

# Vérifier que le nœud est prêt
sudo kubectl get nodes
```

### Installer Helm

```bash
curl https://raw.githubusercontent.com/helm/helm/main/scripts/get-helm-3 | bash

# Vérifier l'installation
helm version
```

### Exposer la config k3s à Helm

```bash
export KUBECONFIG=/etc/rancher/k3s/k3s.yaml
```

---

## Déployer

```bash
# Cloner le projet
git clone https://github.com/Rabuxx/media-server.git
cd media-server

# Déployer avec Helm
helm install media-server media-server-chart

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

## Personnaliser la configuration

Toute la configuration est centralisée dans `values.yaml`. Tu peux modifier les ports, les images ou les tailles de volumes sans toucher aux templates.

```yaml
services:
  - name: jellyfin
    image: jellyfin/jellyfin
    port: 8096
    nodePort: 30096
    volumes:
      - name: jellyfin-config
        mountPath: /config
      ...

volumes:
  - name: jellyfin-config
    hostPath: /home/engineer/projects/J16/media-server/jellyfin/config
    mountPath: /config
    size: 1Gi
  ...
```

Après modification, mets à jour le déploiement :

```bash
helm upgrade media-server media-server-chart
```

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
# Voir les releases Helm déployées
helm list

# Mettre à jour après modification de values.yaml
helm upgrade media-server media-server-chart

# Désinstaller (sans supprimer les données)
helm uninstall media-server

# État des pods
sudo kubectl get pods

# Logs d'un pod
sudo kubectl logs <nom-du-pod>

# Déboguer un problème
sudo kubectl describe pod <nom-du-pod>
```

> Les données (films, config) sont stockées dans les dossiers locaux — désinstaller le chart ne les efface pas.

---

## Ce que j'ai appris

- **Helm** — packager et déployer une application Kubernetes en une commande
- **Templates & values.yaml** — séparer la configuration du code d'infrastructure
- **Boucles `range`** — générer dynamiquement des objets Kubernetes sans répétition
- **PersistentVolumes & PVC** — gérer le stockage persistant dans Kubernetes
- **Volumes partagés** — faire communiquer plusieurs services via un dossier commun

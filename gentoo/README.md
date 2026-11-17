# Gentoo — Installation et configuration du système

## Présentation

Préparation d’un système Gentoo dans une machine virtuelle
VMware Workstation, à partir d’un environnement live
et d’une archive stage3 avec OpenRC.

Ce projet porte sur le partitionnement, la préparation
de l’environnement d’installation et la configuration
du système avec Portage.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Architecture | AMD64 — x86_64 |
| Environnement d’installation | Gentoo minimal |
| Variante du stage3 | OpenRC |
| Processeurs virtuels | 2 |
| Mémoire configurée pour la VM | 4 Go |
| Disque virtuel | 60 Go |
| Réseau | NAT |
| Démarrage | UEFI |
| Table de partitions | GPT |

## Objectifs

- Préparer le stockage pour une installation Linux.
- Déployer le système de base depuis une archive stage3.
- Configurer Portage et les paramètres du système.
- Travailler dans un environnement chroot.
- Préparer le noyau, les services et le démarrage.

## 1. Préparation de l’environnement

Depuis l’environnement live, j’ai configuré le clavier
français et identifié l’interface réseau ainsi que son adresse IP.

J’ai également vérifié l’heure et préparé le disque virtuel
pour accueillir le système.

## 2. Organisation du stockage

J’ai utilisé une table de partitions GPT et séparé
les fonctions de démarrage, d’échange, de système
et de stockage des répertoires personnels.

| Partition | Fonction | Format ou usage | Point de montage prévu |
|---|---|---|---|
| `/dev/sda1` | Démarrage UEFI | FAT32 — partition système EFI | `/boot/efi` |
| `/dev/sda2` | Espace d’échange | Swap Linux | Aucun |
| `/dev/sda3` | Système principal | ext4 | `/` |
| `/dev/sda4` | Répertoires personnels | ext4 | `/home` |

J’ai monté la partition racine sous `/mnt/gentoo`,
puis les partitions EFI et `/home` dans les sous-répertoires
correspondants.

Le swap a été préparé et activé séparément.

## 3. Déploiement du système de base

J’ai téléchargé une archive stage3 AMD64 avec OpenRC
depuis le serveur de distribution Gentoo.

Le stage3 fournit une base de système à extraire dans
la future partition racine. Il ne constitue pas, à lui seul,
une installation prête à démarrer.

J’ai extrait cette archive sous `/mnt/gentoo`
en conservant les informations de propriétaires
et les attributs nécessaires.

## 4. Configuration de Portage

Portage est le système de gestion des paquets de Gentoo.
La commande `emerge` permet notamment de préparer
les installations et les mises à jour.

J’ai travaillé sur le fichier `/etc/portage/make.conf`
pour adapter les paramètres aux ressources de la VM
et aux besoins de l’installation.

Les réglages abordés concernaient notamment :

- le parallélisme de compilation ;
- les paramètres linguistiques ;
- les composants graphiques et les périphériques d’entrée ;
- les miroirs de téléchargement.

Le choix du parallélisme doit tenir compte de la mémoire
disponible, en plus du nombre de processeurs virtuels.

## 5. Travail dans le système installé

Avant d’entrer dans le chroot, j’ai copié la configuration
de résolution DNS et rendu accessibles les systèmes
de fichiers nécessaires, notamment `/proc`, `/sys` et `/dev`.

Le chroot permet de travailler dans l’arborescence
du futur système depuis l’environnement live.
Il ne correspond pas encore à un démarrage sur Gentoo.

Dans cet environnement, j’ai effectué les étapes suivantes :

- récupération d’un instantané du dépôt Gentoo avec `emerge-webrsync` ;
- consultation des profils disponibles ;
- lancement d’une mise à jour de l’ensemble `@world` ;
- configuration et génération des locales ;
- configuration du clavier et du fuseau horaire ;
- préparation du fichier `/etc/fstab`.

L’ensemble `@world` représente les paquets sélectionnés
pour le système, dont la mise à jour entraîne également
le traitement des dépendances nécessaires.

## 6. Préparation du noyau et du démarrage

J’ai abordé la préparation du démarrage avec les composants
suivants :

| Composant | Rôle |
|---|---|
| `gentoo-kernel-bin` | Fournir un noyau Gentoo précompilé |
| `installkernel` | Gérer les opérations associées à l’installation du noyau |
| dracut | Générer l’initramfs utilisé pendant le démarrage |
| GRUB | Charger le système au démarrage |

Le choix d’un noyau précompilé permet de travailler
sur l’installation sans commencer par une compilation
manuelle du noyau.

La configuration décrite dans le compte rendu ne suffit
toutefois pas à confirmer le fonctionnement complet
de la chaîne de démarrage UEFI.

## 7. Gestion des services avec OpenRC

Le stage3 choisi utilise OpenRC pour la gestion des services.

Cette installation m’a permis d’aborder une autre méthode
de gestion du démarrage des services que celle utilisée
avec systemd sur Debian et Arch Linux.

La préparation a porté sur le réseau et le service SSH.
Leur fonctionnement après redémarrage reste à confirmer.

## État du projet

Le compte rendu documente la préparation du stockage,
l’extraction du stage3 et les étapes de configuration
dans le chroot.

La validation finale reste à compléter pour :

- le démarrage autonome de la VM sans l’image d’installation ;
- l’obtention d’une configuration réseau fonctionnelle ;
- l’ouverture d’une session utilisateur ;
- l’accès SSH, si ce service est retenu.

## Compétences mobilisées

- Partitionnement GPT et préparation des systèmes de fichiers.
- Montage d’une arborescence Linux.
- Extraction d’un système de base depuis une archive.
- Configuration de Portage.
- Administration dans un environnement chroot.
- Préparation du noyau et découverte d’OpenRC.

## Références

- [Site officiel Gentoo](https://www.gentoo.org/)
- [Manuel officiel Gentoo AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Téléchargements Gentoo](https://www.gentoo.org/downloads/)

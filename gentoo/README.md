# Gentoo — Installation et configuration du système

## Présentation

J’installe Gentoo dans une machine virtuelle VMware Workstation,
à partir d’une image minimale et d’une archive stage3 avec OpenRC.

Je prépare les partitions, installe les fichiers de base
et configure le système en ligne de commande.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Architecture | AMD64 — x86_64 |
| Environnement d’installation | Image minimale Gentoo |
| Version du stage3 | OpenRC |
| Processeurs virtuels | 2 |
| Mémoire configurée pour la VM | 4 Go |
| Disque virtuel | 60 Go |
| Réseau | NAT |
| Démarrage | UEFI |
| Table de partitions | GPT |

## Objectifs

- Préparer le disque et les partitions.
- Installer les fichiers de base depuis une archive stage3.
- Configurer Portage et les paramètres du système.
- Utiliser un environnement chroot.
- Préparer le noyau, les services et le démarrage.

## 1. Préparation de l’environnement

Depuis l’environnement d’installation, je configure
le clavier en français.

J’identifie l’interface réseau et son adresse IPv4.
Je vérifie également la date, l’heure et les disques
disponibles avant de continuer.

## 2. Organisation du stockage

Je partitionne le disque en GPT avec quatre partitions.

| Partition | Fonction | Format ou usage | Point de montage prévu |
|---|---|---|---|
| `/dev/sda1` | Démarrage UEFI | FAT32 — partition système EFI | `/boot/efi` |
| `/dev/sda2` | Espace d’échange | Swap Linux | Aucun |
| `/dev/sda3` | Système principal | ext4 | `/` |
| `/dev/sda4` | Répertoires personnels | ext4 | `/home` |

Je prépare les systèmes de fichiers, puis je monte
la partition racine dans `/mnt/gentoo`.

Je monte ensuite les partitions EFI et `/home`
dans les dossiers correspondants. J’active aussi le swap,
qui sert d’espace d’échange sur disque.

## 3. Déploiement du système de base

Je télécharge une archive stage3 AMD64 avec OpenRC
depuis le serveur officiel de Gentoo.

Le stage3 contient les fichiers du système de base.
Il faut ensuite les configurer et préparer le démarrage :
extraire l’archive ne suffit pas pour terminer l’installation.

Je l’extrais dans `/mnt/gentoo` en conservant les propriétaires,
les permissions et les attributs des fichiers.

## 4. Configuration de Portage

Portage permet de gérer les paquets sur Gentoo.
J’utilise notamment la commande `emerge` pour les installations
et les mises à jour.

Je modifie le fichier `/etc/portage/make.conf`
pour adapter la configuration à ma machine virtuelle.

Les paramètres abordés concernent :

- le nombre de tâches de compilation simultanées ;
- la langue ;
- les composants graphiques et les périphériques d’entrée ;
- les miroirs de téléchargement.

Pour régler le nombre de tâches, je prends en compte
les processeurs et la mémoire disponibles.

## 5. Travail dans le système installé

Je copie la configuration DNS de l’environnement
d’installation vers le futur système.

Je prépare ensuite l’accès à `/proc`, `/sys` et `/dev`,
puis j’entre dans le système avec `chroot`.

Cela me permet de configurer Gentoo depuis l’environnement
d’installation, avant de démarrer dessus.

Dans le chroot :

- je récupère un instantané du dépôt avec `emerge-webrsync` ;
- je consulte les profils disponibles ;
- je lance la mise à jour de l’ensemble `@world` ;
- je configure et génère les locales ;
- je configure le clavier et le fuseau horaire ;
- je renseigne les montages dans `/etc/fstab`.

L’ensemble `@world` sert à suivre les paquets sélectionnés
pour le système. Sa mise à jour prend aussi en compte
leurs dépendances.

## 6. Préparation du noyau et du démarrage

Je prépare le noyau et les outils nécessaires au démarrage.

| Composant | Rôle |
|---|---|
| `gentoo-kernel-bin` | Fournit un noyau Gentoo précompilé |
| `installkernel` | Gère les opérations liées à l’installation du noyau |
| dracut | Prépare l’initramfs, utilisé au début du démarrage |
| GRUB | Permet de charger le système au démarrage |

Je choisis un noyau précompilé : je ne compile donc pas
le noyau manuellement.

La préparation est décrite dans mon compte rendu.
Le fonctionnement du démarrage UEFI reste à confirmer
par un test final.

## 7. Gestion des services avec OpenRC

La version du stage3 choisie utilise OpenRC
pour gérer les services.

Je découvre ainsi une autre façon de gérer leur démarrage
que celle utilisée avec systemd sur Debian et Arch Linux.

Mon compte rendu aborde la préparation du réseau
et l’activation de SSH avec `rc-update`.

Leur fonctionnement après redémarrage reste à vérifier.

## État du projet

J’ai préparé les partitions, extrait le stage3 et travaillé
sur la configuration du système dans le chroot.

Les éléments conservés ne confirment pas le démarrage final.
Pour terminer la validation, je dois :

- démarrer la VM sans l’image d’installation ;
- vérifier la configuration réseau ;
- ouvrir une session utilisateur ;
- tester la connexion SSH si le service est activé.

## Compétences mobilisées

- Partitionnement GPT et préparation des systèmes de fichiers.
- Montage des partitions.
- Extraction d’une archive stage3.
- Configuration de Portage.
- Travail dans un environnement chroot.
- Préparation du noyau et découverte d’OpenRC.

## Références

- [Site officiel Gentoo](https://www.gentoo.org/)
- [Manuel officiel Gentoo AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64)
- [Téléchargements Gentoo](https://www.gentoo.org/downloads/)

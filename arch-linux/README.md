# Arch Linux — Installation et configuration

## Présentation

Installation manuelle d’Arch Linux dans une machine virtuelle
VMware Workstation : préparation du stockage, installation
du système, configuration réseau et démarrage en UEFI.

Ce projet m’a permis de comprendre les principales étapes
d’une installation Linux et de résoudre un problème de capacité
du système de fichiers.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Système | Arch Linux x86_64 |
| Image d’installation | Octobre 2026 |
| Processeurs virtuels | 2 |
| Mémoire vive | 2 Go |
| Disque virtuel | Initialement 20 Go, puis agrandi à 60 Go |
| Réseau | NAT |
| Mode de démarrage | UEFI |
| Secure Boot | Désactivé pour cette installation |
| Table de partitions | GPT |

## Objectifs

- Installer un système Linux depuis un environnement live.
- Préparer les partitions et les systèmes de fichiers.
- Configurer le réseau, la langue et les comptes utilisateurs.
- Installer et configurer le chargeur de démarrage GRUB.
- Vérifier le démarrage du système et sa connectivité.

## 1. Préparation de l’installation

J’ai téléchargé l’image ISO depuis une source officielle
et comparé son empreinte SHA-256 avec la valeur publiée
par Arch Linux pour cette image.

Depuis l’environnement live, j’ai ensuite configuré le clavier
français, vérifié le mode UEFI et identifié les interfaces réseau.

J’ai également contrôlé la connectivité, réglé le fuseau horaire
sur `Europe/Paris` et activé la synchronisation de l’heure.

## 2. Partitionnement et systèmes de fichiers

J’ai organisé le disque avec une table de partitions GPT
et quatre partitions.

| Partition | Fonction | Format ou usage | Point de montage dans le système installé |
|---|---|---|---|
| `/dev/sda1` | Démarrage UEFI | FAT32 — partition système EFI | `/boot/efi` |
| `/dev/sda2` | Espace d’échange | Swap Linux | Aucun |
| `/dev/sda3` | Système principal | ext4 | `/` |
| `/dev/sda4` | Répertoires personnels | ext4 | `/home` |

Cette organisation sépare le système des répertoires personnels.
Elle correspond au choix retenu pour cette installation.

J’ai préparé les systèmes de fichiers, activé le swap,
puis monté la partition racine sous `/mnt`.

Les partitions EFI et `/home` ont été montées dans
les sous-répertoires correspondants avant l’installation
des paquets.

## 3. Installation du système de base

J’ai utilisé `pacstrap` pour installer le système dans
l’arborescence montée sous `/mnt`.

Les paquets installés comprenaient notamment :

- le système de base, le noyau Linux et les firmwares ;
- les outils d’administration `sudo` et `nano` ;
- NetworkManager pour la gestion du réseau ;
- GRUB et les outils nécessaires au démarrage UEFI.

J’ai ensuite généré le fichier `/etc/fstab` avec des identifiants
UUID pour décrire les systèmes de fichiers à monter
et leurs points de montage.

## 4. Configuration du système

Avec `arch-chroot`, j’ai travaillé dans l’environnement
du système installé pour préparer sa configuration
avant son premier démarrage.

| Élément | Configuration réalisée |
|---|---|
| Langue | Configuration des locales françaises |
| Clavier | Configuration du clavier pour la console |
| Heure | Fuseau horaire `Europe/Paris` |
| Identité | Définition du nom de la machine et adaptation de `/etc/hosts` |
| Réseau | Activation de NetworkManager au démarrage |
| Comptes | Définition des mots de passe et création d’un utilisateur |
| Administration | Configuration de l’accès à `sudo` pour le groupe `wheel` |

## 5. Installation du chargeur de démarrage

J’ai installé GRUB en mode UEFI, en utilisant
la partition EFI montée sous `/boot/efi`.

J’ai ensuite généré son fichier de configuration
dans `/boot/grub/grub.cfg`.

Après avoir quitté le chroot et démonté les systèmes
de fichiers, j’ai redémarré la machine sur le système installé.

## 6. Dépannage réalisé : capacité du système de fichiers

### Problème rencontré

L’installation des paquets a été interrompue par un manque
d’espace disponible.

### Diagnostic

La partition racine `/dev/sda3` avait été agrandie à environ
32 Go, mais le système de fichiers ext4 n’utilisait pas encore
cette nouvelle capacité.

### Intervention

J’ai redimensionné le système de fichiers avec `resize2fs`,
puis contrôlé l’espace disponible sur `/mnt` avec `df`.

### Résultat

L’installation des paquets avec `pacstrap` a ensuite
pu reprendre et aboutir.

Cette intervention m’a permis de distinguer trois niveaux
du stockage : le disque virtuel, la partition et le système
de fichiers.

## 7. Vérifications et résultats

| Contrôle | Résultat consigné |
|---|---|
| Mode de démarrage de l’environnement d’installation | UEFI 64 bits |
| Organisation du stockage | Partitions et systèmes de fichiers contrôlés |
| Capacité après redimensionnement | Espace disponible contrôlé sur `/mnt` |
| Installation des paquets | Réussie après correction du stockage |
| Démarrage du système installé | Réalisé |
| Connectivité après démarrage | Test vers une adresse IP réussi |

Le test vers une adresse IP valide la connectivité
vers la destination testée. Il ne démontre pas, à lui seul,
le fonctionnement de la résolution DNS.

## 8. Essais complémentaires

J’ai également effectué des manipulations autour de GNOME,
Xfce et KDE Plasma.

Ces essais sont distincts de l’installation du système de base.
Leurs configurations et leurs résultats ne sont pas détaillés
dans cette page.

## Compétences mobilisées

- Préparation d’une machine virtuelle Linux.
- Partitionnement GPT et préparation des systèmes de fichiers.
- Montage des partitions et configuration de `/etc/fstab`.
- Installation et configuration depuis un environnement chroot.
- Gestion des comptes et des privilèges.
- Configuration du démarrage UEFI avec GRUB.
- Diagnostic et redimensionnement d’un système de fichiers ext4.

## Références

- [Téléchargements officiels Arch Linux](https://archlinux.org/download/)
- [Guide officiel d’installation](https://wiki.archlinux.org/title/Installation_guide)
- [Manuel de resize2fs](https://man.archlinux.org/man/resize2fs.8.en)

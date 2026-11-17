# Debian — Instantanés système avec Timeshift

## Présentation

J’installe et configure Timeshift sur une machine Debian
avec une interface graphique.

Je prépare un emplacement pour les instantanés et travaille
sur les paramètres de création et d’exclusion des fichiers.

Timeshift permet de revenir à un état précédent du système.
Il ne remplace pas une sauvegarde des documents personnels.

## Environnement technique

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Système | Debian 13 |
| Interface | Bureau graphique |
| Démarrage | UEFI |
| Organisation du stockage | LVM avec `/home` séparé |
| Système de fichiers concerné | ext4 |
| Type d’instantané | RSYNC |
| Stockage supplémentaire | Disque virtuel SCSI de 10 Go |

## Objectifs

- Installer Timeshift.
- Identifier un emplacement pour les instantanés.
- Configurer les exclusions des répertoires utilisateurs.
- Lancer la création d’un instantané.
- Préparer un test de restauration du système.

## 1. Préparation de l’environnement

J’installe Fastfetch pour afficher un résumé des informations
du système.

Je vérifie ensuite les disques, leurs points de montage
et l’espace disponible avec `lsblk` et `df`.

Ces vérifications me permettent de choisir où stocker
les instantanés avant de configurer Timeshift.

## 2. Organisation du stockage

Je prévois d’abord d’utiliser une clé USB pour conserver
les instantanés en dehors du disque système.

La clé étant initialement en FAT32, je prépare un formatage
en ext4 pour cet usage.

Linux sait lire le FAT32. Le choix d’ext4 répond ici
aux besoins du stockage des instantanés RSYNC,
notamment aux liens physiques utilisés par Timeshift.

Le formatage efface les données du support :
il faut donc identifier le bon périphérique avant d’agir.

## 3. Détection du périphérique USB

La clé est indiquée comme connectée dans VMware,
mais je ne la retrouve pas dans la machine Debian.

Je vérifie sa présence avec `lsusb`, puis je consulte
les derniers messages du noyau avec `journalctl`.

Je teste également :

- un autre port USB ;
- la reconnexion du périphérique à la VM ;
- le passage du contrôleur USB virtuel de 2.0 à 3.2.

Ces essais ne permettent pas de rendre la clé disponible.
La cause exacte du problème n’est pas confirmée.

## 4. Ajout d’un disque virtuel

Pour poursuivre les manipulations, j’ajoute un disque
virtuel SCSI de 10 Go.

Je vérifie sa détection avec `lsblk` :
il apparaît sous le nom `/dev/sdb`.

J’ouvre ensuite `cfdisk` pour préparer son partitionnement.

Cette solution permet de poursuivre le travail sur le stockage
sans avoir résolu le problème de connexion USB.

Un second disque virtuel reste toutefois dépendant du stockage
de l’hôte VMware. Il ne constitue pas, à lui seul,
une protection contre la panne de cet hôte.

## 5. Installation et configuration de Timeshift

J’installe Timeshift depuis le gestionnaire de logiciels
de Debian.

Dans l’assistant, je sélectionne le mode RSYNC,
adapté au système de fichiers ext4 utilisé.

Ce mode utilise des copies de fichiers et des liens physiques
pour partager les fichiers inchangés entre les instantanés.

Je configure les exclusions pour ne pas inclure
les répertoires personnels des utilisateurs ordinaires.
Le répertoire du compte `root` est traité séparément
dans les paramètres.

Je lance ensuite la création d’un instantané
avec le bouton « Créer ».

## 6. Vérifications et résultats

| Élément | Résultat consigné |
|---|---|
| Vérification du stockage | Disques et espace disponible examinés |
| Connexion de la clé USB | Non résolue |
| Ajout du disque virtuel | Disque `/dev/sdb` détecté |
| Installation de Timeshift | Réalisée |
| Configuration du mode | RSYNC sélectionné |
| Exclusions utilisateurs | Paramétrées dans Timeshift |
| Création d’un instantané | Lancée depuis l’interface |

La destination finale de l’instantané reste à confirmer :
le compte rendu mentionne encore la clé USB après l’ajout
du disque virtuel.

## 7. Restauration et planification

La suite prévue consiste à modifier le nom de la machine,
puis à restaurer l’état précédent pour vérifier le résultat.

Une planification quotidienne avec une conservation
de sept instantanés est également prévue.

Les éléments conservés ne permettent pas de confirmer
la restauration ni le fonctionnement de cette planification.

## État du projet

Timeshift a été installé et configuré, puis une création
d’instantané a été lancée.

Pour terminer la validation, je dois :

- confirmer le support de destination ;
- vérifier que l’instantané est terminé et disponible ;
- tester la restauration et contrôler le résultat ;
- vérifier la planification quotidienne et la conservation.

## Compétences mobilisées

- Identification des disques et de l’espace disponible.
- Recherche d’un périphérique USB dans une VM.
- Consultation des messages du noyau.
- Ajout d’un disque virtuel.
- Configuration de Timeshift et des exclusions.
- Préparation d’un test de restauration.

## Références

- [Projet officiel Timeshift](https://github.com/linuxmint/timeshift)
- [Documentation Debian](https://www.debian.org/doc/)

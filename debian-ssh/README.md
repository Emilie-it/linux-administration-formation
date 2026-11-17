# Debian — Configuration d’un serveur SSH

## Objectif

Préparer une machine Debian administrable à distance :
configurer le réseau, gérer les privilèges et mettre en place
un serveur SSH.

SSH permet d’ouvrir une session distante chiffrée.
Le client initie la connexion ; le serveur SSH la reçoit.

## Environnement

| Élément | Configuration |
|---|---|
| Hyperviseur | VMware Workstation |
| Système serveur | Debian, installation sans environnement graphique |
| Ressources du serveur | 2 processeurs virtuels, 4 Go de RAM, disque de 60 Go |
| Démarrage | UEFI |
| Partitionnement | LVM avec `/home` séparé |
| Réseau virtuel | NAT |
| Nom du serveur | `ssh-server` |
| Adresse IPv4 du serveur | `192.168.134.140/24` |
| Adresse IPv4 du client Linux | `192.168.134.141/24` |
| Passerelle | `192.168.134.2` |
| Port SSH configuré | TCP `2026` |

Ces adresses correspondent au réseau virtuel utilisé pendant
les manipulations.

## 1. Préparation du système

Après l’installation de Debian, j’ai installé `sudo`
et ajouté mon compte utilisateur au groupe `sudo`.

L’objectif était d’utiliser un compte ordinaire pour travailler,
puis d’élever les privilèges uniquement pour les opérations
d’administration.

J’ai également renommé le serveur `ssh-server` et adapté
son entrée dans `/etc/hosts`.

### Précision sur les privilèges

Un utilisateur membre du groupe `sudo` reste un utilisateur
ordinaire tant qu’il n’élève pas ses privilèges.

L’ajout à ce groupe lui donne toutefois des possibilités
d’administration importantes : il doit rester limité
aux comptes qui en ont besoin.

## 2. Configuration réseau

J’ai relevé les adresses et la route par défaut avant de modifier
la configuration.

Le serveur et le client Linux ont ensuite reçu des adresses
IPv4 fixes dans `/etc/network/interfaces`.

| Machine | Adresse | Masque | Passerelle |
|---|---|---|---|
| Serveur SSH | `192.168.134.140` | `255.255.255.0` | `192.168.134.2` |
| Client Linux | `192.168.134.141` | `255.255.255.0` | `192.168.134.2` |

### Point de vigilance

Une adresse attribuée automatiquement par DHCP n’est pas,
par nature, instable.

Le choix d’une adresse fixe répond ici au besoin de retrouver
le serveur à une adresse connue.

Avant de réutiliser manuellement une adresse précédemment
attribuée par DHCP, il faut vérifier la plage du serveur DHCP
virtuel pour éviter un conflit d’adresses. Cette vérification
reste à documenter.

## 3. Vérification du service SSH

Le serveur OpenSSH était installé sur la machine serveur.

J’ai contrôlé l’état du service et recherché son écoute
sur le port TCP 22, avant de modifier sa configuration.

Côté client Linux, j’ai vérifié la présence du programme `ssh`.

### Client et serveur : deux rôles différents

- `openssh-client` fournit les outils pour initier une connexion.
- `openssh-server` permet de recevoir des connexions.
- `ssh` est le programme client.
- `sshd` est le processus serveur.

Une machine utilisée uniquement comme cliente n’a pas besoin
d’héberger un serveur SSH.

## 4. Configuration du serveur SSH

J’ai modifié le fichier `/etc/ssh/sshd_config` pour travailler
sur les paramètres suivants :

| Paramètre | Configuration recherchée |
|---|---|
| `Port` | Écouter sur le port TCP `2026` |
| `ListenAddress` | Écouter sur `192.168.134.140` |
| `PermitRootLogin` | Interdire la connexion SSH directe au compte `root` |

Après les modifications, j’ai redémarré le service SSH.

### Port personnalisé

Changer le port peut réduire certaines tentatives automatisées
visant uniquement le port 22.

Cela ne remplace ni une authentification solide,
ni les mises à jour, ni les restrictions d’accès réseau.

### Adresse d’écoute

`ListenAddress` définit l’adresse locale sur laquelle le serveur
accepte les connexions.

Ce paramètre ne sélectionne pas les clients autorisés.
Le filtrage des adresses sources relève notamment du pare-feu.

## 5. Authentification par clé

J’ai généré une paire de clés sur le client Linux et entrepris
la copie de la clé publique vers le compte distant.

Le principe est le suivant :

- la clé privée reste sur le client et doit être protégée ;
- la clé publique est placée sur le serveur ;
- le serveur vérifie que le client possède la clé privée
  correspondant à une clé publique autorisée.

### Limites de validation

La génération a été effectuée depuis le compte `root` du client.
La paire de clés appartient donc à ce compte local.

Cela n’impose pas une connexion au compte `root` du serveur :
le compte distant est celui indiqué dans la commande SSH.

Les éléments conservés ne suffisent pas à démontrer
une ouverture de session réussie par clé.

La désactivation de l’authentification par mot de passe
est mentionnée dans mon compte rendu initial, mais son efficacité
reste à vérifier.

Je ne considère donc pas l’authentification exclusivement
par clé comme validée.

## 6. Corrections apportées à l’analyse

### Compte local et compte distant

Dans une connexion SSH, le nom placé avant `@` désigne
le compte utilisé sur le serveur.

Il ne désigne pas nécessairement le compte actuellement ouvert
sur le client.

### Détection d’une connexion perdue

J’avais présenté `ClientAliveInterval` et `ClientAliveCountMax`
comme un délai maximal d’inactivité de l’utilisateur.

Ces paramètres servent à détecter un client qui ne répond plus.
Un client connecté peut répondre automatiquement même si
l’utilisateur ne saisit aucune commande.

Ils ne constituent donc pas, à eux seuls, une limite
d’inactivité au clavier.

### Fichier hosts et IPv6

J’avais commenté des entrées IPv6 dans `/etc/hosts`.

Cette modification ne désactive pas IPv6 et n’est pas nécessaire
pour renommer la machine. Elle ne doit pas être présentée
comme une étape normale de configuration de SSH.

## 7. Vérifications restant à documenter

| Vérification | Résultat attendu |
|---|---|
| Configuration SSH valide | Absence d’erreur de syntaxe |
| Écoute du serveur | Adresse et port conformes à la configuration |
| Connexion depuis le client Linux | Ouverture d’une session avec le compte distant prévu |
| Authentification par clé | Connexion réussie sans recours au mot de passe du compte distant |
| Connexion directe à `root` | Refus par le serveur |
| Authentification par mot de passe | Refus si l’accès exclusivement par clé est retenu |
| Connexion depuis Windows | Test réalisé et résultat conservé |

Ces vérifications ne sont pas présentées comme réussies
en l’absence de résultats conservés.

## Bilan

Ce travail m’a permis de manipuler la configuration réseau
de Debian, les privilèges utilisateur et le service OpenSSH.

La relecture a également permis de corriger plusieurs confusions :
client et serveur SSH, compte local et compte distant,
adresse d’écoute et filtrage, connexion perdue et inactivité.

La validation finale de l’authentification a été effectué avec succès.

## Documentation de référence

- [Configuration réseau Debian](https://wiki.debian.org/NetworkConfiguration)
- [Manuel OpenSSH : configuration du serveur](https://man.openbsd.org/sshd_config)
- [Manuel OpenSSH : client SSH](https://man.openbsd.org/ssh)

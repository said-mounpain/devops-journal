# Journal de bord — Semaine 1 (02 au 08 novembre 2026)

> « Ce qui n'est pas écrit n'a pas été fait. »

Dans le cadre de ma formation pour devenir ingénieur DevOps, nous avons un projet fil rouge
qui nous permet de maîtriser de manière pratique les concepts étudiés.
Ce fichier documente les commandes de la **semaine 1**.

## Récapitulatif

| Ticket | Sujet | Statut |
|---|---|---|
| OPS-101 | Journal de bord | ✅ |
| OPS-102 | Onboarding de deux développeurs | ✅ |
| OPS-103 | Le dossier d'équipe | ✅ |
| OPS-104 | Le secret de l'application |✅ |
| OPS-105 | Incident: les photos ne s'envoient plus | ✅ |
| OPS-106 | Défi : déployer sans être administrateur |✅ |

---

## OPS-101 — Journal de bord

**Contexte.** Chez Mboa Market, chaque ingénieur tient un journal de ce qu'il fait sur les serveurs.
C'est ce qui permet de reprendre le travail d'un collègue, et de comprendre un incident après coup.

### Mise en place du dépôt

```bash
ssh-keygen -t ed25519
cat ~/.ssh/id_ed25519.pub
```

→ Clé publique ajoutée dans GitHub, à *Settings → SSH and GPG keys*.

```bash
git clone git@github.com:said-mounpain/devops-journal.git
```

```text
The authenticity of host 'github.com (140.82.121.3)' can't be established.
ED25519 key fingerprint is SHA256:+DiY3wvvV6TuJJhbpZisF/zLDA0zPMSvHdkr4UvCOqU.
Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

> [!NOTE]
> Première connexion SSH à GitHub : l'empreinte a été vérifiée sur la
> [documentation officielle GitHub](https://docs.github.com/en/authentication/keeping-your-account-and-data-secure/githubs-ssh-key-fingerprints)
> avant de répondre `yes`. La clé d'hôte a été ajoutée à `~/.ssh/known_hosts`.

### Configuration de l'identité Git

```bash
git config --global user.name "Said Mounpain"
git config --global user.email "said.mounpain@example.com"
```

> [!TIP]
> Sans cette configuration, `git commit` échoue avec
> `Author identity unknown`. L'option `--global` applique l'identité à tous les dépôts.

### Premier commit

```bash
git add .
git commit -m "docs: Journal de bord des commandes des exercices OPS-101 à OPS-103"
git push
```

---

## OPS-102 — Onboarding de deux développeurs

**Contexte.** Amina (backend) et Koffi (frontend) arrivent lundi.
Chacun a son propre compte : en entreprise, on ne partage jamais un compte.

### 1. Création du groupe `equipe`

```bash
sudo groupadd equipe
```

**Vérification**

```bash
getent group equipe
```

```text
equipe:x:1003:
```

### 2. Création des comptes `amina` et `koffi`

```bash
sudo useradd -m -s /bin/bash amina
sudo useradd -m -s /bin/bash koffi
```

- `-m` → crée le dossier personnel (`/home/amina`, `/home/koffi`)
- `-s /bin/bash` → shell de connexion `bash`

**Vérification**

```bash
cat /etc/passwd | grep amina
cat /etc/passwd | grep koffi
```

```text
amina:x:1003:1004::/home/amina:/bin/bash
koffi:x:1004:1005::/home/koffi:/bin/bash
```

### 3. Ajout des comptes au groupe `equipe`

```bash
sudo usermod -aG equipe amina
sudo usermod -aG equipe koffi
```

> [!WARNING]
> Ne **jamais** oublier le `-a` dans `-aG`. Sans lui, `usermod -G` **remplace**
> la liste des groupes secondaires au lieu de l'enrichir.

**Vérification**

```bash
getent group equipe
```

```text
equipe:x:1003:amina,koffi
```

---

## OPS-103 — Le dossier d'équipe

**Contexte.** L'équipe veut un dossier commun pour ses scripts et ses notes :
`/srv/mboamarket/partage`. Jules, le stagiaire, ne doit ni le lire ni y écrire.

### 1. Création et permissions

```bash
sudo mkdir -p /srv/mboamarket/partage
sudo chown root:equipe /srv/mboamarket/partage
sudo chmod 2770 /srv/mboamarket/partage
```

| Commande | Effet |
|---|---|
| `mkdir -p` | crée l'arborescence complète si besoin |
| `chown root:equipe` | propriétaire `root`, groupe `equipe` |
| `chmod 2770` | `rwx` pour root et le groupe, rien pour les autres, **+ SGID** |

> [!NOTE]
> Le **SGID** (`2`) garantit que tout fichier créé dans le dossier appartiendra
> au groupe `equipe`, pas au groupe personnel de son créateur.

### 2. Vérification des permissions

```bash
ls -l /srv/mboamarket/
```

```text
total 24
-rw-r--r-- 1 root root   5100 Nov  4 19:56 TICKETS.md
drwxr-xr-x 2 root root   4096 Nov  4 19:56 app
drwxr-xr-x 2 root root   4096 Nov  4 19:56 config
drwxrws--- 2 root equipe 4096 Nov  6 23:06 partage
drwxr-xr-x 2 root root   4096 Nov  4 19:56 uploads
```

> [!TIP]
> Le `s` à la place du `x` du groupe (`drwxrws---`) confirme que le **SGID** est actif.

### 3. Tests d'accès

| Utilisateur | Commande | Résultat |
|---|---|---|
| **amina** (membre de `equipe`) | `sudo -u amina touch /srv/mboamarket/partage/amina.txt` | ✅ fichier créé, groupe `equipe` |
| **jules** (non membre) | `sudo -u jules touch /srv/mboamarket/partage/jules.txt` | ❌ `Permission denied` |

**Preuve — amina**

```bash
sudo ls -l /srv/mboamarket/partage/
```

```text
total 0
-rw-rw-r-- 1 amina equipe 0 Nov  6 23:28 amina.txt
```

**Preuve — jules**

```bash
sudo -u jules touch /srv/mboamarket/partage/jules.txt
```

```text
touch: cannot touch '/srv/mboamarket/partage/jules.txt': Permission denied
```

> [!NOTE]
> Le fichier `amina.txt` appartient bien au groupe `equipe` (et non au groupe
> primaire d'amina) : l'héritage du **SGID** fonctionne comme attendu.
> Jules, absent du groupe `equipe`, est correctement bloqué par le `0` des « autres ».

---
## OPS-104 - Le secret de l'application
**Contexte.** L'audit de sécurité a trouvé que `/srv/mboamarket/config/.env`, qui contient le mot de passe de la base de données, est lisible **et modifiable** par tous les comptes de la machine.

**À faire.** Rends ce fichier lisible par l'application seulement.
### 1- verification des droits du fichier actuel
```bash
ls -la /srv/mboamarket/config
```

```text
total 12
drwxr-xr-x 2 root root 4096 Nov  4 19:56 .
drwxr-xr-x 6 root root 4096 Nov  6 22:25 ..
-rw-rw-rw- 1 root root  401 Nov  4 19:56 .env
```
> [!NOTE]
>Le fichier caché `.env` appartient au propriétaire et groupe `root` avec les droits `666`

### 2- On rend le fichier lisible par l'application seulement
```bash
sudo chown root:mboa /srv/mboamarket/config/.env
```
### 3- On vérifie le résultat
```bash
ls -la /srv/mboamarket/config
```
```text
total 12
drwxr-xr-x 2 root root 4096 Nov  4 19:56 .
drwxr-xr-x 6 root root 4096 Nov  6 22:25 ..
-rw-rw-rw- 1 root mboa  401 Nov  4 19:56 .env
```
### 4- On configure les permissions pour quel seul le root et le l'application (mboa) puisse le lire
```bash
sudo chmod 640 /srv/mboamarket/config/.env
```
### 5-On vérifie 
```bash
 ls -la /srv/mboamarket/config
```
```text
total 12
drwxr-xr-x 2 root root 4096 Nov  4 19:56 .
drwxr-xr-x 6 root root 4096 Nov  6 22:25 ..
-rw-r----- 1 root mboa  401 Nov  4 19:56 .env
```
 
### 6-Tests d'accès

| Utilisateur | Commande | Résultat |
|---|---|---|
| **mboa** (membre de `mboa`) | `sudo -u mboa cat /srv/mboamarket/config/.env` | ✅ fic>
| **jules** (non membre) | `sudo -u jules cat /srv/mboamarket/config/.env` | ❌ `Permission>

---
## OPS-105 - Incident: les photos ne s'envoient plus
**Contexte.** Depuis ce matin, le service client reçoit des plaintes : quand un client joint une photo à sa commande, le site affiche une erreur. Tout le reste marche.

### 1. Trouver les erreurs dans les logs de l'application
```bash
 cat /var/log/mboamarket/app.log | grep ERROR
```

```text
2026-11-04 09:09:58 ERROR [upload] PermissionError: [Errno 13] Permission denied: '/srv/mboamarket/uploads/commande-1042-photo.jpg'
2026-11-04 09:09:58 ERROR [web] POST /commandes/1042/photo 500 14ms client=3870
2026-11-04 09:15:30 ERROR [upload] PermissionError: [Errno 13] Permission denied: '/srv/mboamarket/uploads/commande-1043-photo.jpg'
2026-11-04 09:15:30 ERROR [web] POST /commandes/1043/photo 500 11ms client=5102
2026-11-04 09:26:41 ERROR [upload] PermissionError: [Errno 13] Permission denied: '/srv/mboamarket/uploads/commande-1044-photo.jpg'
2026-11-04 09:26:41 ERROR [web] POST /commandes/1044/photo 500 13ms client=2967
2026-11-04 09:40:51 ERROR [upload] PermissionError: [Errno 13] Permission denied: '/srv/mboamarket/uploads/commande-1045-photo.jpg'
2026-11-04 09:40:51 ERROR [web] POST /commandes/1045/photo 500 12ms client=6018
```
> [!NOTE]
> Les erreurs sont des liées aux droits d'accès du dossier /srv/mboamarket/uploads où sont stokées les photos des clients

### 2. On vérifie les droits du dossier uploads
```bash
 ls -la /srv/mboamarket/
```
```text
total 32
drwxr-xr-x 6 root root   4096 Nov  6 22:25 .
drwxr-xr-x 3 root root   4096 Nov  4 19:56 ..
-rw-r--r-- 1 root root   5100 Nov  4 19:56 TICKETS.md
drwxr-xr-x 2 root root   4096 Nov  4 19:56 app
drwxr-xr-x 2 root root   4096 Nov  4 19:56 config
drwxrws--- 2 root equipe 4096 Nov  7 11:58 partage
drwxr-xr-x 2 root root   4096 Nov  4 19:56 uploads
```
> [!NOTE]
> Le dossier uploads n'est pas dans le groupe `mboa` de l'application, seule le `root` a tous les acccès
### 3. on ajoute le dossier au groupe mboa de l'application
```bash
 sudo chown root:mboa /srv/mboamarket/uploads
```
### 4. on vérifie
```bash
 ls -la /srv/mboamarket/
```
```text
total 32
drwxr-xr-x 6 root root   4096 Nov  6 22:25 .
drwxr-xr-x 3 root root   4096 Nov  4 19:56 ..
-rw-r--r-- 1 root root   5100 Nov  4 19:56 TICKETS.md
drwxr-xr-x 2 root root   4096 Nov  4 19:56 app
drwxr-xr-x 2 root root   4096 Nov  4 19:56 config
drwxrws--- 2 root equipe 4096 Nov  7 11:58 partage
drwxr-xr-x 2 root mboa   4096 Nov  4 19:56 uploads
```
### 5. on donne les droits d'accès à l'application (mboa) et à personne d'autre
```bash
sudo chmod 770 /srv/mboamarket/uploads
```
### 6. On vérifie
```bash
 ls -la /srv/mboamarket/
```

```text
total 32
drwxr-xr-x 6 root root   4096 Nov  6 22:25 .
drwxr-xr-x 3 root root   4096 Nov  4 19:56 ..
-rw-r--r-- 1 root root   5100 Nov  4 19:56 TICKETS.md
drwxr-xr-x 2 root root   4096 Nov  4 19:56 app
drwxr-xr-x 2 root root   4096 Nov  4 19:56 config
drwxrws--- 2 root equipe 4096 Nov  7 11:58 partage
drwxrwx--- 2 root mboa   4096 Nov  4 19:56 uploads
```
---
## OPS-106 - Défi : Déployer sans être administrateur
**Contexte.** Koffi doit pouvoir mettre en ligne une nouvelle version avec `/usr/local/bin/mboa-deploy`, qui doit tourner en root. Mais il ne doit pas devenir administrateur de la machine : c'est le principe du **moindre privilège**.

### 1.On écrit  une règle dans sudoers.d permettant à koffi déployer sans être admin
```bash
 sudo visudo -f /etc/sudoers.d/koffi-deploy
```
```text
   on insère la ligne suivante dans l'éditeur
   koffi ALL=(root) NOPASSWD: /usr/local/bin/mboa-deploy
```
> [!TIPE]
>Une erreur de syntaxe dans /etc/sudoers n'est pas un incident bénin : sudo refuse alors le fichier entier et plus personne ne peut élever ses privilèges, y compris vous. Sur un serveur distant sans accès console, cela devient un incident de production. C'est la raison d'être de visudo : il ouvre une copie de travail, verrouille le fichier contre les éditions concurrentes et surtout contrôle la syntaxe avant de remplacer l'original.

### 2.On met le fichier contenant la règle en lecture seule pour le root et le groupe root par sécurité
```bash
sudo chmod  0440 /etc/sudoers.d/koffi-deploy
```
### 3. on vérifie les droits
```bash
 sudo ls -l  /etc/sudoers.d/
```

```text
total 16
-r--r----- 1 root root  139 Nov  3 08:34 90-cloud-init-users
-r--r----- 1 root root   28 Nov  4 22:21 90-labo-said
-r--r----- 1 root root 1068 Jan 29  2024 README
-r--r----- 1 root root   54 Nov  7 14:30 koffi-deploy
```
### 4.on contrôle la synthaxe sans éditer le fichier
```bash
    sudo visudo -c
```
```text
/etc/sudoers: parsed OK
/etc/sudoers.d/90-cloud-init-users: parsed OK
/etc/sudoers.d/90-labo-said: parsed OK
/etc/sudoers.d/README: parsed OK
/etc/sudoers.d/koffi-deploy: parsed OK
```
> [!NOTE]
>chaque fichier doit afficher `parsed OK`. Si une erreur apparaît, la configuration est cassée et plus personne ne peut utiliser sudo.

### On regarde la liste des droits sudo pour koffi
```bash 
 sudo -lU koffi
```

```text
Matching Defaults entries for koffi on said:
    env_reset, mail_badpass,
    secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin,
    use_pty

User koffi may run the following commands on said:
    (root) NOPASSWD: /usr/local/bin/mboa-deploy
```
### On vérifie en lançant deploiement sans mot de passe
```bash
sudo -u koffi sudo -n /usr/local/bin/mboa-deploy
```
```text
→ Récupération de la version v1.261107.1519
→ Copie dans /srv/mboamarket/app
→ Redémarrage de l'application
✓ Mboa Market v1.261107.1519 est en ligne (déployée par koffi).
```
### on vérifie que koffi ne peut lancer aucune autre commande
```bash
  sudo -u koffi  /bin/ls
```
```text
/bin/ls: cannot open directory '.': Permission denied
```
### koffi n'est pas dans le group sudo
```bash
 id koffi
```
```text
uid=1004(koffi) gid=1005(koffi) groups=1005(koffi),1003(equipe)
```
---

## Bilan de la semaine

    Comptes individuels créés pour Amina et Koffi, ajoutés au groupe equipe.

    Dossier partagé /srv/mboamarket/partage opérationnel avec 2770 + SGID.

    Critères d'acceptation vérifiés par des tests concrets (amina ✅ / jules ❌).

    Journal versionné sur GitHub, en Markdown, pour être lisible par l'équipe.

    OPS-104 : /srv/mboamarket/config/.env en root:mboa 640 — lisible par mboa ✅, refusé pour jules ❌.

    OPS-105 : cause trouvée dans app.log (droits d'écriture manquants sur uploads) ; corrigé avec mboa en 750 — mboa écrit ✅, jules refusé ❌.

    OPS-106 : règle /etc/sudoers.d/50-koffi-deploy (koffi ALL=(root) NOPASSWD: /usr/local/bin/mboa-deploy, 440), validée par visudo -c et sudo -lU — koffi lance le déploiement sans mot de passe, aucune autre commande root, pas dans sudo.

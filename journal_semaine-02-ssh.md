# Journal de bord — Semaine 2 (14 au 15 novembre 2026)

> « Ce qui n'est pas écrit n'a pas été fait. »

Dans le cadre de ma formation pour devenir ingénieur DevOps, nous avons un projet fil rouge
qui nous permet de maîtriser de manière pratique les concepts étudiés.
Ce fichier documente les commandes de la **semaine 2**.
>Mboa Market automatise ses déploiements. La CI, le robot qui teste et publie le code, entrera sur la passerelle de déploiement avec sa propre clé, sous un compte à elle : deploy.


## Récapitulatif

| Ticket | Sujet | Statut |
|---|---|---|
| OPS-401 |Le compte deploy sans mot de passe | ✅ |
| OPS-402 |La clé de la CI ouvre le compte deploy | ✅ |
| OPS-403 |Durcir le ssh de la passerelle | ✅ |
| OPS-404 |Seul deploy entre sur la passerelle |✅ |
| OPS-405 | Mboa-comptes commerce comme un vrai script | ✅ |
| OPS-406 | Mboa-comptes se relance sans rien casser |✅ |
| OPS-407 |mboa-comptes s'arrête net si la liste manque |✅ |
| OPS-406 | La clé de la CI ne fait qu'une chose |✅ |

**Idée directrice du TP (comprise après coup).** Construire un accès de bout en bout en appliquant le **principe du moindre privilège** à chaque couche :

| Couche | Ticket | Question à laquelle le ticket répond |
|---|---|---|
| Identité | OPS-401 | *Qui* est la CI ? |
| Authentification | OPS-402 | *Comment* entre-t-elle ? |
| Surface d'attaque | OPS-403 | Quelles voies d'entrée *fermer* ? |
| Autorisation | OPS-404 | *Qui* a le droit d'entrer ? |
| Outillage | OPS-405–407 | L'outil qui crée les comptes est-il *fiable* ? |
| Exécution | OPS-408 | Que peut faire la CI *une fois entrée* ? |

---

## OPS-401 — Le compte deploy sans mot de passe 

**contexte**: La CI a besoin d'un compte à elle, pas du tien.

### 1. création  du compte deploy
```bash
	sudo useradd -m -s /bin/bash
	cat /etc/passwd | grep deploy
	sudo passwd -S deploy
```
```text
	deploy:x:1007:1009::/home/deploy:/bin/bash
	deploy L 11/15/2026 0 99999 7 -1

```
---

## OPS-402 - La clé de la CI ouvre le compte deploy
**contexte**:Le compte deploy n'a pas de mot de passe (OPS-401). La CI doit donc s'y connecter par clé SSH, sans mot de passe. La clé privée de la CI est /var/lib/labo-tp04/ci_ed25519 (lisible par root seulement) ; sa clé publique est /opt/mboa/cles/ci.pub.

### 1. on crée le dossier ssh et authorized_keys dans le home de deploy
```bash
	sudo mkdir /home/deploy/.ssh
	sudo chown deploy:deploy /home/deploy/.ssh
	sudo chmod 700 /home/deploy/.ssh
	sudo touch /home/deploy/.ssh/authorized_keys
        sudo chown deploy:deploy /home/deploy/.ssh/authorized_keys
        sudo chmod 600 /home/deploy/.ssh/authorized_keys
	sudo -u deploy ls -al /home/deploy/.ssh

```
```text
total 8
drwx------ 2 deploy deploy 4096 Nov 15 10:44 .
drwxr-x--- 3 deploy deploy 4096 Nov 15 10:44 ..
-rw------- 1 deploy deploy    0 Nov 15 10:44 authorized_keys

```
>[!TIP]
> les droits du dossier `.ssh` doivent être en 700 et le fichier `authorized_keys` en 600 et tout appartient à `deploy`

### 2. Copie de la clé public de CI dans deploy

```bash 
	cat /opt/mboa/cles/ci.pub
	sudo -u deploy nano /home/deploy/.ssh/authorized_keys
	sudo cat /home/deploy/.ssh/authorized_keys 
```
```text
ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAILJ6X5h16xPjL5pIE3arbSQRBUijSGU395PZlgHj826L ci@mboa-mark>
```
### 3. on test la connexion via ssh de CI a deploy

```bash
	sudo ssh -i /var/lib/labo-tp04/ci_ed25519 -p 2222 deploy@127.0.0.1
```
```text
	The authenticity of host '[127.0.0.1]:2222 ([127.0.0.1]:2222)' can't be established.
	ED25519 key fingerprint is SHA256:1QHVSvL/R9xT27OHsPbmtNUF/cJ0Slg5bUSb8/TYfng.
	This key is not known by any other names.
	Are you sure you want to continue connecting (yes/no/[fingerprint])? yes
	Warning: Permanently added '[127.0.0.1]:2222' (ED25519) to the list of known hosts.

```
---

## OPS-3 - Durcir le SSH de la passerelle
**context** La passerelle a été réglée « pour que ça marche ». Maintenant que la clé de la CI fonctionne (OPS-402), on ferme tout ce qui n'est pas la clé.
**Objectif**. La commande sudo sshd -T -f /etc/mboa/passerelle/sshd_config affiche :
```text
	 passwordauthentication no
    	 kbdinteractiveauthentication no
   	 permitrootlogin no
    	maxauthtries 3          (ou moins)
    	logingracetime 30       (ou moins)
    	x11forwarding no
```
### 1. on verifie les Includes ans le fichier /etc/mboa/passerelle/sshd_config
```bash
 sudo nano /etc/mboa/passerelle/sshd_config
```

```text
...
Include /etc/mboa/passerelle/sshd_config.d/*.conf
PermitRootLogin yes
MaxAuthTries 10
LoginGraceTime 120
X11Forwarding yes
...
```
>[!NOTE]
> on constate que qu'il inclue tous les fichiers de configurations présents dans sshd_config.d
> Donc toutes les configurations présentes dans ce dossier on la priorité sur cette présente dans sshd_config
### 2. On modifie le fichie tel que voulu plus haut et on ajoute kdinteractiveauthentification
 
```text
PermitRootLogin no
MaxAuthTries 3
LoginGraceTime 30
X11Forwarding no
kbdinteractiveauthentication no
	
```
### 3. On regarde dans les fichiers de config inclus dans sshd_config.d
```bash
  sudo nano  /etc/mboa/passerelle/sshd_config.d/50-cloud-init.conf 
 ```
```text
	passwordAuthentication yes

```
> Cette configuration devait être prioritaire car elle vient avant celles du fichier sshd_config
> on le met a `no` 
### 4. on vérifie les configurations réellement apliquées

```bash
sudo sshd -T -f /etc/mboa/passerelle/sshd_config | grep 'passwordauthentication|kbdinteractive|permitrootlogin|maxauthtries|logingracetime|x11forwarding

```
```text
maxauthtries 3
logingracetime 30
permitrootlogin no
x11forwarding no
passwordauthentication no
kbdinteractiveauthentication no

```
>[!NOTE]
> on a fermé toutes les autres entrées en dehors du ssh
---

## OPS-404 Seul deploy entre sur la passerelle
**Contexte** Le compte stagiaire a une clé qui ouvre la passerelle. Il ne devrait pas. Terminé quand : deploy entre sur la passerelle ; stagiaire n'y entre plus, ni personne d'autre.
### 1. on filtre ceux qui peuvent se connecter avec AllowUsers
```bash
 sudo nano /etc/mboa/passerelle/sshd_config
```text
...
AllowUsers deploy
...
```
### 2. on essaye de se connecter avec le satagiare
```bash
 sudo ssh -i /var/lib/labo-tp04/stagiaire_ed25519 -p 2222 stagiaire@127.0.0.1
```
```text
stagiaire@127.0.0.1: Permission denied (publickey).
```

---
## OPS-405  - Script pour créer les comptes automatiquement
```ini
#!/usr/bin/env bash
 2 # mboa-comptes : crée un compte pour chaque membre de l'équipe de Mboa Market.
 3 # Usage : sudo mboa-comptes [liste]      (par défaut : /opt/mboa/equipe.txt)
 4 set -euo pipefail
 5
 6 liste="${1:-/opt/mboa/equipe.txt}"
 7
 8 # 1. La liste doit exister (sera durci en OPS-407, mais utile dès maintenant)
 9 if [ ! -f "$liste" ]; then
10     echo "mboa-comptes : liste introuvable : $liste" >&2
11     exit 1
12 fi
13
14 # 2. Lecture ligne par ligne, robuste aux \r Windows et à la dernière ligne sans \n
15 while IFS= read -r ligne || [ -n "$ligne" ]; do
16     # 2a. enlever le \r final (ligne Windows)
17     ligne="${ligne%$'\r'}"
18
19     # 2b. trim des espaces/tabulations en début et fin
20     ligne="${ligne#"${ligne%%[![:space:]]*}"}"
21     ligne="${ligne%"${ligne##*[![:space:]]}"}"
22
23     # 2c. ignorer ligne vide et commentaire
24     [ -z "$ligne" ] && continue
25     case "$ligne" in
26         \#*) continue ;;
27     esac
28
29     # 2d. idempotence : ne créer que si le compte n'existe pas
30     if id -u "$ligne" >/dev/null 2>&1; then
31         echo "compte déjà présent : $ligne"
32     else
33         useradd -m -s /bin/bash "$ligne"
34         echo "compte créé : $ligne"
35     fi
36 done < "$liste"
37
38 echo "terminé"

```
---
## OPS-406 - La clé de la CI ne fait qu'une chose
Contexte: Si quelqu'un vole la clé de la CI, il ne doit pouvoir que lancer un
déploiement : ni shell, ni autre commande, ni tunnel.
```ini
restrict,command="/usr/local/bin/mboa-deployer" ssh-ed25519 AAAAC3NzaC1lZDI1NTE>
``` 
>[!NOTE]
> On ajoute `restrict`  devant la clé public et on spécifie la commande unique permise à deploy

```bash
sudo ssh -i /var/lib/labo-tp04/ci_ed25519 -p 2222 deploy@127.0.0.1
``` 
```text
PTY allocation request failed on channel 0
mboa-deployer : déploiement de Mboa Market sur said, le 18/11/2026 à 20:46
Connection to 127.0.0.1 closed.
```

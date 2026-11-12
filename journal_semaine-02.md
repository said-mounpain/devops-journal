# Journal de bord — Semaine 2 (09 au 11 novembre 2026)

> « Ce qui n'est pas écrit n'a pas été fait. »

Dans le cadre de ma formation pour devenir ingénieur DevOps, nous avons un projet fil rouge
qui nous permet de maîtriser de manière pratique les concepts étudiés.
Ce fichier documente les commandes de la **semaine 2**.

## Récapitulatif

| Ticket | Sujet | Statut |
|---|---|---|
| OPS-201 |Le serveur rame | ✅ |
| OPS-202 |Les outils de l'équipe | ✅ |
| OPS-203 |L'API ne démarre plus | ✅ |
| OPS-204 | La sauvegarde toutes les 5 minutes |✅ |
| OPS-205 | Incident : la transaction en échec | ✅ |
| OPS-206 | Défi : le disque se remplit |✅ |

---

## OPS-201 — Le serveur rame

**Contexte.** Depuis vendredi, tout est lent sur la pré-production. Quelqu'un a lancé un programme à la main avant de partir en week-end.

### On affiche les processus qui consomme le CPU par ordre décroissant

```bash
ps aux --sort=-%cpu | head
```
```text
	USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
mboa       52146 59.8  0.0   7744  3448 ?        Rs   Nov07 3185:09 /bin/bash /usr/local/bin/mboa-miniatures
said       73042  0.0  0.0  15260  7404 ?        R    10:26   0:00 sshd: said@pts/2
said       72916  0.0  0.0  14968  7148 ?        S    10:09   0:00 sshd: said@pts/0
said       73043  0.0  0.0   9196  5528 pts/2    Ss   10:26   0:00 -bash
root       73016  0.0  0.0      0     0 ?        I    10:23   0:00 [kworker/u8:1-flush-8:0]
root       72868  0.0  0.0      0     0 ?        I    10:00   0:00 [kworker/u8:4-events_power_efficient]
root       18640  0.0  0.3 223592 27332 ?        SLsl Nov04   1:48 /sbin/multipathd -d -s
root       73074  0.0  0.0      0     0 ?        I    10:30   0:00 [kworker/2:2-events]
said       73006  0.0  0.0   9064  5428 pts/1    Ss+  10:22   0:00 -bash

```
>[!NOTE]
> On constate que le programme mboa-miniature de l'utilisateur mboa occupe presue 60% du CPU
> On pouvait aussi utiliser la commande `top` pour voir cela en temps réel
> Le programme tourne depuis le `07 novembre` avec le `PID 52146`
### On tue le processus normalement
```bash
  sudo kill 52146
```
### On vérifie si le processus est mort

```bash
ps aux --sort=-%cpu | head
```
```text
USER         PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
said       73042  0.0  0.0  15260  7404 ?        S    10:26   0:00 sshd: said@pts/2
said       72916  0.0  0.0  14968  7148 ?        S    10:09   0:00 sshd: said@pts/0
said       73043  0.0  0.0   9196  5528 pts/2    Ss   10:26   0:00 -bash
root       73016  0.0  0.0      0     0 ?        I    10:23   0:00 [kworker/u8:1-events_power_efficient]
root       72868  0.0  0.0      0     0 ?        I    10:00   0:00 [kworker/u8:4-events_power_efficient]
root       18640  0.0  0.3 223592 27332 ?        SLsl Nov04   1:48 /sbin/multipathd -d -s
root       73074  0.0  0.0      0     0 ?        I    10:30   0:00 [kworker/2:2-events]
root          17  0.0  0.0      0     0 ?        I    Nov03   1:38 [rcu_preempt]
said       72917  0.0  0.0   9196  5608 pts/0    Ss+  10:09   0:00 -bash

```
### On vérifie 

```bash
 uptime
```
```text
 10:39:58 up 8 days,  2:06,  3 users,  load average: 0.38, 0.47, 0.54
 10:41:03 up 8 days,  2:07,  3 users,  load average: 0.12, 0.37, 0.51
 10:55:55 up 8 days,  2:22,  5 users,  load average: 0.00, 0.01, 0.17
```

> [!NOTE]
> On constate que la moyenne des processeur utilisé `load average` décroit avec le temps



---

## OPS-202 — Les outils de l'équipe

**Contexte.** L'équipe veut les mêmes outils d'observation sur tous les serveurs : `htop` pour les processus, `ncdu` pour l'espace disque.

### 1.Installation des outils

```bash
sudo apt update
sudo apt install htop
sudo apt install ncdu
```


**Vérification**

```bash
apt show htop

```

```text
Package: htop
Version: 3.3.0-4build1
Priority: optional
Section: utils
...
```
### On regarde les fichiers posés par ncdu
```bash
dpkg -L ncdu
```
```text
/.
/usr
/usr/bin
/usr/bin/ncdu
/usr/share
/usr/share/doc
/usr/share/doc/ncdu
/usr/share/doc/ncdu/changelog.Debian.gz
/usr/share/doc/ncdu/copyright
/usr/share/man
/usr/share/man/man1
/usr/share/man/man1/ncdu.1.gz
```

---

## OPS-203 — L'API ne démarre plus

**Contexte.** L'API de la boutique (`/srv/mboamarket/app/api.py`) tourne comme un service systemd : `mboa-api`. Elle est tombée, et elle ne repart pas. Le fichier du service, `/etc/systemd/system/mboa-api.service`, a été modifié à la main vendredi :
 il contient **deux** erreurs.

### 1.On vérifie l'état du service

```bash
systemctl status mboa-api
```
```text
× mboa-api.service - API de Mboa Market (préproduction)
     Loaded: loaded (/etc/systemd/system/mboa-api.service; disabled;>
     Active: failed (Result: exit-code) since Sat 2026-11-07 17:58:3>
   Duration: 2ms
   Main PID: 52160 (code=exited, status=203/EXEC)
        CPU: 1ms
```


> [!NOTE]
> Le service est en état d'échec avec le code d'erreur  **status=203/EXEC**.
> Ceci suggère une érreur dans l'écriture du service;
### 2. On fouille dans le journal du service pour en savoir plus
```bash
sudo journalctl -u mboa-api
```
```text
Nov 07 17:58:21 said systemd[1]: Started mboa-api.service - API de Mboa Market (préproduction).
Nov 07 17:58:21 said systemd[1]: mboa-api.service: Main process exited, code=exited, status=203/EXEC
Nov 07 17:58:21 said systemd[1]: mboa-api.service: Failed with result 'exit-code'.
Nov 07 17:58:26 said systemd[1]: mboa-api.service: Scheduled restart job, restart counter is at 1.
Nov 07 17:58:26 said systemd[1]: Started mboa-api.service - API de Mboa Market (préproduction).
Nov 07 17:58:26 said systemd[1]: mboa-api.service: Main process exited, code=exited, status=203/EXEC
Nov 07 17:58:26 said systemd[1]: mboa-api.service: Failed with result 'exit-code'.
Nov 07 17:58:31 said systemd[1]: mboa-api.service: Scheduled restart job, restart counter is at 2.
Nov 07 17:58:31 said systemd[1]: Started mboa-api.service - API de Mboa Market (préproduction).
Nov 07 17:58:31 said systemd[1]: mboa-api.service: Main process exited, code=exited, status=203/EXEC
Nov 07 17:58:31 said systemd[1]: mboa-api.service: Failed with result 'exit-code'.
Nov 07 17:58:37 said systemd[1]: mboa-api.service: Scheduled restart job, restart counter is at 3.
Nov 07 17:58:37 said systemd[1]: mboa-api.service: Start request repeated too quickly.
Nov 07 17:58:37 said systemd[1]: mboa-api.service: Failed with result 'exit-code'.
Nov 07 17:58:37 said systemd[1]: Failed to start mboa-api.service - API de Mboa Market (préproduction).
```
> On constate une intérruption de service avec ` status=203/EXEC` ce qui suggère une erreur dans le fichier service 
### 3.On ouvre le fichier service mboa

```bash
 sudo nano /etc/systemd/system/mboa-api.service
```

```ini
[Unit]
Description=API de Mboa Market (préproduction)
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
User=mboa
Group=mboa
WorkingDirectory=/srv/mboamarket/ap
EnvironmentFile=-/srv/mboamarket/config/.env
ExecStart=/usr/bin/python  api.py
Restart=on-failure
RestartSec=5

[Install]
WantedBy=multi-user.target


```

> [!WARNING]
> On peut observer que le chemin vers `api` n'est pas absolu 
> il faut vérifier la librairie python installé dans notre machine

### 3. On vérifie le python installé dans notre machine

```bash
 ls /usr/bin/ | grep python

```

```text
pybabel-python3
python3
python3.12

```
> [!NOTE]
> On observe que c'est python3 qui est installé et donc on doit éxécuter le fichier avec `python3`

### 4. On modifie le fichier de service

```bash
sudo nano /etc/systemd/system/mboa-api.service 

```

```ini
[Unit]
Description=API de Mboa Market (préproduction)
After=network.target
StartLimitIntervalSec=60
StartLimitBurst=3

[Service]
User=mboa
Group=mboa
WorkingDirectory=/srv/mboamarket/ap
EnvironmentFile=-/srv/mboamarket/config/.env
ExecStart=/usr/bin/python3 /srv/mboamarket/app/api.py
Restart=on-failure
RestartSec=5

```

### 5. On vérifie les droits du fichier api 
```bash
ls -l /srv/mboamarket/app/api.py
```
```text
-rw-r----- 1 root root 600 Nov  4 19:56 /srv/mboamarket/app/api.py
```
> [!NOTE]
> l'utilisateur `mboa` n'a aucun droit sur le fichier `api.py` 
> le fichier n'est pas éxécutable

### 6. On attribut les droits d'éxécution à l'utilisateur mboa 
```bash
 sudo chown root:mboa /srv/mboamarket/app/api.py
 sudo chmod 770 /srv/mboamarket/app/api.py
```
```text
-rwxrwx--- 1 root mboa 1688 Nov  8 00:25 /srv/mboamarket/app/api.py

```
### 7. On relance le servcie
```bash
sudo systemctl daemon-reload 
sudo systemctl restart mboa-api
systemctl status mboa-api
```
```text
● mboa-api.service - API de Mboa Market (préproduction)
     Loaded: loaded (/etc/systemd/system/mboa-api.service; disabled; preset: enabled)
     Active: activating (auto-restart) (Result: exit-code) since Wed 2026-11-11 12:38:17 CET; 1s ago
    Process: 79308 ExecStart=/usr/bin/python3 /srv/mbomarket/app/api.py (code=exited, status=200/CHDIR)
   Main PID: 79308 (code=exited, status=200/CHDIR)
        CPU: 2ms

```
>[!NOTE]
> le service a une erreur `status=200/CHDIR` ce qui suggère une erreur dans le fichier du service
> `200/CHDIR` est une erreur en lien avec la ligne `WorkingDirectory`

### 8. on cherche le contenu de WorkingDirectory

```bash
   cat /etc/systemd/system/mboa-api.service | grep "WorkingDirectory"
```
```text
    WorkingDirectory=/srv/mboamarket/ap
``` 
> [!NOTE] on constate qu'il manque un p à `app`
### 9. On relance le service après modification 
```bash
systemctl status mboa-api
```
```text
systemctl status mboa-api
● mboa-api.service - API de Mboa Market (préproduction)
     Loaded: loaded (/etc/systemd/system/mboa-api.service; disabled; preset: enabled)
     Active: active (running) since Wed 2026-11-11 12:46:18 CET; 8s ago
   Main PID: 79475 (python3)
      Tasks: 1 (limit: 9483)
     Memory: 9.5M (peak: 9.5M)
        CPU: 114ms
     CGroup: /system.slice/mboa-api.service
             └─79475 /usr/bin/python3 /srv/mboamarket/app/api.py

```
### 10. on fait en sorte que le service redémarre avec la machine
```bash
sudo systemctl enable --now mboa-api.service
```
```text
Created symlink /etc/systemd/system/multi-user.target.wants/mboa-api.service → /etc/systemd/system/mboa-api.service.
said@said:~$ systemctl status mboa-api
● mboa-api.service - API de Mboa Market (préproduction)
     Loaded: loaded (/etc/systemd/system/mboa-api.service; enabled; >
     Active: active (running) since Wed 2026-11-11 12:46:18 CET; 2mi>
   Main PID: 79475 (python3)
      Tasks: 1 (limit: 9483)
     Memory: 9.5M (peak: 9.5M)
        CPU: 172ms
     CGroup: /system.slice/mboa-api.service
             └─79475 /usr/bin/python3 /srv/mboamarket/app/api.py

```
---
## OPS-204 - La sauvegarde toutes les 5 minutes

**Contexte.** Les commandes du jour sont dans `/srv/mboamarket/data/commandes.csv`, et rien ne les sauvegarde. L'équipe veut une copie toutes les 5 minutes, avec des traces dans le journal.

### 1- On écrit le script qui sauvegarde les commandes
```bash
  sudo nano /usr/local/bin/mboa-sauvegarde
```

```text
\#!/bin/bash

dossier="/var/backups/mboamarket"

if [[ -d "$dossier" ]]; then
	echo "le dossier existe"
else

	mkdir -p /var/backups/mboamarket
fi

date_jour=$(date '+%Y%m%d-%H%M')

cp /srv/mboamarket/data/commandes.csv "$dossier/commandes-${date_jour}.csv"

echo "Le fichiercommandes-${date_jour} a bien été sauvegardé"

```
> [!NOTE] 
>le caractère `\` devant le shebang n'existe pas c'est juste pour eviter que makedown interprête cella comme un titre

### 2- On crée un service qui va exécuter le bash précédent
```bash
sudo nano /etc/systemd/system/mboa-sauvegarde.service
```
```ini
[Unit]
Description=Backup des commandes tous les 5 min


[Service]
User=mboa
Group=mboa
Type=oneshot
ExecStart=/bin/bash /usr/local/bin/mboa-sauvegarde

```
> [!NOTE]
> le service doit être éxécuté par l'utilisateur `mboa` et donc de groupe `mboa`. De plus, il doit s'éxécuter une seule fois `Type=oneshot`.
> il faut aussi s'assurer du chemin absolu de votre bash sur votre machine car `/usr/bin/bash` marche chez certains mais d'autre comme moi c'est ce qui est écrit plus haut.
### 3- On crée le timer du service mboa-sauvegarde qui va le lancer toutes les 5 minutes
```bash
sudo nano /etc/systemd/system/mboa-sauvegarde.timer

```
```ini
[Unit]
Description=Sauvegarde toute les 5 min

[Timer]
OnCalendar=*:0/5
Persistent=true

[Install]
WantedBy=timers.target

```
> [!TIP]
> c'est important de mettre `Persistent=true` pour que le timer puisse rattraper les tâches non faites en cas d'arrêt de la machine.
> Vérifier la prochaine date du lancement du service avec `systemd-analyze calendar`
### 4- On active le timer 
```bash
sudo systemctl daemon-reload
sudo systemctl enable --now  mboa-sauvegarde.timer
sudo systemctl status mboa-sauvegarde.timer
```

```text
● mboa-sauvegarde.timer - Sauvegarde toute les 5 min
     Loaded: loaded (/etc/systemd/system/mboa-sauvegarde.timer; enabled; preset: enabled)
     Active: active (waiting) since Wed 2026-11-11 22:17:49 CET; 22h ago
    Trigger: Thu 2026-11-12 21:00:00 CET; 3s left
   Triggers: ● mboa-sauvegarde.service

Nov 11 22:17:49 said systemd[1]: Started mboa-sauvegarde.timer - Sauvegarde toute les 5 min.

```
> [!NOTE]
> le système va redémarrer dans 3s (`3s left`)
### 5-On vérifie les sauvegardes précédentes
```bash
sudo ls /var/backups/mboamarket/ | tail -5

```
```text
commandes-20261112-2045.csv
commandes-20261112-2050.csv
commandes-20261112-2055.csv
commandes-20261112-2100.csv
commandes-20261112-2105.csv

```
 
---
## OPS-205 -Incident : la transaction en échec


**Contexte.** Le service client a reçu l'appel d'un client : il a été débité, mais sa commande n'existe pas. Le module de paiement écrit ses messages dans le journal du serveur, sous le nom `mboa-paiement`. Une seule transaction a vraiment échoué : c'est une **erreur**, pas un simple avertissement.

### 1. On regarde le journal de mboa-paiement 
```bash
 sudo journalctl -t mboa-paiement -p err
```

```text
Nov 07 17:58:21 said mboa-paiement[52026]: Transaction TX-94622 en échec : le prestataire de paiement n'a pas répondu (délai dépassé,>

```
> [!NOTE] 
>`-p err` permet de faire un filtre par priorité d'erreur: emerg, alert, crit et err. Il faut gérer ce genre d'érreur tout de suite après constat. 
> On a l'heure et la date à laquelle l'incident s'est produit ainsi que la raison on peut maintenant traité la requête du client à partir du numéro de transaction `TX-94622`.

---
## OPS-206 - Défi : le disque se remplit

**Contexte.** L'alerte « disque » sonne. Les exports des clients, dans `/srv/mboamarket/exports`, ont grossi d'un coup, alors que chaque export pèse quelques kilo-octets. Les exports, l'application en a besoin : on n'efface pas tout.


### 1.On vérifie les tailles des sous dossiers de exports
```bash
 du -ah  /srv/mboamarket/exports | sort -h | tail

```
> [!TIP] 
>On identifie le fichier avec une taille anormale et on le suprime avec `rm`.
> Utiliser `sudo lsof -L1` pour voir si le dossier n'est pas utilisé par un procéssus encours et tué le si c'est le cas pour le noyaux libère effectivement le disque dur.

---

## Bilan de la semaine


    Journal versionné sur GitHub, en Markdown, pour être lisible par l'équipe.

    Ecriture des services et timers
    
    Dépannage des services en pannes

    Analyse des incidents avec journalctl
    
    Liération d'un disque saturé

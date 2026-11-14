# Journal de bord — Semaine 2 (11 au 13 novembre 2026)

> « Ce qui n'est pas écrit n'a pas été fait. »

Dans le cadre de ma formation pour devenir ingénieur DevOps, nous avons un projet fil rouge
qui nous permet de maîtriser de manière pratique les concepts étudiés.
Ce fichier documente les commandes de la **semaine 2**.
#Atelier projet pro
> Atelier « administrer un serveur Linux ». La boutique en ligne de Mboa Market arrive sur ce serveur de **pré-production** : son site doit être servi par **Nginx**, avec l'API du stock derrière. Nadia, la webdesigneuse, rejoint l'équipe. Puis Koffi publie la version de novembre…


## Récapitulatif

| Ticket | Sujet | Statut |
|---|---|---|
| OPS-301 |Le compte de Nadia | ✅ |
| OPS-302 |Le dossier du site | ✅ |
| OPS-303 |Nginx sur le serveur | ✅ |
| OPS-304 |Nadia livre la boutique |✅ |
| OPS-305 |la boutique en ligne, avec nginx | ✅ |
| OPS-306 | Défi : le disque se remplit |✅ |

---

## OPS-301 — Le compte de Nadia

**Contexte.** Nadia, webdesigneuse, arrive demain. C'est elle qui mettra à jour les pages du site, sans être administratrice du serveur.

### On crée le groupe web ains que le compte de Nadia
```bash
  sudo groupadd web
  sudo useradd -m -s /bin/bash -G web nadia
  id nadia
```
```text
  uid=1005(nadia) gid=1007(nadia) groups=1007(nadia),1006(web)

```
---
## OPS-302 - le dossier du site

**Contexte.** Le site vivra dans `/srv/mboamarket/site`. L'équipe web y écrit, Nginx doit pouvoir le lire, et personne d'autre n'y écrit.
### 1. On crée le dossier site dans mboamarket

```bash
  sudo mkdir /srv/mboamarket/site
  sudo chown root:web /srv/mboamarket/site
  sudo chmod 2775 /srv/mboamarket/site
  ls -ld /srv/mboamarket/site
```
```text
  drwxr-xr-x 13 root root 4096 Nov  8 16:10 /srv/mboamarket/
```
### 2. On vérifie que nadia peut bien créer un fichier dans le dossier site

```bash
  sudo -u nadia touch /srv/mboamarket/site/essai
  ls -l /srv/mboamarket/site
```
```text
total 16
-rw-rw-r-- 1 nadia web    0 Nov 14 11:54 essai
drwxr-sr-x 2 nadia web 4096 Nov  8 18:42 images
-rw-r--r-- 1 nadia web 1848 Nov  8 18:42 index.html
-rw-r--r-- 1 nadia web 1412 Nov  8 18:42 stock.js
-rw-r--r-- 1 nadia web 2137 Nov  8 18:42 style.css

```
>[!TIP]
> Un nouveau fichier part de 666 et un nouveau dossier de 777, moins le umask. Sur Ubuntu, ton compte a
>l’umask 0002 : tes fichiers naissent en 664 (rw-rw-r--) et tes dossiers en 775. Avec sudo, l’umask devient
>0022 : 644 et 755.

---

## OPS-303 - Nginx sur le serveur

**Contexte.** L'équipe a choisi Nginx comme serveur web : il sert les pages du site, et passe les appels d'API au bon service.

### 1. Mise à jour des paquets et installation de nginx

```bash
  sudo apt update
  sudo apt install -y nginx
  apt show nginx
```
```text
Package: nginx
Version: 1.24.0-2ubuntu7.19
Priority: optional
Section: web
Origin: Ubuntu
...
```
### 2. On vérifie qu'il tourne et redémarre avec la machine
```bash
  systemctl status nginx
```
```text
sudo systemctl status nginx
● nginx.service - A high performance web server and a reverse proxy server
     Loaded: loaded (/usr/lib/systemd/system/nginx.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-11-14 12:52:51 CET; 25ms ago
       Docs: man:nginx(8)
    Process: 105599 ExecStartPre=/usr/sbin/nginx -t -q -g daemon on; master_process on; (c>
    Process: 105601 ExecStart=/usr/sbin/nginx -g daemon on; master_process on; (code=exite>
   Main PID: 105602 (nginx)
      Tasks: 5 (limit: 9483)
     Memory: 3.7M (peak: 4.2M)
        CPU: 39ms

```
### 3. On vérifie le service qui occupe le port 80
```bash
 sudo ss -tlnp | grep ":80"
```
```text
LISTEN 0      5          127.0.0.1:8080      0.0.0.0:*    users:(("python3",pid=102798,fd=3))     
```
>nginx n'écoute pas au port 80

####  📝 Incident : Échec de démarrage de Nginx après réinstallation

**Symptômes :**
* `nginx.service` affiché comme `active (running)` par systemd.
* La commande `ss -tlnp | grep nginx` ne renvoie aucune écoute sur le port 80.

**Cause racine :** 
Après une suppression accidentelle des fichiers de configuration par défaut (`sites-enabled/default`), une instance "fantôme" du processus maître Nginx est restée bloquée. Systemd la considérait comme active, mais elle n'avait plus de workers actifs pour écouter sur le port 80.

**Résolution :**
1.Activation via un lien symbolique dans `sites-enabled`.
2. Validation de la syntaxe : `sudo nginx -t` (succès).
3. **Action décisive :** Forcer l'arrêt complet du service fantôme et le redémarrer.
   ```bash
   sudo systemctl stop nginx
   sudo systemctl start nginx
### 4.on test la commande curl
```bash
 curl -I http://localhost"
```
```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sat, 14 Nov 2026 12:06:09 GMT
Content-Type: text/html
Content-Length: 632
Last-Modified: Sun, 08 Nov 2026 16:53:11 GMT
Connection: keep-alive
ETag: "6af0a8f7-278"
Accept-Ranges: bytes

```
### 5. on regarde les processus de nginx et leurs comptes 
```bash
ps -o user,pid,ppid,cmd -C nginx 
```
```text
USER         PID    PPID CMD
root      105720       1 nginx: master process /usr/sbin/nginx -g daemon on; master_process
www-data  105721  105720 nginx: worker process
www-data  105722  105720 nginx: worker process
www-data  105723  105720 nginx: worker process
www-data  105724  105720 nginx: worker process

```
>[!NOTE]
> Seul le processus `master` est en root ce qui protège le système des intrusions

---
## OPS-304 -Nadia livre la boutique
**Contexte.** Nadia a livré la première version du site : `/srv/mboamarket/livraisons/site-v1/`. C'est elle qui la dépose dans le dossier du site : les fichiers seront à elle, et au groupe `web`.
### 1. copie du contenu de la livraison de la machine de nadia vers le serveur
```bash
sudo -u nadia cp -r /srv/mboamarket/livraisons/site-v1/. /srv/mboamarket/site/
ls -l /srv/mboamarket/site /srv/mboamarket/site/images
```
```text
total 16
drwxr-sr-x 2 nadia web 4096 Nov  8 18:42 images
-rw-r--r-- 1 nadia web 1848 Nov 14 14:02 index.html
-rw-r--r-- 1 nadia web 1412 Nov 14 14:02 stock.js
-rw-r--r-- 1 nadia web 2137 Nov 14 14:02 style.css

/srv/mboamarket/site/images:
total 4
-rw-r--r-- 1 nadia web 270 Nov 14 14:02 logo.svg

```
>[!NOTE]
>On note bien que les fichiers du sites sont a `nadia` et au groupe `web`comme spécifié par le setgid `2` spécifié dans les droits du dossier site

---
## OPS-305 -La boutique en ligne, avec ses journaux

**Contexte.** La boutique doit remplacer la page « Welcome to nginx », avec ses propres journaux : ses visites et ses erreurs à part, pour la retrouver facilement.
### 1.Mise en place du bloc serveur par défaut pour servir l'application Mboamarket
```ini
   server {
       listen 80 default_server;
       server_name _;

       root /srv/mboamarket/site;
       index index.html;

       access_log /var/log/nginx/mboamarket.access.log;
       error_log  /var/log/nginx/mboamarket.error.log;

       location / {
           try_files $uri $uri/ =404;
       }
   }

```
### 2. On crée un racoursi(lien symbolique) du site dans le dossier sites-enabled de nginx
```bash
sudo ln -s /etc/nginx/sites-available/mboamarket /etc/nginx/sites-enabled/
```
### 3. on test la configuration
```bash
  sudo nginx -t
```
```test
2026/11/14 14:41:58 [emerg] 106726#106726: a duplicate default server for 0.0.0.0:80 in /etc/nginx/sites-enabled/mboamarket:2
nginx: configuration file /etc/nginx/nginx.conf test failed

```
>[!NOTE]
> Deux sites veulent être le serveur par défaut
### 4. On retire le lien du site **default** de nginx et on reteste

```bash
	sudo rm /etc/nginx/sites-enabled/default
        sudo nginx -t
```
```text
nginx: the configuration file /etc/nginx/nginx.conf syntax is ok
nginx: configuration file /etc/nginx/nginx.conf test is successful
```
### 5. on recharge Nginx
```bash
  sudo systemctl reload nginx
```
### 6. On vérifie avec un curl

```bash
curl -s http://localhost/ | grep "<title>"
```
```text
 <title>Mboa Market — la boutique</title>

```
---

## OPS-306 - Le compte de l'API du stock
**Contexte.** L'équipe a écrit une petite API qui donne le stock de chaque produit : `/srv/mboamarket/stock/stock.py`, sur le port 8090. On va en faire un service. D'abord, son compte : un service, un compte. Si l'API est piratée, l'attaquant n'a que les droits de ce compte

### 1. on crée un compte system stock

```bash
sudo useradd --system --no-create-home --shell usr/sbin/nologin stock
getent passwd stock
```
```text
stock:x:997:987::/home/stock:/usr/sbin/nologin
```
>[!TIP]
> le system met `/home/stock/` par défaut. `ls -la /home/` montre que le dossier stock n'existe pas dans la réalité.
> `nologin` pour pas de session interactive idéal pour les services.

---

## OPS-307 - L'API du stock devient un service

**Contexte.** Jusqu'ici, quelqu'un lançait l'API à la main dans son terminal : elle s'arrêtait à sa déconnexion, et personne ne la relançait quand elle plantait.

### 1. on crée le service de l'API
```bash
   sudo nano /etc/systemd/system/mboa-stock.service
```

```ini
[Unit]
Description= API du stock de Mboa Market
After=network.target

[Service]
User=stock
Group=stock
ExecStart=/usr/bin/python3 /srv/mboamarket/stock/stock.py
Restart=on-failure

[Install]
WantedBy=multi-user.target
```
### 2. on recharge systemd et on relance le service mboa-stock
```bash
 sudo systemctl daemon-reload 
 sudo systemctl enable --now mboa-stock
 systemctl status mboa-stock
```
```text
● mboa-stock.service - API du stock de Mboa Market
     Loaded: loaded (/etc/systemd/system/mboa-stock.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-11-14 15:39:30 CET; 4s ago
   Main PID: 107893 (python3)
      Tasks: 1 (limit: 9483)
     Memory: 10.3M (peak: 10.5M)
        CPU: 85ms
     CGroup: /system.slice/mboa-stock.service
             └─107893 /usr/bin/python3 /srv/mboamarket/stock/stock.py

```
---

## OPS-308 -La preuve: systemd relance l'API
**Contexte.** `Restart=on-failure`, ça se prouve : on tue l'API comme le ferait un plantage, et on regarde systemd la relancer.
### 1. on tue le service mboa-stock de Main PID 107893

```bash
  sudo kill -9 107893
```
### 2. on vérifie si l'API s'est relancée

```bash
 sudo journalctl -u mboa-stock -n 10
```
```text
Nov 14 15:50:35 said systemd[1]: mboa-stock.service: Main process exited, code=killed, sta>
Nov 14 15:50:35 said systemd[1]: mboa-stock.service: Failed with result 'signal'.
Nov 14 15:50:36 said systemd[1]: mboa-stock.service: Scheduled restart job, restart counte>
Nov 14 15:50:36 said systemd[1]: Started mboa-stock.service - API du stock de Mboa Market.
Nov 14 15:50:36 said python3[108128]: API du stock Mboa Market (v1) : à l'écoute sur 127.0>

```
---

## OPS-309 -Nginx devant l'API
**Contexte.** L'API n'écoute que sur la machine (`127.0.0.1`), et c'est voulu : les visiteurs passent par Nginx, qui reçoit `/api/stock` et le transmet à l'API.

### 1. on connecte l'API au site mboa-market via Nginx (Reverse Proxy)
```bash
  sudo nano /etc/nginx/sites-available/mboamarket
```
```ini
	location /api/ {
           proxy_pass http://127.0.0.1:8090/;
       }

```
### 2. on recharge Nginx et fait un curl

```bash
sudo nginx -t && sudo systemctl reload nginx
curl http://localhost/api/stock
```
```text
{"version": "v1", "produits": [{"id": "poivre-penja", "nom": "Poivre de Penja", "stock": 42}, {"id": "cafe-ouest", "nom": "Café de l'Ouest", "stock": 18}, {"id": "miel-oku", "nom": "Miel blanc d'Oku", "stock": 7}, {"id": "sirop-erable", "nom": "Sirop d'érable", "stock": 0}, {"id": "pagne-wax", "nom": "Pagne wax", "stock": 25}]}

```

---
## OPS-310 — Incident : les promotions en 403
**Contexte.**Après une mise à jour de koffi du site web et de l'APi,  Le service client appelle : la page des promotions affiche « 403 Forbidden ».

### 1.On fait un curl sur la page nouvellement ajoutée

```bash
  curl -I curl -I http://localhost/promo.html
```
```text
HTTP/1.1 403 Forbidden
Server: nginx/1.24.0 (Ubuntu)
Date: Sat, 14 Nov 2026 15:33:39 GMT
Content-Type: text/html
Content-Length: 162
Connection: keep-alive
```
>[!NOTE]
> erreur `403 Forbidden` il doit avoir un problème de droit sur le fichier

### 2.On cherche plus de détails dans les journaux du site

```bash
 sudo tail -n 5 /var/log/nginx/mboamarket.error.log 
```
```text
026/11/14 16:30:07 [error] 108299#108299: *58 connect() failed (111: Connection refused) while connecting to upstream, client: 127.0.0.1, server: _, request: "GET /api/stock HTTP/1.1", upstream: "http://127.0.0.1:8090/stock", host: "127.0.0.1"
2026/11/14 16:30:08 [error] 108297#108297: *60 open() "/srv/mboamarket/site/promo.html" failed (13: Permission denied), client: 127.0.0.1, server: _, request: "GET /promo.html HTTP/1.1", host: "127.0.0.1"
2026/11/14 16:30:08 [error] 108297#108297: *61 connect() failed (111: Connection refused) while connecting to upstream, client: 127.0.0.1, server: _, request: "GET /api/stock HTTP/1.1", upstream: "http://127.0.0.1:8090/stock", host: "127.0.0.1"
2026/11/14 16:30:08 [error] 108297#108297: *63 open() "/srv/mboamarket/site/promo.html" failed (13: Permission denied), client: 127.0.0.1, server: _, request: "GET /promo.html HTTP/1.1", host: "127.0.0.1"
2026/11/14 16:33:39 [error] 108297#108297: *64 open() "/srv/mboamarket/site/promo.html" failed (13: Permission denied), client: 127.0.0.1, server: _, request: "HEAD /promo.html HTTP/1.1", host: "localhost"

```
### 3. on regarde les droits du fichier promo
```bash
 ls -l /srv/mboamarket/site/promo.html 
```
```text
  -rw------- 1 root web 1310 Nov 14 16:28 /srv/mboamarket/site/promo.html
```
> le fichier n'est accessible qu'au root et à personne d'autre même pas Nginx
### 4.On ajoute les droits au group web et Nginx

```bash
  sudo chown 664 /srv/mboamarket/site/promo.html
  curl -I http://localhost/promo.html
```
```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Sat, 14 Nov 2026 15:37:44 GMT
Content-Type: text/html
Content-Length: 1310
Last-Modified: Sat, 14 Nov 2026 15:28:45 GMT
Connection: keep-alive
ETag: "6af87e2d-51e"
Accept-Ranges: bytes
```
## OPS-311 - Incident : le stock en 502
**Contexte.** Plus aucun stock ne s'affiche sur la boutique : `/api/stock` répond « 502 Ba>


### 1. On vérifie que le service API est ok
```bash
	systemctl status mboa-stock
```
```text
mboa-stock.service - API du stock de Mboa Market
     Loaded: loaded (/etc/systemd/system/mboa-stock.service; enabled; preset: enabled)
     Active: failed (Result: exit-code) since Sat 2026-11-14 16:28:47 CET; 38min ago
   Duration: 186ms
    Process: 108819 ExecStart=/usr/bin/python3 /srv/mboamarket/stock/stock.py (code=exited>
   Main PID: 108819 (code=exited, status=1/FAILURE)
        CPU: 183ms

```
> L'API ne fonctionne plus on doit regarder dans son journal
### 2. on regarde le journal de API

```bash
 sudo journalctl -u mboa-stock --no-pager -n 20

```
```text
Nov 14 16:28:46 said python3[108817]:     FICHIER = pathlib.Path(os.environ["STOCK_FICHIER"])
Nov 14 16:28:46 said python3[108817]:                            ~~~~~~~~~~^^^^^^^^^^^^^^^^^
Nov 14 16:28:46 said python3[108817]:   File "<frozen os>", line 685, in __getitem__
Nov 14 16:28:46 said python3[108817]: KeyError: 'STOCK_FICHIER'
Nov 14 16:28:46 said systemd[1]: mboa-stock.service: Main process exited, code=exited, status=1/FAILURE
Nov 14 16:28:46 said systemd[1]: mboa-stock.service: Failed with result 'exit-code'.
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Scheduled restart job, restart counter is at 4.
Nov 14 16:28:47 said systemd[1]: Started mboa-stock.service - API du stock de Mboa Market.
Nov 14 16:28:47 said python3[108819]: Traceback (most recent call last):
Nov 14 16:28:47 said python3[108819]:   File "/srv/mboamarket/stock/stock.py", line 21, in <module>
Nov 14 16:28:47 said python3[108819]:     FICHIER = pathlib.Path(os.environ["STOCK_FICHIER"])
Nov 14 16:28:47 said python3[108819]:                            ~~~~~~~~~~^^^^^^^^^^^^^^^^^
Nov 14 16:28:47 said python3[108819]:   File "<frozen os>", line 685, in __getitem__
Nov 14 16:28:47 said python3[108819]: KeyError: 'STOCK_FICHIER'
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Main process exited, code=exited, status=1/FAILURE
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Failed with result 'exit-code'.
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Scheduled restart job, restart counter is at 5.
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Start request repeated too quickly.
Nov 14 16:28:47 said systemd[1]: mboa-stock.service: Failed with result 'exit-code'.
Nov 14 16:28:47 said systemd[1]: Failed to start mboa-stock.service - API du stock de Mboa Market.

```
>[!NOTE]
> on constate une erreur d'éxécution keyError `STOCK_FICHIER`. D'après le code c'est une variable d'environnement.
> Il faut le spécifier dans le service mboamarket pour que systemd le prenne en compte

### 3. On ajoute la varaible d'environnement dans le service mboamarket

```ini
[Service]
...
Environment="STOCK_FICHIER=/srv/mboamarket/stock/stock.csv"
...
```

### 4. On rechage systemd et on redémarre le service mboa-stock
```bash
sudo systemctl daemon-reload
sudo systemctl restart mboa-stock
systemctl status mboa-stock
```

```text
● mboa-stock.service - API du stock de Mboa Market
     Loaded: loaded (/etc/systemd/system/mboa-stock.service; enabled; preset: enabled)
     Active: active (running) since Sat 2026-11-14 17:28:23 CET; 11s ago
   Main PID: 110141 (python3)
      Tasks: 1 (limit: 9483)
     Memory: 12.7M (peak: 12.8M)
        CPU: 152ms
     CGroup: /system.slice/mboa-stock.service
             └─110141 /usr/bin/python3 /srv/mboamarket/stock/stock.py
```

---
## OPS-312 - Défi : corriger l'outil de publication
**Contexte.** Si Koffi publie en décembre une version avec de nouvelles pages, elles casseront de la même façon : la cause est dans l'outil.
### 1. on ouvre le code de l'outil pour voir d'où vient le problème

```bash
sudo /usr/local/bin/mboa-publier
```
```ini
#!/usr/bin/env bash
# mboa-publier : publie la dernière version livrée de la boutique Mboa Market,
# son site et l'API du stock. L'outil de l'équipe, écrit par Koffi.
#
#   sudo mboa-publier                  publie la version de novembre
#   sudo mboa-publier --essai          la publie dans un dossier temporaire, pour voir les>
#   sudo mboa-publier --essai DOSSIER  la publie dans DOSSIER (vide), et l'y laisse
#
# Labo Runbookly : Mboa Market est une entreprise fictive.

set -euo pipefail
# Les fichiers de la livraison restent privés pendant la copie.
umask 077

VERSION=novembre
LIVRAISON=/srv/mboamarket/livraisons/$VERSION
SITE=/srv/mboamarket/site
STOCK=/srv/mboamarket/stock
JOURNAL=/var/log/mboamarket/publications.log


```
>[!NOTE]
> On constate que le umask est à 077 ce qui crée des fichiers en 600 et les dossiers en 700.
> C'est la cause des erruers de permissions rencontrées précédement lors du deploiement de novembre.

### 2. On modifie le mask de l'outil a 002
```ini
#!/usr/bin/env bash
# mboa-publier : publie la dernière version livrée de la boutique Mboa Market,
# son site et l'API du stock. L'outil de l'équipe, écrit par Koffi.
#
#   sudo mboa-publier                  publie la version de novembre
#   sudo mboa-publier --essai          la publie dans un dossier temporaire, pour voir les>
#   sudo mboa-publier --essai DOSSIER  la publie dans DOSSIER (vide), et l'y laisse
#
# Labo Runbookly : Mboa Market est une entreprise fictive.

set -euo pipefail
# Les fichiers de la livraison restent privés pendant la copie.
umask 002
``` 
### 3. On vérifie les droits des fichiers après création
```bash
sudo mboa-publier --essai 
```
```text
Essai : la version novembre publiée dans /tmp/tmp.wQ77JocDIN, avec ces droits :
  drwxrwxr-x  images
  -rw-rw-r--  images/logo.svg
  -rw-rw-r--  images/novembre.svg
  -rw-rw-r--  index.html
  -rw-rw-r--  promo.html
  -rw-rw-r--  stock.js
  -rw-rw-r--  style.css

```
---
## OPS-313 - Le projet sur GitHub
**Contexte.** Chez Mboa Market, la configuration d'un serveur vit dans un dépôt Git : si le serveur disparaît, on le reconstruit à partir du dépôt, et chacun voit ce qui a changé, quand, et pourquoi. Ton dépôt `mboa-market-serveur` est aussi ton projet de la semaine : sur ton GitHub, il montre ce que tu sais faire. Son README est la première page que lira un recruteur.



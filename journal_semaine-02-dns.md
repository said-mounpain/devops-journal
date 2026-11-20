# Journal de bord — Semaine 2 (15 au 18 novembre 2026)

> « Ce qui n'est pas écrit n'a pas été fait. »

Dans le cadre de ma formation pour devenir ingénieur DevOps, nous avons un projet fil rouge
qui nous permet de maîtriser de manière pratique les concepts étudiés.
Ce fichier documente les commandes de la **semaine 2**.
>Mboa Market ouvre sa boutique au public, sur TON sous-domaine :  https://<ton prénom>.formationdevops.xyz        (ta VM : hostname -s)

>Le TP a installé sur ta VM trois services, réglés « pour que ça marche » :  mboa-boutique    la boutique, en HTTP, port 8080          /etc/mboa/boutique
>.conf
>   mboa-admin       la console d'administration, port 9000   /etc/mboa/admin.conf
> mboa-metriques   les métriques, en UDP, port 8125         /etc/mboa/metriques.conf


## Récapitulatif

| Ticket | Sujet | Statut |
|---|---|---|
| OPS-501 |La résolution de ton sous-domaine, de la racine à l'IP | ✅ |
| OPS-502 | La console d'administration n'est plus sur Internet | ✅ |
| OPS-503 | nginx en reverse proxy devant la boutique | ✅ |
| OPS-504 | La boutique sait qui la demande, et par quel nom |✅ |
| OPS-505 | HTTPS avec un vrai certificat | ✅ |
| OPS-506 | Le HTTP renvoie vers le HTTPS |✅ |
| OPS-507 |Le port UDP des métriques n'est plus ouvert au monde |✅ |
| OPS-508 | Une requête suivie de bout en bout |✅ |

---

## OPS-501 — La résolution de ton sous-domaine, de la racine à l'IP

Explique, étape par étape, comment <prénom>.formationdevops.xyz devient une
adresse IP.

### 1. on suis  le processus de résolution DNS depuis les serveurs racine jusqu'au serveur DNS faisant autorité pour le domaine.
```bash
	dig +trace said.formationdevops.xyz
```

```text
; <<>> DiG 9.18.39-0ubuntu0.24.04.9-Ubuntu <<>> +trace said.formationdevops.xyz
;; global options: +cmd
.			50897	IN	NS	g.root-servers.net.
.			50897	IN	NS	c.root-servers.net.
.			50897	IN	NS	e.root-servers.net.
.			50897	IN	NS	h.root-servers.net.
.			50897	IN	NS	a.root-servers.net.
.			50897	IN	NS	l.root-servers.net.
.			50897	IN	NS	d.root-servers.net.
.			50897	IN	NS	k.root-servers.net.
.			50897	IN	NS	f.root-servers.net.
.			50897	IN	NS	b.root-servers.net.
.			50897	IN	NS	j.root-servers.net.
.			50897	IN	NS	m.root-servers.net.
.			50897	IN	NS	i.root-servers.net.
;; Received 239 bytes from 127.0.0.53#53(127.0.0.53) in 2 ms

;; communications error to 2001:500:1::53#53: timed out
;; communications error to 2001:500:1::53#53: timed out
xyz.			172800	IN	NS	x.nic.xyz.
xyz.			172800	IN	NS	y.nic.xyz.
xyz.			172800	IN	NS	z.nic.xyz.
xyz.			172800	IN	NS	generationxyz.nic.xyz.
xyz.			86400	IN	DS	3599 8 2 B9733869BC84C86BB59D102BA5DA6B27B2088552332A39DCD54BC4E8 D66B0499
xyz.			86400	IN	DS	18130 8 2 5AA5961266594CCEAC50949A99219FE004F130E1864A427143E9FF2E 641CAC6F
xyz.			86400	IN	RRSIG	DS 8 1 86400 20261201050000 20261118040000 8763 . NJFdkIgZ1GqUqBm4SOhrr1Q1zsGd1I0Vd87qbqN5vA6Q0lMt5i0jNI1Q SwMcQuRSGfVSya0+fsCPfu8heK+5APo1r0VJVPLcOBC+xWVgs4YasEF2 TG860N9NbihAGqlSCuGDpb6Tmzfm+8VuVIXE4XdwyUiOSP0BB/luvJLV QTO7igp0eB3rWXllxBKZbT11PP/NPoWwS77ecXo4Mz2VOhUAXbQCR9zV 8hddXCvum7dBduMo3mFOdmwe2zl2hhxeP+HVLv5iqzeYMEqi7cKu9bDd kgkCZ4jooHBCiG35tTUisrxZaRBve0e7jDA/1HXeexhcap14y+8SJwJh rCQhrA==
;; Received 692 bytes from 2001:500:1::53#53(h.root-servers.net) in 8 ms

formationdevops.xyz.	3600	IN	NS	dns1.registrar-servers.com.
formationdevops.xyz.	3600	IN	NS	dns2.registrar-servers.com.
2tjms8vm0h8n7j9e748d19gpnujm0emj.xyz. 900 IN NSEC3 1 1 0 - 2TJP5FRIV5KUA5J3R2QK9SCPC27BTDPH NS SOA RRSIG DNSKEY NSEC3PARAM
2tjms8vm0h8n7j9e748d19gpnujm0emj.xyz. 900 IN RRSIG NSEC3 8 2 900 20261213073223 20261113060223 65206 xyz. osyL8Q4a5Hs2EDDRZIkWcrkY4hsAiw8V8TUIvMIXscHaJ7zh3tPLy5ih f8uOnJ+garfcO0VahqNZBvxm0b5L9/jblqKtokR32uNEJi9gnE4+Puxr YCNM66PhwytmYFdR8M66ndN7GHWEjIJibnFEzdDnrS8r7yKm0iRoqxcP rKU=
nvarsfm9mli08nm8kgkb841ihi1gjlle.xyz. 900 IN NSEC3 1 1 0 - NVB6K465P6GIS9VCM0UO55UF1H8SRJR5 NS DS RRSIG
nvarsfm9mli08nm8kgkb841ihi1gjlle.xyz. 900 IN RRSIG NSEC3 8 2 900 20261213161712 20261113144712 65206 xyz. nqu9JM1xDciOcFmoypbaOXEf0xj1+wusdXOMrXjUASVPLpLFSoLjm8Go rfIDazztkUOpBBCmrfFeJweFp5KQU5DKRffT/OQHoQB0KEmJoXV+F0mD 98qfXDl1HCGVITyrKm6PBm8EJLXgQzPXhVx0IdCV5eDpXKaruordXflQ s4s=
;; Received 625 bytes from 185.24.64.42#53(y.nic.xyz) in 11 ms

said.formationdevops.xyz. 300	IN	A	13.140.200.16
formationdevops.xyz.	1800	IN	NS	dns1.registrar-servers.com.
formationdevops.xyz.	1800	IN	NS	dns2.registrar-servers.com.
;; Received 128 bytes from 2610:a1:1025::200#53(dns2.registrar-servers.com) in 4 ms
```
### 2. on interprète le résultat du dig

```text

Résolution DNS de said.formationdevops.xyz

1. SERVEUR RACINE : h.root-servers.net

Le résolveur DNS (dig) interroge le serveur racine h.root-servers.net pour savoir quels serveurs sont responsables de l'extension .xyz. Le serveur racine répond en indiquant les serveurs DNS de .xyz, notamment x.nic.xyz, y.nic.xyz et z.nic.xyz.

2. SERVEUR DE L'EXTENSION .xyz : y.nic.xyz

Dig interroge ensuite le serveur y.nic.xyz pour savoir quels serveurs sont responsables du domaine formationdevops.xyz. Le serveur répond en indiquant les serveurs DNS dns1.registrar-servers.com et dns2.registrar-servers.com.

3. SERVEUR DE formationdevops.xyz : dns2.registrar-servers.com

Dig interroge ensuite dns2.registrar-servers.com pour obtenir l'adresse IPv4 de said.formationdevops.xyz. Le serveur DNS répond avec l'enregistrement A contenant l'adresse 13.140.200.16.

4. ADRESSE IP OBTENUE : 13.140.200.16

Le nom said.formationdevops.xyz est ainsi résolu en adresse IPv4 13.140.200.16.

```
---
## OPS-502 -La console d'administration n'est plus sur Internet
Un client a ouvert http://<ton IP>:9000 depuis chez lui. Il ne devait pas.
### 0. on fait un curl pour voir si mboa-admin est accessible par un tier
```bash
curl -I http://13.140.200.16:9000/

```
```text
HTTP/1.0 200 OK
Server: mboa-admin 
Date: Wed, 18 Nov 2026 21:50:42 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 462

```
### 1. vérifier les ports associés aux services
```bash
	ss -tlnp
```
```text
State    Recv-Q   Send-Q     Local Address:Port     Peer Address:Port  Process  
LISTEN   0        5                0.0.0.0:9000          0.0.0.0:*              
LISTEN   0        4096       127.0.0.53%lo:53            0.0.0.0:*              
LISTEN   0        511              0.0.0.0:443           0.0.0.0:*   
...
```
>[!NOTE]
> On remarque que le service mboa-admin est disponible sur internet `0.0.0.0`

### 2. on change l'adresse du fichier de configuration du service mboa-admin

```bash
sudo nano /etc/mboa/admin.conf 
sudo systemctl restart mboa-admin
```

```ini
MBOA_ADRESSE=127.0.0.1
MBOA_PORT=9000

```
### 3. on vérifie s'il est toujours accessible par un tier

```bash
curl -I http://13.140.200.16:9000/
```
```text
curl: (7) Failed to connect to 13.140.200.16 port 9000 after 0 ms: Couldn't connect to server
```
---

## OPS-503 -nginx en reverse proxy devant la boutique

Terminé quand http://<prénom>.formationdevops.xyz/ affiche la boutique,
servie par nginx sur le port 80, et que mboa-boutique n'écoute plus que sur
127.0.0.1:8080 (seul nginx lui parle).

### 1. On modifie le fichier de configuration de mboa-boutique

```bash
	sudo /etc/mboa/boutique.conf 
```
```text
MBOA_ADRESSE=127.0.0.1
MBOA_PORT=8080

```
### 2. On fait pointer le domaine vers la boutique en modifiant le  site mboamarket dans Nginx 
```bash
	sudo nano  /etc/nginx/sites-available/mboamarket 
```

```ini
server {
    listen 80 default_server;
    server_name said.formationdevops.xyz;

    root /srv/mboamarket/site;
    index index.html;

    access_log /var/log/nginx/mboamarket.access.log;
    error_log  /var/log/nginx/mboamarket.error.log;

    # API → backend 8090
    location /api/ {
        proxy_pass http://127.0.0.1:8090/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

 # Tout le reste → frontend 8080
    location / {
        proxy_pass http://127.0.0.1:8080/;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

```
>[!NOTE]
>Il faut bien vérifier le port utilisé par l'API pour éviter les conflits de port 

### 3. On relance Nginx, mboa-boutique  et on fait un curl

```bash
sudo systemctl daemon-reload
sudo systemctl reload nginx
curl -I said.formationdevops.xyz
```
```text
HTTP/1.1 200 OK
Server: nginx/1.24.0 (Ubuntu)
Date: Fri, 20 Nov 2026 20:21:05 GMT
Content-Type: text/html; charset=utf-8
Content-Length: 425
Connection: keep-alive

```
---

## OPS-504 ★★  La boutique sait qui la demande, et par quel nom
**Contexte**Derrière nginx, la boutique croit que tous ses clients s'appellent 127.0.0.1.

### 1.On ajoute dans le fichier de configuration de la Nginx de la boutique les lignes de **proxy set_header**
```ini
  	proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
```
>[!NOTE]
> On reload le service Nginx et le tout est joué.


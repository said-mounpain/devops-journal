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

## Bilan de la semaine

- Comptes individuels créés pour Amina et Koffi, ajoutés au groupe `equipe`.
- Dossier partagé `/srv/mboamarket/partage` opérationnel avec `2770` + SGID.
- Critères d'acceptation vérifiés par des tests concrets (`amina` ✅ / `jules` ❌).
- Journal versionné sur GitHub, en Markdown, pour être lisible par l'équipe.

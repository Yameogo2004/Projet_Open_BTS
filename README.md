# 📡 Projet OpenBTS — Déploiement et expérimentation d'un réseau GSM privé avec SDR (USRP B210)

<p align="center">
  <img src="https://img.shields.io/badge/Technology-OpenBTS-blue?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Network-GSM%202G-green?style=for-the-badge" />
  <img src="https://img.shields.io/badge/SDR-USRP%20B210-red?style=for-the-badge" />
  <img src="https://img.shields.io/badge/Container-Docker-2496ED?style=for-the-badge&logo=docker&logoColor=white" />
  <img src="https://img.shields.io/badge/Project-Telecommunications-orange?style=for-the-badge" />
  <img src="https://img.shields.io/badge/University-ENSA%20Tétouan-purple?style=for-the-badge" />
</p>

<p align="center">
  <b>Rapport de projet technique — ENSA Tétouan</b><br/>
  <i>Département Télécommunications — Filière Génie des Systèmes des Télécommunications et Réseaux</i>
</p>

---

## 📖 Présentation du projet

Ce projet porte sur la **mise en place, la configuration et l'expérimentation d'un réseau GSM privé expérimental** basé sur la solution open source **OpenBTS** et une radio logicielle **Ettus USRP B210**.

L'ensemble de la plateforme est déployé dans un **conteneur Docker** exécuté au sein d'une **machine virtuelle VMware** sous **Ubuntu Desktop**. Le réseau intègre les services suivants :

- **OpenBTS** : station de base GSM (BTS)
- **Asterisk** : routage des appels vocaux via SIP
- **sipauthserve** : registre des abonnés (IMSI/TMSI)
- **smqueue** : serveur de messages SMS
- **UHD** : pilotes de la SDR

L'objectif est double : mettre en œuvre une infrastructure GSM complète et **analyser quantitativement les performances radio** (RXLEV, SNR, FER, TXPWR, Timing Advance) dans un environnement contrôlé, en confrontant les mesures aux modèles de propagation théoriques (FSPL, Okumura-Hata).

---

## 🎯 Objectifs

- Comprendre l'architecture et le fonctionnement d'un réseau GSM.
- Déployer et configurer une station de base **OpenBTS** avec une SDR **USRP B210**.
- Enregistrer des terminaux mobiles sur le réseau expérimental (IMSI/TMSI).
- Établir des **communications vocales** entre abonnés (postes 100 et 200).
- Tester le **service SMS** (smqueue).
- Réaliser une **campagne de mesures radio** (8 positions, 2 téléphones, 16 relevés TCH/F).
- Analyser les indicateurs : **RXLEV, SNR, FER, TXPWR, Timing Advance**.
- Étudier l'**affaiblissement de propagation (Path Loss)** et comparer aux modèles **FSPL** et **Okumura-Hata**.
- Identifier les **contraintes techniques** liées au déploiement d'une plateforme SDR/GSM.

---

## 🏗️ Architecture et technologies

### Schéma fonctionnel de la plateforme

```text
┌─────────────────────────────────────────────────────────────┐
│                    Machine virtuelle (VMware)               │
│                          Ubuntu Desktop                     │
│  ┌───────────────────────────────────────────────────────┐  │
│  │              Conteneur Docker (penthertz/openbts)     │  │
│  │                                                       │  │
│  │   ┌──────────┐   ┌──────────────┐   ┌─────────────┐   │  │
│  │   │ OpenBTS  │◄─►│ sipauthserve │   │  smqueue    │   │  │
│  │   │  (BTS)   │   │  (registre)  │   │   (SMS)     │   │  │
│  │   └────┬─────┘   └──────────────┘   └─────────────┘   │  │
│  │        │                                              │  │
│  │        │ SIP                                          │  │
│  │   ┌────▼─────┐                                        │  │
│  │   │ Asterisk │  (routage appels vocaux)               │  │
│  │   └──────────┘                                        │  │
│  │        ▲                                              │  │
│  │        │ UHD                                          │  │
│  └────────┼──────────────────────────────────────────────┘  │
│           │                                                 │
│           │ USB 3.0 (/dev/bus/usb)                         │
│  ┌────────▼────────┐                                        │
│  │   USRP B210     │  (SDR — émission/réception RF)         │
│  └────────┬────────┘                                        │
└───────────┼─────────────────────────────────────────────────┘
            │  Radio (GSM 2G)
            │
     ┌──────┴──────┐
     │  📱 Tel 100 │     📱 Tel 200
     └─────────────┘     └─────────────┘
```

### Composants logiciels

| Composant | Rôle |
|---|---|
| **Docker Engine** (`docker.io`) | Conteneurisation de l'environnement OpenBTS |
| **OpenBTS** (image `penthertz/openbts:latest`, ~1,1 Go) | Station de base GSM pré-compilée |
| **Asterisk** | Serveur de téléphonie IP — routage des appels vocaux (SIP) |
| **smqueue** | Serveur de SMS |
| **sipauthserve** | Registre des abonnés |
| **UHD** (USRP Hardware Driver) | Pilotes de la SDR, inclus dans l'image |
| **cloud-guest-utils, socat, nano** | Paquets utilitaires (growpart, communication réseau, édition) |

### Matériel et environnement

| Élément | Détail |
|---|---|
| Ordinateur hôte | Machine physique (hôte VMware) |
| Machine virtuelle | **VMware** — Ubuntu Desktop, disque étendu de 6 Go à **40 Go** |
| SDR | **Ettus USRP B210** |
| Terminaux | **2 téléphones mobiles** (postes 100 et 200) |
| Réseau | PLMN **MCC/MNC 001/01** (affiché `00101` ou `101`), nom court **Range** |

### Outils de diagnostic

`lsusb`, `uhd_find_devices`, `socat`, `sqlite3`, `nmcli.py`, `nano`, `tail`, `tcpdump`.

### Scripts et fichiers de configuration

- **Scripts Python créés** : `collecte_mesures.py`, `ecoute_api.py`, `parse_chans.py`
- **Fichiers modifiés** : `sip.conf`, `extensions.conf`, `nmcli.py`, `daemon.json`, `config.toml`

---

## 🚀 Guide de déploiement (résumé)

> ⚠️ **Avertissement** — Les expérimentations doivent être réalisées **exclusivement dans un environnement autorisé et maîtrisé**, avec les autorisations réglementaires nécessaires pour toute émission radio.

### Étape 1 — Préparation de la VM (VMware)

Le disque initial (6 Go) provoque `No space left on device` lors du pull Docker. Extension à **40 Go** via VMware, puis redimensionnement de la partition :

```bash
lsblk
sudo parted /dev/sda mklabel msdos
sudo parted /dev/sda mkpart primary ext4 0% 100%
sudo mkfs.ext4 /dev/sda1
sudo mkdir -p /mnt/data
sudo mount /dev/sda1 /mnt/data
```

> ⚠️ La recréation de table de partitions et `mkfs.ext4` **effacent toutes les données** du disque ciblé. Vérifier le périphérique avec `lsblk` et sauvegarder avant exécution.

### Étape 2 — Installation de Docker et redirection du stockage

```bash
sudo apt update && sudo apt install docker.io -y
sudo usermod -aG docker $USER
newgrp docker

sudo systemctl stop docker
sudo mkdir -p /mnt/data/docker
sudo rsync -aP /var/lib/docker/ /mnt/data/docker
echo '{"data-root": "/mnt/data/docker"}' | sudo tee /etc/docker/daemon.json

# Redirection containerd (indispensable)
sudo mkdir -p /mnt/data/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml > /dev/null
sudo sed -i "s|root = '/var/lib/containerd'|root = '/mnt/data/containerd'|g" /etc/containerd/config.toml
sudo systemctl restart containerd docker
```

### Étape 3 — Lancement du conteneur et connexion à la SDR

```bash
sudo docker pull penthertz/openbts
sudo docker run -it --privileged -v /dev/bus/usb:/dev/bus/usb --net=host penthertz/openbts /bin/bash

chown -R user:user /dev/bus/usb
chmod -R 777 /dev/bus/usb
uhd_find_devices
```

### Étape 4 — Configuration des services et des abonnés

```bash
cd /root/OpenBTS/NodeManager
nano nmcli.py   # remplacer socket.send(request) par socket.send_string(request)

sipauthserve &
smqueue &
cd /OpenBTS
./OpenBTS > /tmp/openbts.log 2>&1 &
tail -n 10 /tmp/openbts.log   # vérifier « system ready »

cd /root/OpenBTS/NodeManager
./nmcli.py sipauthserve subscribers create "Tel1" IMSI<IMSI_tel1> 100
./nmcli.py sipauthserve subscribers create "Tel2" IMSI<IMSI_tel2> 200
./nmcli.py sipauthserve subscribers read
```

### Étape 5 — Configuration d'Asterisk (appels vocaux)

```bash
apt install asterisk -y
nano /etc/asterisk/sip.conf
nano /etc/asterisk/extensions.conf

asterisk -C /etc/asterisk/asterisk.conf -vvvvv
asterisk -r
# puis : sip reload, dialplan reload, sip show peers, dialplan show public
```

- `sip.conf` : deux peers SIP (un par téléphone), avec IMSI, contexte `public`, numéros 100 et 200.
- `extensions.conf` : plan de numérotation dans `[public]` avec la macro `dialGSM`.

### Étape 6 — Configuration des SMS (smqueue)

```bash
sqlite3 /etc/OpenBTS/smqueue.db ".tables"
sqlite3 /etc/OpenBTS/smqueue.db "UPDATE CONFIG SET VALUE = '3' WHERE KEYSTRING = 'SC.Register.Digits.Min';"
```

### Étape 7 — Activation du réseau

Dans la console `OpenBTS>` (ou via `socat` sur le port 49300) :

```text
config Control.LUR.OpenRegistration .*
config Control.LUR.SendTMSIs 1
config GSM.Radio.RxGain 20
```

Après enregistrement, désactivation de l'enregistrement ouvert :

```text
config Control.LUR.OpenRegistration ""
```

---

## 📊 Résultats et mesures

### Validation fonctionnelle

| Test | Résultat |
|---|---|
| Enregistrement des terminaux | ✅ 2 mobiles enregistrés avec TMSI attribué |
| Appels vocaux 100 ↔ 200 | ✅ Établis — qualité audio variable selon la position |
| SMS 100 → 200 | ✅ Fonctionnel |
| SMS 200 → 100 | ❌ Routage inverse non résolu |
| Collecte des indicateurs radio | ✅ 16 relevés TCH/F (8 positions × 2 téléphones) |

### Campagne de mesures radio (16 novembre 2026)

**Paramètres analysés** : RXLEV (niveau reçu), SNR, FER (Frame Erasure Rate), TXPWR (puissance d'émission commandée), Timing Advance (TA).

| Indicateur | Observation |
|---|---|
| **FER téléphone A** | **16,88 %** en position 6 |
| **TXPWR téléphone A** | Commandé à **33 dBm** (max) dès la position 4 |
| **Téléphone B** | Reste stable sur toute la campagne |
| **Corrélation FER ↔ RXLEV descendant** | **ρ = −0,69** |
| **Corrélation FER ↔ signal montant** | **ρ = −0,91** |
| **Timing Advance** | Mobiles dans le **premier pas de résolution (≈ 553 m)** |
| **Distances** | Estimées par inversion du modèle FSPL à partir du RXLEV |

### Modèles de propagation

L'étude du **Path Loss** (calculé à partir du RXLEV) a permis de confronter les mesures aux prédictions :

- **FSPL** (Free Space Path Loss)
- **Okumura-Hata**
- **Ajustement log-distance** à partir des distances équivalentes

> Les résultats montrent que le canal radio réel s'écarte du FSPL en espace libre, ce qui est attendu en environnement indoor/urbain.

---

## ⚠️ Difficultés rencontrées et solutions

| Difficulté | Solution apportée |
|---|---|
| **Détection et stabilité de la SDR (USRP B210)** | Droits USB (`chmod 777 /dev/bus/usb`), `--privileged`, `uhd_find_devices` |
| **Conflits de contextes Asterisk** | Reconfiguration des contextes `sip.conf` / `extensions.conf` |
| **Base de données des mesures désactivée / API inutilisable** | Contournement par scripts Python dédiés (`collecte_mesures.py`, `parse_chans.py`) |
| **Communication inter-services (ZMQ)** | Vérification des sockets et des ports |
| **Espace disque et permissions** | Extension VM 6 → 40 Go, redirection Docker + containerd vers `/mnt/data` |
| **Incompatibilité Python 2/3 de `nmcli.py`** | Remplacement de `socket.send(request)` par `socket.send_string(request)` |

---

## 📁 Structure du dépôt

```text
Projet_Open_BTS/
│
├── config_openbts.xlsx
├── mesures_openbts (1).xlsx
├── mesures_openbts_distances.xlsx
├── mesures_sms (1).xlsx
├── Modeles_propagation_OpenBTS.xlsx
│
├── Projet_OpenBTS_rapport_corrige_final.pdf
│
└── video et images des tests/
    ├── Images expérimentales
    └── Vidéos de démonstration
```

| Fichier | Contenu |
|---|---|
| `config_openbts.xlsx` | Configuration OpenBTS relevée le 16/11/2026 |
| `mesures_openbts (1).xlsx` | Mesures radio pendant les appels |
| `mesures_openbts_distances.xlsx` | Distances estimées et comparaison aux modèles |
| `mesures_sms (1).xlsx` | Captures radio pendant les échanges SMS |
| `Modeles_propagation_OpenBTS.xlsx` | Analyse FSPL / Okumura-Hata / log-distance |
| `Projet_OpenBTS_rapport_corrige_final.pdf` | **Rapport complet** (54 pages + annexes) |

---

## 📈 Documentation

📄 **Rapport complet** : [`Projet_OpenBTS_rapport_corrige_final.pdf`](./Projet_OpenBTS_rapport_corrige_final.pdf)

Le rapport contient :

- Chapitre 1 — Introduction et objectifs
- Chapitre 2 — Architecture et environnement technique
- Chapitre 3 — Guide de déploiement pas à pas
- Chapitre 4 — Tests fonctionnels, campagne de mesures et analyse
- Chapitre 5 — Difficultés rencontrées et solutions
- Chapitre 6 — Conclusion et perspectives
- **Annexe A** — Commandes utilisées et procédures de référence (11 tableaux)
- **Annexe B** — Captures d'écran de preuve
- **Annexe C** — Notions fondamentales (GSM, OpenBTS, SDR)

🎥 **Vidéos et images des tests** : [`video et images des tests/`](./video%20et%20images%20des%20tests/)

---

## 🧪 Environnement expérimental

- **Type** : Réseau GSM privé expérimental (2G)
- **Bande** : GSM 900 / 1800 (selon configuration USRP B210)
- **PLMN** : MCC/MNC `001/01` — nom court `Range`
- **Abonnés** : 2 terminaux (postes 100 et 200)
- **Durée de la campagne** : 16 novembre 2026
- **Conditions** : Environnement maîtrisé, mesures en 8 positions

> ⚠️ **Cadre réglementaire** — Toute émission radio doit être réalisée dans un cadre autorisé. Ce projet est strictement **académique et expérimental**.

---

## 👨‍💻 Auteurs

**Projet réalisé par :**

- **YAMEOGO Ariel Barthelemy W.**
- **Omar Hassan Abdoul-fatah**
- **Nachda Nourouddine**
- **Souheib Hassan Ahmed**

**Encadré par :** Pr. **Mohamed Charif**

**Établissement :** Université Abdelmalek Essaâdi — École Nationale des Sciences Appliquées de Tétouan (ENSA Tétouan)  
**Département :** Télécommunications  
**Filière :** Génie des Systèmes des Télécommunications et Réseaux  
**Année universitaire :** 2026–2027  
**Lieu :** Tétouan — Maroc

---

## 🔗 Liens

- 🐙 GitHub : [@Yameogo2004](https://github.com/Yameogo2004)
- 🌐 Portfolio : [Mon portfolio](https://yameogo2004.github.io/PORTFOLIO/)

---

## 🙏 Remerciements

Nous tenons à remercier **Pr. Mohamed Charif** pour son encadrement, ses conseils techniques et sa disponibilité tout au long de ce projet, ainsi que l'**ENSA Tétouan** pour la mise à disposition des ressources matérielles et logicielles nécessaires à l'expérimentation.

---

## 📜 Licence

Projet **académique et expérimental**.

Les composants logiciels tiers (OpenBTS, Asterisk, smqueue, sipauthserve, UHD, Docker) restent soumis à **leurs licences respectives** (GPL, AGPL, etc.).

---

<p align="center">
  <i>Rapport de projet technique — ENSA Tétouan — 2026/2027</i><br/>
  <b>« Mise en place et configuration d'un réseau GSM privé avec OpenBTS et SDR (USRP B210) »</b>
</p>

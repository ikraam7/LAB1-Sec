# LAB 1 — Mise en place de l’environnement Mobexler

## 1. Informations générales

- **Intitulé du TP** : Mise en place du lab Mobexler + snapshot clean
- **Objectif principal** : préparer un environnement de travail propre, stable et reproductible pour les prochains TP de sécurité mobile.
- **Système hôte** : Windows
- **Solution de virtualisation** : VMware
- **Machine virtuelle utilisée** : Mobexler
- **Cible Android utilisée** : Émulateur Android Studio
- **Outil de communication Android** : ADB

---

## 2. Objectifs pédagogiques

L’objectif de ce lab est de préparer un environnement permettant de :

- démarrer Mobexler sans erreur ;
- accéder à Internet depuis Mobexler via une carte réseau en mode NAT ;
- disposer d’un réseau privé de laboratoire via une carte Host-Only ;
- vérifier les interfaces réseau et la route par défaut ;
- tester la connectivité Internet et DNS ;
- vérifier la présence des outils essentiels comme ADB, Java et Python ;
- créer un snapshot propre de référence ;
- préparer une cible Android de test ;
- vérifier la communication avec la cible Android via ADB ;
- préparer une configuration proxy pour les futurs TP d’analyse réseau.

---

## 3. Glossaire

| Terme | Définition |
|---|---|
| VM | Machine virtuelle exécutée dans un logiciel comme VMware ou VirtualBox. |
| OVA/OVF | Format permettant d’importer ou d’exporter une machine virtuelle prête à l’emploi. |
| Snapshot | Sauvegarde de l’état d’une machine virtuelle à un instant précis. |
| NAT | Mode réseau qui permet à la VM d’accéder à Internet via la machine hôte. |
| Host-Only | Mode réseau privé entre la machine hôte et la machine virtuelle. |
| ADB | Android Debug Bridge, outil utilisé pour communiquer avec un appareil Android ou un émulateur. |
| Proxy | Intermédiaire réseau utilisé pour rediriger et observer le trafic d’une cible Android. |

---

## 4. Arborescence du dépôt

L’organisation conseillée du dépôt GitHub est la suivante :

```text
LAB1-Mobexler/
├── README.md
└── captures/
    ├── 01_download_mobexler_hash.jpg
    ├── 02_vm_network_adapters.jpg
    ├── 03_mobexler_login_desktop.jpg
    ├── 04_ip_a_interfaces.jpg
    ├── 05_ip_route.jpg
    ├── 06_ping_internet.jpg
    ├── 07_tools_versions.jpg
    ├── 08_snapshot_clean_baseline.jpg
    ├── 09_android_studio_emulator_running.jpg
    ├── 10_adb_devices_windows.jpg
    ├── 11_android_device_info.jpg
    └── 13_proxy_settings_android.jpg
```

## 5. Prérequis

Avant de commencer le TP, les éléments suivants étaient nécessaires :

- virtualisation activée dans le BIOS ;
- VMware installé sur Windows ;
- fichier `Mobexler.ova` téléchargé ;
- Android Studio installé ;
- Android SDK Platform Tools installé ;
- un émulateur Android disponible ;
- au moins 4 Go de RAM disponible pour la VM ;
- espace disque suffisant pour importer la VM.

---

## 6. Architecture du lab

L’environnement mis en place repose sur la structure suivante :

```text
Machine hôte Windows
│
├── VMware
│   └── VM Mobexler
│       ├── ens33 : NAT
│       └── ens34 : Host-Only
│
└── Android Studio
    └── Émulateur Android
```

La carte NAT permet à Mobexler d’accéder à Internet.  
La carte Host-Only sert de réseau privé de laboratoire entre la VM et l’environnement de test.

---

## 7. Téléchargement de Mobexler

La première étape a consisté à télécharger l’image OVA de Mobexler à partir du lien fourni dans l’énoncé du TP.

Après le téléchargement, le fichier `Mobexler.ova` a été placé dans le dossier `Downloads` de l’utilisateur Windows.

Pour vérifier que le fichier téléchargé était bien présent et exploitable, un hash SHA256 a été calculé avec PowerShell.

Commande utilisée :

```powershell
Get-FileHash .\Mobexler.ova -Algorithm SHA256
```

Le résultat montre que le fichier `Mobexler.ova` est présent et que son empreinte SHA256 a bien été générée.
<p align="center">
<img width="1442" height="128" alt="01_download_mobexler_hash" src="https://github.com/user-attachments/assets/8244df10-dce0-412d-b3ff-9f4b3c34bf7b" />
</p>

---

## 8. Importation de la VM dans VMware

Après le téléchargement, la machine virtuelle Mobexler a été importée dans VMware à partir du fichier OVA.

La configuration réseau de la VM a ensuite été vérifiée.

Deux cartes réseau ont été configurées :

| Carte réseau | Mode | Rôle |
|---|---|---|
| Network Adapter | NAT | Accès Internet pour Mobexler |
| Network Adapter 2 | Host-Only | Réseau privé de laboratoire |

Cette configuration est importante, car elle sépare l’accès Internet du réseau de test.
<p align="center">
<img width="275" height="53" alt="02_vm_network_adapters" src="https://github.com/user-attachments/assets/07658fbd-8f74-4c5d-82c6-71c2bbfcb0e8" />
</p>

---

## 9. Premier démarrage de Mobexler

La VM Mobexler a ensuite été démarrée dans VMware.

L’écran de connexion s’est affiché correctement, ce qui confirme que l’importation de la VM s’est déroulée avec succès.

La session utilisée est la session Mobexler.
<p align="center">
<img width="797" height="552" alt="03_mobexler_login_desktop" src="https://github.com/user-attachments/assets/fc778f1a-9508-49bd-857e-d6578c5728f7" />
</p>

---

## 10. Vérification des interfaces réseau

Après la connexion à Mobexler, un terminal a été ouvert afin de vérifier les interfaces réseau.

Commande utilisée :

```bash
ip a
```

Le résultat a montré plusieurs interfaces :

| Interface | Adresse IP | Rôle |
|---|---|---|
| `lo` | `127.0.0.1` | Interface locale |
| `ens33` | `192.168.206.129/24` | Interface NAT |
| `ens34` | `192.168.183.101/24` | Interface Host-Only |
| `docker0` | `172.17.0.1/16` | Interface Docker interne |

L’interface `ens33` correspond à la carte NAT.  
L’interface `ens34` correspond à la carte Host-Only.

### Remarque

Au départ, l’interface Host-Only ne possédait pas une adresse correcte.  
Elle a ensuite été corrigée afin d’utiliser l’adresse suivante :

```text
192.168.183.101/24
```

Cette adresse sera utilisée plus tard comme adresse proxy côté Android.
<p align="center">
<img width="722" height="402" alt="04_ip_a_interfaces" src="https://github.com/user-attachments/assets/6d747e93-ec29-41f5-b72a-8043bd23912e" />
</p>

---

## 11. Vérification de la route par défaut

Pour vérifier la route réseau utilisée par la VM, la commande suivante a été exécutée :

```bash
ip route
```

Le résultat observé contient la route par défaut suivante :

```text
default via 192.168.206.2 dev ens33 proto dhcp metric 100
```

Cela signifie que Mobexler utilise l’interface `ens33` pour sortir vers Internet.

Le réseau Host-Only est également présent :

```text
192.168.183.0/24 dev ens34 proto kernel scope link src 192.168.183.101
```

Cette configuration confirme que les deux réseaux sont bien séparés :

- `ens33` pour Internet ;
- `ens34` pour le réseau privé de laboratoire.
<p align="center">
<img width="711" height="135" alt="05_ip_route" src="https://github.com/user-attachments/assets/9874131b-b5c6-49ef-9c9b-d0ee02470428" />
</p>

---

## 12. Test de connectivité Internet

Pour vérifier que Mobexler dispose bien d’un accès Internet, deux tests de ping ont été réalisés.

Premier test vers une adresse IP publique :

```bash
ping -c 2 8.8.8.8
```

Deuxième test vers un nom de domaine :

```bash
ping -c 2 google.com
```

Les deux tests ont réussi avec `0% packet loss`.

Le ping vers `8.8.8.8` confirme que la connectivité Internet fonctionne.  
Le ping vers `google.com` confirme que la résolution DNS fonctionne également.
<p align="center">
<img width="727" height="355" alt="06_ping_internet" src="https://github.com/user-attachments/assets/41cb2bd0-ed2a-4a67-b73a-6d7d3d9d1117" />
</p>

---

## 13. Vérification des outils installés

Les outils principaux nécessaires pour les prochains TP ont ensuite été vérifiés.

Commandes utilisées :

```bash
adb version
java -version
python3 --version
```

Résultats observés :

| Outil | Version observée |
|---|---|
| ADB | Android Debug Bridge version 1.0.39 |
| Java | OpenJDK 11.0.23 |
| Python | Python 3.7.3 |

La présence de ces outils confirme que Mobexler est prêt pour les prochaines manipulations liées à Android.
<p align="center">
<img width="718" height="256" alt="07_tools_versions" src="https://github.com/user-attachments/assets/a8217063-04e6-4e0a-a894-d95ec3462694" />
</p>

---

## 14. Création du snapshot propre

Après validation du démarrage, du réseau et des outils, un snapshot propre a été créé dans VMware.

Nom du snapshot :

```text
CLEAN_BASELINE_LAB1
```

Description utilisée :

```text
Import OK, NAT+HostOnly OK, boot OK, Internet OK, prêt ADB
```

Ce snapshot représente l’état propre de référence du lab.

Il permettra de revenir rapidement à une configuration saine si un futur TP modifie la VM, le réseau, les certificats, les proxys ou les outils installés.
<p align="center">
<img width="415" height="277" alt="08_snapshot_clean_baseline" src="https://github.com/user-attachments/assets/be1119c2-5f21-499f-b373-b652b4988d9d" />
</p>

---

## 15. Préparation de la cible Android

Pour ce TP, la cible Android utilisée est un émulateur lancé depuis Android Studio sur la machine hôte Windows.

L’émulateur utilisé est :

| Élément | Valeur |
|---|---|
| Device | Pixel 2 XL |
| Version Android | Android 11.0 |
| API | 30 |
| Architecture | x86_64 |

L’émulateur a été lancé avec succès depuis Android Studio.
<p align="center">
<img width="381" height="60" alt="09_android_studio_emulator_running" src="https://github.com/user-attachments/assets/1bc2e152-c82c-4713-bb99-e4e32ca1fb17" />
</p>

---

## 16. Vérification ADB depuis Windows

Comme l’émulateur Android Studio s’exécute sur la machine hôte Windows, la vérification ADB a été faite depuis PowerShell.

Commande utilisée :

```powershell
& "$env:LOCALAPPDATA\Android\Sdk\platform-tools\adb.exe" devices
```

Résultat obtenu :

```text
List of devices attached
emulator-5554    device
```

Le statut `device` confirme que l’émulateur Android est correctement reconnu par ADB.
<p align="center">
<img width="1026" height="76" alt="10_adb_devices_windows" src="https://github.com/user-attachments/assets/6c8f8a1c-ee25-4756-8544-66822eb1c77a" />
</p>

---

## 17. Collecte des informations Android

Pour documenter la cible Android utilisée, plusieurs commandes ADB ont été exécutées.

Commandes utilisées :

```powershell
.\adb.exe shell getprop ro.product.model
.\adb.exe shell getprop ro.build.version.release
.\adb.exe shell getprop ro.product.cpu.abi
.\adb.exe shell id
```

Résultats observés :

| Commande | Résultat |
|---|---|
| `ro.product.model` | `sdk_gphone_x86_64` |
| `ro.build.version.release` | `11` |
| `ro.product.cpu.abi` | `x86_64` |
| `id` | `uid=2000(shell)` |

Ces informations confirment que la cible Android est active, accessible et prête pour les prochains tests.
<p align="center">
<img width="1420" height="215" alt="11_android_device_info" src="https://github.com/user-attachments/assets/8d300adb-6e62-4f8c-a065-db30ae7e7410" />
</p>

---

## 18. Configuration du proxy Android

Afin de préparer les prochains TP d’analyse du trafic réseau, le proxy Wi-Fi de l’émulateur Android a été configuré manuellement.

Paramètres utilisés :

| Paramètre | Valeur |
|---|---|
| Proxy | Manual |
| Proxy hostname | `192.168.183.101` |
| Proxy port | `8080` |

L’adresse `192.168.183.101` correspond à l’adresse Host-Only de Mobexler sur l’interface `ens34`.

Le port `8080` est utilisé pour les outils d’interception réseau comme Burp Suite ou MobSF Proxy.

### Remarque

Le champ `Bypass proxy for` doit idéalement être laissé vide afin que tout le trafic HTTP passe par le proxy configuré.
<p align="center">
<img width="275" height="505" alt="13_proxy_settings_android" src="https://github.com/user-attachments/assets/a9f3aba2-4eaf-4189-b421-b84d57b184a6" />
</p>

---


## 19. Résultats obtenus

À la fin du TP, les résultats suivants ont été obtenus :

| Élément vérifié | Résultat |
|---|---|
| Import de Mobexler | Réussi |
| Démarrage de Mobexler | Réussi |
| Carte NAT | Configurée |
| Carte Host-Only | Configurée |
| IP NAT | `192.168.206.129/24` |
| IP Host-Only | `192.168.183.101/24` |
| Route par défaut | Présente via `ens33` |
| Ping vers `8.8.8.8` | Réussi |
| Ping vers `google.com` | Réussi |
| ADB dans Mobexler | Disponible |
| Java dans Mobexler | Disponible |
| Python dans Mobexler | Disponible |
| Snapshot propre | Créé |
| Émulateur Android | Démarré |
| ADB Windows | Fonctionnel |
| Device Android | `emulator-5554 device` |
| Proxy Android | Configuré vers `192.168.183.101:8080` |

---


## 20. Interprétation

La configuration finale montre que l’environnement est prêt pour les prochains TP de sécurité mobile.

L’interface NAT `ens33` permet à Mobexler d’accéder à Internet.  
L’interface Host-Only `ens34` crée un réseau privé utilisable pour les tests de laboratoire.

La présence d’ADB, Java et Python confirme que Mobexler possède les outils nécessaires pour interagir avec des applications Android et exécuter des analyses.

La cible Android a été correctement préparée avec Android Studio, et ADB confirme que l’émulateur est accessible.

La configuration du proxy Android prépare l’environnement pour les prochains travaux liés à l’interception et à l’analyse du trafic HTTP/HTTPS.

---

## 21. Conclusion

Ce TP a permis de mettre en place un environnement Mobexler complet, fonctionnel et reproductible.

Les objectifs principaux ont été atteints :

- Mobexler a été importée et démarrée avec succès ;
- le réseau NAT fonctionne correctement ;
- le réseau Host-Only est configuré ;
- Internet et DNS fonctionnent dans la VM ;
- les outils ADB, Java et Python sont disponibles ;
- un snapshot propre a été créé ;
- une cible Android a été préparée avec Android Studio ;
- ADB reconnaît correctement l’émulateur ;
- le proxy Android est configuré vers l’adresse Host-Only de Mobexler.

L’environnement est donc prêt pour les prochains TP, notamment l’analyse réseau mobile, l’utilisation de proxys, l’analyse dynamique et les tests de sécurité Android.

---

## 22. Auteure

Réalisé par : Ikram Laabouki

Module : Sécurité des Applications Mobiles

Établissement : ENSA Marrakech

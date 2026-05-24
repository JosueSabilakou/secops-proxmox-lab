# Déploiement d'un Laboratoire SecOps d'Entreprise Virtuel sous Proxmox VE

## 📝 Description du Projet
Ce projet consiste en la conception, l'architecture et le déploiement complet d'une infrastructure d'entreprise virtualisée au sein d'un hyperviseur unique. Ce laboratoire simule un environnement de production d'entreprise couplé à un centre de surveillance opérationnelle de la sécurité (SOC). L'objectif est de valider des compétences transversales en administration systèmes, ingénierie réseau et sécurité défensive (Blue Team).

## 🛠️ Technologies & Compétences Validées
* **Hyperviseur :** Proxmox VE (PVE 8.x).
* **Réseau & Routage :** Pare-feu pfSense (Virtuel), Segmentation VLAN.
* **Systèmes :** Windows Server (Active Directory Domain Controller), Windows 10/11, Debian/Ubuntu Linux.
* **Sécurité Défensive :** Elastic Stack (SIEM), Fleet Server, Elastic Agents, Sysmon (Windows Event Logs).

---

## 📐 Architecture Réseau (Segmentation & Sécurité)
Pour garantir l'étanchéité de l'infrastructure et reproduire les exigences de sécurité d'une entreprise, le réseau a été segmenté en 4 zones distinctes via des interfaces réseau virtuelles (Linux Bridges / VLANs) gérées par un routeur/pare-feu **pfSense** :

| Zone (VLAN) | Rôle dans l'Infrastructure | Composants Hébergés |
| :--- | :--- | :--- |
| **WAN** | Accès Internet contrôlé | Interface externe du pfSense |
| **LAN DMZ** | Services accessibles publiquement | Serveur Web WordPress durci |
| **LAN Corp** | Réseau interne de l'entreprise | Contrôleur de domaine Windows Server, Postes clients |
| **LAN SecOps** | Zone d'administration et de surveillance | Serveur SIEM (Elastic Stack), Serveur de logs |

---

## 🚀 Étapes de Réalisation de l'Infrastructure

### 1. Configuration de l'Hyperviseur Proxmox VE
* Installation de Proxmox VE sur une infrastructure dédiée.
* Optimisation de l'allocation des ressources (CPU, RAM, Stockage ZFS/LVM) pour supporter l'exécution simultanée des différentes machines virtuelles (VM) et conteneurs (LXC).
* Configuration des ponts réseau (`vmbr0`, `vmbr1`) pour isoler le trafic interne du trafic internet brut.

### 2. Déploiement et Configuration de pfSense
* Installation de la VM pfSense faisant office de passerelle principale.
* Configuration des règles de filtrage (Firewall Rules) : isolation stricte du LAN SecOps, interdiction pour le réseau de production (LAN Corp) de communiquer directement avec la DMZ, et blocage des flux inter-VLAN non autorisés.
* Activation du serveur DHCP et configuration du DNS pour l'ensemble du laboratoire.

### 3. Administration Système & Services d'Entreprise
* **Windows Server :** Déploiement d'un contrôleur de domaine, configuration d'Active Directory (AD), création d'Unités Organisationnelles (OU), d'utilisateurs et de Groupes de Sécurité.
* **Gestion des Politiques (GPO) :** Implémentation d'une GPO de durcissement des mots de passe et d'une GPO d'audit avancé pour forcer la journalisation des événements système critiques (création de processus, modifications de l'annuaire, connexions échouées).

### 4. Surveillance Cyber & Centralisation SIEM (Blue Team)
* **Déploiement d'Elastic Stack :** Installation d'un serveur Elastic (SIEM) dédié dans la zone SecOps pour centraliser la réception et l'analyse des événements de sécurité.
* **Durcissement de la collecte (Windows) :** Installation de **Sysmon** (System Monitor) sur les machines Windows avec une configuration optimisée (SwiftOnSecurity) pour capturer les comportements malveillants avancés.
* **Déploiement des Agents :** Installation et liaison des Elastic Agents sur les serveurs Linux et Windows via le Fleet Server pour remonter en temps réel :
  * Les journaux d'événements Windows (*Windows Event Logs*).
  * Les fichiers d'authentification Linux (`/var/log/auth.log`).
  * Les flux et connexions réseau établis.

---

## 📊 Scénarios de Validation Opérationnelle (Cas d'Usage)
Pour tester l'efficacité de ce laboratoire, plusieurs attaques simulées ont été menées depuis une machine externe (Kali/Parrot OS) :
1. **Détection de Scan de Port :** Identification immédiate d'un scan agressif Nmap via l'analyse du trafic réseau et génération d'une alerte sur le tableau de bord Elastic SIEM.
2. **Tentative de Force Brute SSH :** Simulation d'une attaque par dictionnaire sur un serveur Linux, capturée par l'Elastic Agent et visualisée dans le SIEM via le suivi des échecs d'authentification consécutifs.
3. **Persistance Active Directory :** Simulation d'une création de compte utilisateur non autorisée dans un groupe d'administration Windows, immédiatement levée par les logs d'audit Sysmon corrélés dans le SIEM.

---

## 📈 Évolutions Prévues
* Intégration d'un système de détection d'intrusion (IDS) de type Snort ou Suricata au niveau du pare-feu pfSense.
* Automatisation du blocage des adresses IP d'attaque via un script Python interagissant avec l'API du pfSense dès la levée d'une alerte critique par le SIEM.

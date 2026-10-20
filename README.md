<div align="center">
  <picture>
    <source media="(prefers-color-scheme: dark)" srcset="assets/banner-dark.svg">
    <img src="assets/banner-light.svg" alt="VoIP MX-ONE Lab" width="100%">
  </picture>
</div>

<div align="center">
  <img src="https://img.shields.io/badge/Mitel_MX--ONE-IPBX-1BA0D7?style=flat-square" />
  <img src="https://img.shields.io/badge/VMware_Workstation-Pro-607078?style=flat-square&logo=vmware&logoColor=white" />
  <img src="https://img.shields.io/badge/Protocole-SIP_UDP%2F5060-4A5D9A?style=flat-square" />
  <img src="https://img.shields.io/badge/Linux-SUSE_Enterprise-73BA25?style=flat-square&logo=suse&logoColor=white" />
  <img src="https://img.shields.io/badge/Statut-Terminé-4ADE80?style=flat-square" />
</div>

<br/>

---

## Présentation

Déploiement complet d'une infrastructure de téléphonie d'entreprise basée sur **Mitel MiVoice MX-ONE Service Node**, réalisé dans le cadre d'un mini-projet académique à l'UIPE (Université Internationale Privée d'Excellence), Bamako — année académique 2025-2026.

Le projet couvre l'intégralité du cycle de vie d'un système IPBX professionnel : virtualisation, configuration avancée et diagnostic de pannes.

> 📄 **Rapport technique complet :** [`Rapport_Technique_VoIP_MX-ONE_KONE_Emmanuel.pdf`](./Rapport_Technique_VoIP_MX-ONE_KONE_Emmanuel.pdf)
>
> 🎬 **Vidéo de démonstration (10 min) :** [Télécharger depuis GitHub Releases](../../releases/latest)

---

## Architecture déployée

```
┌─────────────────────────────────────────────────────┐
│              VMware Workstation Pro                 │
│                                                     │
│  ┌──────────────────────────────────────────────┐  │
│  │     SUSE Linux Enterprise Server            │  │
│  │                                              │  │
│  │   ┌──────────────────────────────────┐      │  │
│  │   │   Mitel MiVoice MX-ONE           │      │  │
│  │   │   Service Node Manager (SNM)     │      │  │
│  │   │   https://192.168.101.10         │      │  │
│  │   └──────────────────────────────────┘      │  │
│  │                                              │  │
│  │   eth0 : 192.168.101.10/24 (management)     │  │
│  │   eth1 : interface téléphonie               │  │
│  └──────────────────────────────────────────────┘  │
│                                                     │
│   VMnet0 (Bridged) ────── VMnet8 (NAT)             │
└─────────────────────────────────────────────────────┘
              │
              ▼
   Softphone MicroSIP / Zoiper
   Extensions 5001 — 5003
   SIP UDP/5060 | RTP 10000-20000
```

| Composant | Valeur / Version | Rôle |
|---|---|---|
| Hyperviseur | VMware Workstation Pro | Virtualisation |
| OS MX-ONE | SUSE Linux Enterprise Server | Système hôte |
| Application | Mitel MiVoice MX-ONE | IPBX / Service Node |
| Adresse IP SNM | 192.168.101.10 | Interface d'administration |
| Softphone | MicroSIP + Zoiper | Terminal SIP |
| Protocole signalisation | SIP (UDP/5060) | Signalisation VoIP |
| Plage RTP | 10000 – 20000 UDP | Transport média |

---

## Résumé des laboratoires

| Laboratoire | Objectif | Durée | Statut |
|---|---|---|---|
| **Lab 1** | Installation et virtualisation MX-ONE | 4h | ✅ Réussi |
| **Lab 2** | Configuration extensions SIP + SIP Trunk | 6h | ✅ Réussi |
| **Lab 3** | Diagnostic et résolution de pannes VoIP | 4h | ✅ Réussi |

---

## Laboratoire 1 — Installation de base

**Objectif :** Déployer l'appliance virtuelle MX-ONE sous VMware et établir la connectivité réseau.

**Étapes réalisées :**
- Installation de VMware Workstation Pro et configuration de deux réseaux virtuels : `VMnet0` (Bridged) et `VMnet8` (NAT)
- Import de l'appliance OVA MX-ONE (4 Go RAM, 2 vCPU, double interface réseau : eth0 management + eth1 téléphonie)
- Configuration réseau via le script d'initialisation MX-ONE — `eth0` assignée à `192.168.101.10/24`
- Validation de la connectivité : `ping 192.168.101.1` et `ping 8.8.8.8`
- Accès au **Service Node Manager (SNM)** via navigateur : `https://192.168.101.10`

**Critères de réussite :**

| Critère | Résultat |
|---|---|
| VM démarrée sans erreur | ✅ Atteint |
| Connectivité réseau établie (ping) | ✅ Atteint |
| SNM accessible via HTTPS | ✅ Atteint |

---

## Laboratoire 2 — Configuration avancée

**Objectif :** Créer les extensions SIP, configurer le softphone, valider les appels internes et mettre en place un SIP Trunk simulé.

### Extensions SIP créées

4 utilisateurs créés dans le **Provisioning Manager MX-ONE**, département `LAB_UIE` :

| User ID | Nom | Prénom | Extension | Département |
|---|---|---|---|---|
| Ass | Diarra | Assanatou | 5001 | LAB_UIE |
| Adja | Sidibe | Adja | 5002 | LAB_UIE |
| Dri | Diaou | Drissa | 5003 | LAB_UIE |
| bmg | Koné | Emmanuel | — | LAB_UIE |

### Configuration softphone (MicroSIP)

```
Serveur SIP (Domain)  : 192.168.101.10
Nom d'utilisateur     : 5001
Authentification ID   : 5001
Transport             : UDP
Port SIP              : 5060
Statut                : Enregistré / En ligne
```

### SIP Trunk simulé

- Plan de numérotation configuré : préfixe `90` → destination externe
- Route `SIP_Trunk_Sim` créée dans `Téléphonie > Lignes externes > Route`

```
Nom de la route        : SIP_Trunk_Sim
Type de signalisation  : SIP
Protocole sortant      : UDP
Port distant           : 5060
Remote Proxy IP        : 192.168.101.254
Type d'appels acceptés : Remote IP
Correspondance entrant : 192.168.101.254
Invite URI             : sip:+?@192.168.101.254
```

**Critères de réussite :**

| Critère | Résultat |
|---|---|
| Extensions créées et configurées | ✅ Atteint |
| Profils différents assignés | ✅ Atteint |
| Softphone enregistré et fonctionnel | ✅ Atteint |
| Appels internes réussis (5001↔5002↔5003) | ✅ Atteint |
| Route SIP configurée | ✅ Atteint |
| Plan de numérotation externe (préfixe 90) | ✅ Atteint |

---

## Laboratoire 3 — Dépannage

**Objectif :** Diagnostiquer et résoudre 3 scénarios de pannes courants en environnement VoIP.

### Scénario 1 — Extension non enregistrée

**Symptôme :** Le softphone affiche `Échec de l'enregistrement` — aucun appel possible.

**Méthodologie de diagnostic :**
```bash
ping 192.168.101.10                    # Test de connectivité réseau
tail -f /var/log/eri_sn/sip.log        # Lecture des logs SIP
netstat -tulnp | grep 5060             # Vérification du port pare-feu
```
**Cause et solution :** Paramètres d'authentification SIP incorrects + port UDP 5060 bloqué sur le pare-feu. Correction des identifiants et ouverture du port.

---

### Scénario 2 — Absence de tonalité

**Symptôme :** Le téléphone ne produit pas de tonalité après décrochage.

**Méthodologie de diagnostic :**
```bash
# Vérifier le statut du Media Server dans SNM : Services > Media Server
tail -f /var/log/eri_sn/media.log      # Logs du serveur média
systemctl restart eri-media            # Redémarrage du service
systemctl status eri-media             # Vérification du statut
```
**Cause et solution :** Service Media Server arrêté. Redémarrage via `systemctl`, ressources DSP confirmées disponibles.

---

### Scénario 3 — Appel sans audio (one-way ou no audio)

**Symptôme :** L'appel est établi (signalisation SIP fonctionnelle) mais aucun audio n'est transmis.

**Méthodologie de diagnostic :**
```bash
# Vérifier la plage RTP 10000-20000 dans la configuration MX-ONE
# Capture du trafic RTP avec Wireshark :
wireshark -i eth0 -f "udp portrange 10000-20000"
# Vérification des codecs négociés dans les logs SIP (G.711, G.729)
```
**Cause et solution :** Plage de ports RTP 10000-20000 UDP bloquée sur le pare-feu.
```bash
iptables -A INPUT -p udp --dport 10000:20000 -j ACCEPT
```
Capture Wireshark confirmant la transmission des flux RTP après correction.

**Critères de réussite :**

| Critère | Résultat |
|---|---|
| Identification correcte des causes | ✅ Atteint |
| Utilisation des outils de diagnostic | ✅ Atteint |
| Résolution effective des problèmes | ✅ Atteint |

---

## Compétences démontrées

- 🖥️ **Virtualisation** — Déploiement d'appliance télécom sur VMware (import OVA, configuration double NIC)
- 📞 **Administration IPBX** — Mitel MX-ONE Service Node Manager, Provisioning Manager
- 🔧 **Protocole SIP** — Création d'extensions, configuration SIP Trunk, plan de numérotation
- 🔍 **Dépannage VoIP** — Analyse de logs SIP, capture RTP, diagnostic pare-feu
- 🐧 **Systèmes Linux** — SUSE Enterprise Server, `systemctl`, `iptables`, `netstat`, `tail`
- 📡 **Analyse réseau** — Wireshark, capture de trames, analyse protocoles RTP/SIP

---

## Informations projet

| Champ | Valeur |
|---|---|
| Module | Introduction à la Télécommunication |
| Établissement | Université Internationale Privée d'Excellence (UIPE), Bamako 🇲🇱 |
| Encadrant | M. Amadou Bagayoko |
| Année académique | 2025-2026 |
| Auteur | Emmanuel Elisé Koné (`bmg`) |
| Date de rendu | Semaine 6 — 2026 |

---

<img src="assets/divider.svg" width="100%" alt="">

## Licence

Documentation et contenus sous [licence CC BY 4.0](./LICENSE) · © 2026 Emmanuel Elisé Koné. Tu peux les réutiliser et les adapter, à condition de me citer.

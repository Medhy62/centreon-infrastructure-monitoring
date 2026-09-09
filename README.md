# centreon-infrastructure-monitoring
Refonte et optimisation d’une plateforme Centreon : supervision systèmes/réseaux, structuration des hôtes et groupes, ajustement des seuils, réduction des faux positifs et création de dashboards.

# Centreon Infrastructure Monitoring

## Présentation

Ce projet présente la **refonte complète d'une plateforme de supervision Centreon** dans un environnement d'entreprise.

Une instance Centreon était déjà présente, mais sa configuration avait évolué au fil du temps et nécessitait une remise à plat : hôtes obsolètes, groupes peu structurés, services inutilisés, seuils d'alertes inadaptés et tableaux de bord devenus difficiles à exploiter.

L'objectif a donc été de **repartir sur une architecture de supervision propre, structurée et maintenable**, tout en conservant l'instance Centreon existante.

> Les noms d'équipements, adresses IP, domaines et informations permettant d'identifier l'infrastructure d'origine ont volontairement été supprimés ou anonymisés.

---

## Objectifs du projet

Les principaux objectifs étaient :

* Auditer la configuration Centreon existante
* Identifier les équipements réellement présents dans l'infrastructure
* Supprimer les anciens hôtes et services devenus inutiles
* Recréer une organisation cohérente des équipements
* Uniformiser les noms des hôtes et des services
* Superviser les principales métriques systèmes et réseaux
* Adapter les seuils `WARNING` et `CRITICAL`
* Réduire les faux positifs
* Structurer les groupes d'hôtes
* Construire des dashboards exploitables par l'équipe IT
* Faciliter l'identification des incidents
* Améliorer la lisibilité globale de la supervision
* Préparer la plateforme pour de futurs équipements

---

# Architecture supervisée

L'environnement supervisé comprend plusieurs catégories d'équipements.

```text
                    +----------------------+
                    |      CENTREON        |
                    | Monitoring Platform  |
                    +----------+-----------+
                               |
          +--------------------+--------------------+
          |                    |                    |
          v                    v                    v
   +-------------+      +-------------+      +-------------+
   |  Serveurs   |      |   Réseau    |      |  Services   |
   +-------------+      +-------------+      +-------------+
   | Linux       |      | Switches    |      | Applications|
   | Windows     |      | Firewalls   |      | Stockage    |
   | Hyperviseurs|      | Wi-Fi       |      | VoIP        |
   | VM          |      | Routeurs    |      | Services IT |
   +-------------+      +-------------+      +-------------+
```

---

# Technologies utilisées

| Technologie                | Utilisation                             |
| -------------------------- | --------------------------------------- |
| Centreon                   | Supervision centrale                    |
| Linux                      | Système hébergeant la plateforme        |
| SNMP                       | Supervision des équipements réseau      |
| Centreon Plugins           | Collecte des métriques                  |
| SSH                        | Administration et diagnostic            |
| ICMP                       | Vérification de disponibilité           |
| HTTP / HTTPS               | Monitoring de services web              |
| VMware / Hyper-V / Proxmox | Supervision de plateformes virtualisées |
| Fortinet                   | Supervision firewall                    |
| UniFi                      | Supervision réseau / Wi-Fi              |
| Cisco                      | Supervision switches                    |
| Dell                       | Supervision stockage                    |

---

# Méthodologie

La reconstruction de la supervision a été réalisée progressivement afin d'éviter de conserver les erreurs ou incohérences de l'ancienne configuration.

```text
Audit
  |
  v
Inventaire
  |
  v
Nettoyage
  |
  v
Classification
  |
  v
Reconfiguration
  |
  v
Réglage des seuils
  |
  v
Dashboards
  |
  v
Validation
```

---

# 1. Audit de l'existant

La première étape a consisté à analyser la configuration déjà présente dans Centreon.

Les principaux points contrôlés :

* Liste des hôtes
* Templates utilisés
* Services associés
* Plugins Centreon
* Groupes d'hôtes
* Groupes de services
* Notifications
* Seuils d'alertes
* Dashboards
* Hôtes en erreur permanente
* Services inconnus
* Équipements retirés de l'infrastructure

Certaines anciennes configurations généraient des alertes alors que les équipements concernés n'existaient plus.

---

# 2. Nettoyage de la plateforme

Une phase importante du projet a été le nettoyage des objets Centreon historiques.

Exemples d'éléments supprimés :

* anciens équipements réseau
* anciennes machines virtuelles
* services de test
* duplications de services
* anciens templates
* checks inutilisés
* équipements retirés de production

Avant toute suppression :

```text
Identification
      |
      v
Vérification réseau
      |
      v
Vérification inventaire
      |
      v
Validation
      |
      v
Suppression Centreon
```

Cette approche permet d'éviter de supprimer accidentellement un équipement toujours actif.

---

# 3. Organisation des hôtes

Une nomenclature cohérente a été mise en place.

Exemple :

```text
SRV-XXX
SW-XXX
FW-XXX
AP-XXX
VM-XXX
STORAGE-XXX
```

Cela permet d'identifier rapidement le type d'équipement directement depuis Centreon.

---

# 4. Organisation des groupes

Les équipements ont ensuite été classés dans différents groupes fonctionnels.

Exemple de structure :

```text
ENV_PRODUCTION

GROUPE_SERVEURS
├── Linux
├── Windows
├── Hyperviseurs
└── Machines virtuelles

GROUPE_RESEAU
├── Switches
├── Firewalls
├── Wi-Fi
├── Routeurs
└── WAN

GROUPE_INFRASTRUCTURE
├── Stockage
├── Sauvegarde
└── Applications

GROUPE_SERVICES
├── VoIP
├── Applications
└── Services métiers
```

L'utilisation des groupes simplifie ensuite :

* les dashboards
* les recherches
* les filtres
* les alertes
* les maintenances
* les opérations d'administration

---

# 5. Supervision système

Les serveurs disposent de plusieurs métriques essentielles.

## CPU

Surveillance de l'utilisation processeur.

```text
CPU Usage
```

Exemple de seuils :

```text
WARNING  : 80 %
CRITICAL : 90 %
```

Les valeurs peuvent être adaptées selon le rôle du serveur.

---

## Mémoire

Surveillance de la consommation RAM.

```text
Memory Usage
```

Exemple :

```text
WARNING  : 80 %
CRITICAL : 90 %
```

Une utilisation mémoire importante n'indique cependant pas systématiquement un problème, notamment sous Linux où la mémoire libre est souvent utilisée comme cache.

---

# Load Average

Le `Load Average` est particulièrement important pour la supervision Linux.

Il représente le nombre moyen de processus :

* en cours d'exécution
* en attente de CPU
* en attente de certaines ressources système

Les valeurs sont généralement mesurées sur :

```text
1 minute
5 minutes
15 minutes
```

Exemple :

```text
load1
load5
load15
```

---

## Interprétation du Load Average

Le Load Average doit toujours être comparé au nombre de CPU ou de cœurs disponibles.

Exemple :

```text
Machine : 4 CPU

Load = 1     -> faible charge
Load = 4     -> CPU pleinement utilisé
Load = 6     -> plusieurs tâches attendent
Load = 10    -> surcharge importante
```

Il est donc préférable de ne pas utiliser les mêmes seuils pour toutes les machines.

---

# 6. Adaptation des seuils

L'une des améliorations importantes a été de revoir les seuils historiques.

Des valeurs trop faibles peuvent générer de nombreux faux positifs.

Par exemple, pour un équipement ayant naturellement un Load Average élevé :

```text
WARNING  : 4
CRITICAL : 6
```

peut être totalement inadapté.

Le seuil doit dépendre de plusieurs facteurs :

```text
Nombre de CPU
     +
Charge habituelle
     +
Type d'équipement
     +
Historique des métriques
     =
Seuil Centreon
```

---

# Cas des équipements réseau

Certains équipements embarqués peuvent présenter un Load Average élevé même lorsque leur CPU reste très peu utilisé.

Exemple :

```text
CPU idle : ~100 %
Load Average : > 4
```

Cela peut notamment provenir :

* de processus internes
* de tâches kernel
* de processus en attente I/O
* du fonctionnement spécifique de l'OS embarqué

Il est donc nécessaire d'analyser l'équipement avant de modifier les seuils.

L'objectif n'est pas de supprimer une alerte mais de déterminer si celle-ci représente réellement une anomalie.

---

# 7. Supervision des équipements réseau

Les équipements réseau sont principalement supervisés via SNMP.

Principales métriques :

```text
Ping
CPU
Memory
Load
Interface Status
Interface Traffic
Errors
Packet Loss
Uptime
Temperature
Power Supply
Fans
```

Selon les équipements, d'autres métriques peuvent également être récupérées.

---

# Points d'accès Wi-Fi

Les points d'accès disposent notamment des checks :

```text
Ping
CPU
Memory
Load Average
Uptime
Interfaces
Traffic
```

Les seuils de Load Average ont été adaptés selon le comportement réel des équipements.

---

# Switches

Les switches sont supervisés pour identifier rapidement :

* surcharge CPU
* consommation mémoire
* interfaces DOWN
* erreurs réseau
* saturation
* perte de connectivité

Exemple :

```text
Switch
 |
 +-- Ping
 +-- CPU
 +-- Memory
 +-- Interface Status
 +-- Interface Traffic
 +-- Errors
 +-- Uptime
```

---

# Firewalls

Les firewalls peuvent être surveillés sur plusieurs critères :

```text
Availability
CPU
Memory
Sessions
Interfaces
VPN
Uptime
HA
Traffic
```

---

# Stockage

Les équipements de stockage disposent également de métriques spécifiques.

Exemples :

```text
Health
Capacity
Controllers
Disks
Nodes
Alerts
Power Supplies
Interfaces
```

L'objectif est notamment de faire remonter dans Centreon les alertes matérielles générées directement par la baie de stockage.

---

# 8. Gestion des alertes

Une attention particulière a été portée aux faux positifs.

Une alerte Centreon doit correspondre à une situation nécessitant potentiellement une action.

Le processus utilisé est :

```text
Alerte
   |
   v
Analyse
   |
   +----> Faux positif
   |          |
   |          v
   |     Ajustement seuil
   |
   +----> Incident réel
              |
              v
          Diagnostic
              |
              v
          Correction
```

---

# États Centreon

Les principaux états utilisés sont :

| État     | Signification                              |
| -------- | ------------------------------------------ |
| OK       | fonctionnement normal                      |
| WARNING  | anomalie nécessitant une surveillance      |
| CRITICAL | problème nécessitant une intervention      |
| UNKNOWN  | Centreon ne peut pas récupérer la métrique |

---

# 9. Dashboards

La refonte a également concerné les dashboards Centreon.

L'objectif était de construire une interface permettant de comprendre l'état du SI en quelques secondes.

Plusieurs vues peuvent être utilisées.

---

## Dashboard global

Vue synthétique de l'ensemble de l'infrastructure.

Widgets utilisés :

```text
Single Metric

Services en anomalie

Hosts en anomalie

Heatmap

Top N

Groupes de production
```

Exemple :

```text
+------------------------------------------------------+
|                 INFRASTRUCTURE STATUS                |
+-------------------------+----------------------------+
| Hosts OK                | Services OK                |
| Hosts Critical          | Services Critical          |
+-------------------------+----------------------------+
|                                                      |
|                    HEATMAP                           |
|                                                      |
+------------------------------------------------------+
|             SERVICES EN ANOMALIE                     |
+------------------------------------------------------+
|                TOP N ALERTES                         |
+------------------------------------------------------+
```

---

# Dashboard serveurs

Vue dédiée aux infrastructures serveurs.

```text
CPU
RAM
Load
Disk
Availability
Services critiques
```

---

# Dashboard réseau

Vue dédiée aux équipements réseau.

```text
Switches
Firewalls
Wi-Fi
Interfaces
CPU
Traffic
Availability
```

---

# Dashboard alertes

Une vue spécifique permet de se concentrer uniquement sur les incidents.

```text
Hosts DOWN
Services WARNING
Services CRITICAL
Services UNKNOWN
```

---

# 10. Filtrage par groupes

Les widgets ont été associés aux nouveaux groupes Centreon.

Cela permet par exemple d'afficher uniquement :

```text
ENV_PRODUCTION
```

ou :

```text
GROUPE_SERVEURS
```

ou :

```text
GROUPE_RESEAU
```

Cela évite que des équipements de test ou hors production apparaissent sur le dashboard principal.

---

# 11. Mode Kiosk

Centreon peut être affiché en permanence sur un écran de supervision.

Exemple avec Firefox :

```bash
firefox --kiosk "https://monitoring.example.local/"
```

L'objectif est d'afficher directement le dashboard principal sur un écran dédié.

Cela permet une supervision visuelle permanente du SI.

---

# 12. Export et vérification de la configuration

Des exports peuvent être réalisés afin de contrôler la configuration Centreon.

Par exemple :

```text
Hosts
Host Groups
Services
Templates
```

Le résultat peut ensuite être analysé au format :

```text
CSV
TSV
JSON
```

Cela permet notamment de vérifier :

* qu'un hôte appartient au bon groupe
* que tous les équipements sont supervisés
* qu'aucun ancien équipement n'existe encore
* que la nomenclature est cohérente

---

# 13. Résultat

Après la refonte :

* la configuration Centreon est plus lisible
* les hôtes sont correctement catégorisés
* les anciens équipements ont été supprimés
* les faux positifs ont été réduits
* les seuils correspondent davantage aux équipements
* les dashboards permettent une lecture rapide de l'état du SI
* les groupes facilitent les opérations d'administration
* l'ajout de nouveaux équipements est plus simple

---

# Architecture finale

```text
                         CENTREON
                            |
            +---------------+---------------+
            |               |               |
         SERVEURS          RESEAU        SERVICES
            |               |               |
     +------+------+   +----+----+    +-----+-----+
     |      |      |   |    |    |    |     |     |
   Linux Windows  VM  SW   FW   WiFi  Apps Storage VoIP
```

---

# Compétences mises en œuvre

Ce projet m'a permis de travailler sur différents sujets liés à l'administration système et réseau :

* Centreon
* Monitoring
* Linux
* SNMP
* Administration système
* Administration réseau
* Diagnostic d'incidents
* Supervision d'infrastructure
* Gestion des alertes
* Analyse de performance
* Capacity monitoring
* Dashboards
* Documentation technique
* Standardisation
* Exploitation / RUN

---

# Améliorations possibles

Plusieurs évolutions peuvent être envisagées.

## Automatisation

Automatiser la création des hôtes avec :

```text
Centreon API
+
Ansible
```

Exemple :

```text
Inventory Ansible
       |
       v
Centreon API
       |
       v
Création automatique
des hosts / services
```

---

## Infrastructure as Code

Une autre évolution serait de versionner une partie de la configuration Centreon.

```text
Git
 |
 +-- Hosts
 +-- Templates
 +-- Scripts
 +-- Plugins
 +-- Documentation
```

---

## Centralisation des logs

Centreon peut également être utilisé conjointement avec une plateforme de gestion des logs.

```text
Centreon
   |
Monitoring / Metrics
   |
Infrastructure
   |
Logs
   |
Graylog
```

Centreon est alors utilisé pour détecter une anomalie tandis que la plateforme de logs permet d'en rechercher la cause.

---

# Structure du repository

```text
centreon-infrastructure-monitoring/
│
├── README.md
│
├── docs/
│   ├── architecture.md
│   ├── host-groups.md
│   ├── monitoring-strategy.md
│   └── thresholds.md
│
├── examples/
│   ├── hosts.example.md
│   ├── services.example.md
│   └── thresholds.example.md
│
├── scripts/
│   └── README.md
│
└── images/
    └── README.md
```

---

# Confidentialité

Ce repository présente une version anonymisée d'un projet réalisé dans un environnement professionnel.

Aucune information confidentielle n'est publiée.

Les éléments suivants ont notamment été supprimés ou modifiés :

* adresses IP
* noms de domaine
* noms de serveurs internes
* noms d'utilisateurs
* noms de l'entreprise
* informations d'authentification
* données réseau sensibles
* règles de sécurité
* configurations propriétaires

Les exemples présents dans le repository sont uniquement destinés à illustrer la méthodologie utilisée.

---

# Auteur

Projet personnel de documentation basé sur une expérience réelle de **refonte et d'administration d'une infrastructure de supervision Centreon**.

Domaines associés :

`System Administration` · `Network Administration` · `Monitoring` · `Linux` · `Cybersecurity` · `Infrastructure` · `Automation`

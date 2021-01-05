---
title: Spécifications techniques et conformité de l’appareil Microsoft Azure Stack Edge Pro R | Microsoft Docs
description: Découvrez les spécifications techniques et la conformité de votre appareil Azure Stack Edge Pro R
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: 56d301762fde41f727b0b425d6c41a423f08103c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96465324"
---
# <a name="azure-stack-edge-pro-r-technical-specifications"></a>Spécifications techniques de l’appareil Azure Stack Edge Pro R

Les composants matériels de votre appareil Azure Stack Edge Pro R sont conformes aux spécifications techniques décrites dans cet article. Les spécifications techniques décrivent les blocs d’alimentation (PSU), la capacité de stockage, les boîtiers et les normes environnementales.


## <a name="compute-memory-specifications"></a>Spécifications de calcul et de mémoire

L’appareil Azure Stack Edge Pro R présente les spécifications suivantes en matière de calcul et de mémoire :

| Caractéristique       | Valeur                  |
|---------------------|------------------------|
| UC    | 2 processeurs 10 cœurs Intel Xeon Silver 4114 |
| Mémoire              | 256 Go de RAM (2666 MT/s)     |


## <a name="compute-acceleration-specifications"></a>Spécifications de l’accélération de calcul

Une unité centrale graphique (GPU) est incluse sur chaque appareil, qui active Kubernetes, l’apprentissage profond et des scénarios d’apprentissage automatique.

| Caractéristique           | Valeur                  |
|-------------------------|----------------------------|
| GPU   | Une GPU Nvidia T4 <br> Pour plus d’informations, consultez [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).| 

## <a name="power-supply-unit-specifications"></a>Spécifications des blocs d’alimentation

L’appareil Azure Stack Edge Pro R est équipé de deux blocs d’alimentation de 100-240 V avec des ventilateurs hautes performances. Les deux blocs d’alimentation fournissent une alimentation redondante. En cas de défaillance d’un blocs d’alimentation, l’appareil continue à fonctionner normalement sur l’autre bloc jusqu’au remplacement du module défaillant. Le tableau suivant liste les spécifications techniques des blocs d’alimentation.

| Caractéristique           | BLOC D’ALIMENTATION DE 550 W                  |
|-------------------------|----------------------------|
| Puissance de sortie maximale    | 550 W                      |
| Dissipation thermique (maximale)                   | 2 891 BTU/heure                |
| Fréquence               | 50/60 Hz                   |
| Sélection de la plage de tension | Détermination automatique : 115-230 V CA |
| Enfichable à chaud           | Oui                        |

## <a name="network-specifications"></a>Spécifications réseau

L’appareil Azure Stack Edge Pro R dispose de quatre interfaces réseau, PORT1 à PORT4. 


|Caractéristique  |Description                              |
|----------------------|----------------------------------|
|Interfaces réseau    |**2 x 1 GbE RJ45** <br> Le PORT 1 est utilisé comme interface de gestion pour la configuration initiale et est statique par défaut. Une fois l’installation initiale terminée, vous pouvez l’utiliser pour les données avec n’importe quelle adresse IP. Toutefois, après réinitialisation, l’interface revient à l’adresse IP statique. <br>L’autre d’interface, le PORT 2, est configurable par l’utilisateur, peut être utilisée pour le transfert de données et utilise le protocole DHCP par défaut.     |
|Interfaces réseau    |**2 x 25 GbE SFP28** <br> Ces interfaces de données, le PORT 3 et le PORT 4, peuvent être configurées pour utiliser le protocole DHCP (par défaut) ou être statiques.            |

Votre appareil Azure Stack Edge Pro R est équipé des composants matériels réseau suivants :

* **Carte réseau Mellanox Dual Port 25G ConnectX - 4 canaux** : PORT 3 et PORT 4. 

<!--Here are the details for the Mellanox card: MCX4421A-ACAN

| Parameter           | Description                 |
|-------------------------|----------------------------|
| Model    | ConnectX®-4 Lx EN network interface card                      |
| Model Description               | 25GbE dual-port SFP28; PCIe3.0 x8; ROHS R6                    |
| Device Part Number (XR2) | MCX4421A-ACAN  |
| PSID (R640)           | MT_2420110034                         |-->
<!-- confirm w/ Ravi what is this-->

Pour obtenir la liste complète des câbles, commutateurs et transmetteurs pris en charge pour ces cartes réseau, consultez le document : [Produits compatibles avec la carte réseau Mellanox Dual Port 25G ConnectX - 4 canaux](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).

## <a name="storage-specifications"></a>Spécifications de stockage

Les appareils Azure Stack Edge Pro R disposent de 8 disques de données et de 2 disques SATA 2 M.2 faisant office de disques de système d’exploitation. Pour plus d’informations, consultez [Disques SATA M.2](https://en.wikipedia.org/wiki/M.2).

#### <a name="storage-for-1-node-device"></a>Stockage pour appareil à nœud unique

Le tableau suivant présente les détails de la capacité de stockage de l’appareil à nœud unique.

|     Caractéristique                          |     Valeur             |
|--------------------------------------------|-----------------------|
|    Nombre de disques SSD     |    8                  |
|    Capacité de disque SSD unique                     |    8 To               |
|    Capacité totale                          |    64 To              |
|    Capacité utilisable totale*                  |    ~ 42 To          |

**Une partie de l’espace est réservé à un usage interne.*

<!--#### Storage for 4-node device

The following table has the details for the storage capacity of the 4-node device.

|     Specification                          |     Value             |
|--------------------------------------------|-----------------------|
|    Number of solid-state drives (SSDs)     |    32 (4 X 8 disks for 4 devices)                |
|    Single SSD capacity                     |    8 TB               |
|    Total capacity                          |    256 TB              |
|    Total usable capacity*                  |    ~ 163 TB          |

**After mirroring and parity, and reserving some space for internal use.* -->


## <a name="enclosure-dimensions-and-weight-specifications"></a>Dimensions et poids des boîtiers

Les tableaux suivants répertorient les dimensions et les caractéristiques de poids des différents boîtiers.

### <a name="enclosure-dimensions"></a>Dimensions de boîtier 

Le tableau suivant indique les dimensions de l’appareil et de l’onduleur dans le boîtier, en millimètres et en pouces.

|     Boîtier     |     Millimètres     |     Pouces     |
|-------------------|---------------------|----------------|
|    Hauteur         |    301,2            |    11,86       |
|    Largeur          |    604,5            |    23,80       |
|    Longueur         |    740,4            |    35,50       |

<!--#### For the 4-node system

For the 4-node system, the servers and the heater are shipped in a 5U case and the UPS are shipped in a 4U case.

The following table lists the dimensions of the 5U device case:  

|     Enclosure     |     Millimeters   |     Inches     |
|-------------------|-------------------|----------------|
|    Height         |    387.4          |    15.25       |
|    Width          |    604.5          |    23.80       |
|    Length         |    901.7          |    35.50       |

The following table lists the dimensions of the 4U UPS case: 

|     Enclosure     |     Millimeters   |     Inches    |
|-------------------|-------------------|---------------|
|    Height         |    342.9          |    13.5       |
|    Width          |    604.5          |   23.80       |
|    Length         |    901.7          |   35.50       |
-->

### <a name="enclosure-weight"></a>Poids du boîtier 

Le poids de l’appareil dépend de la configuration du boîtier.

|     Boîtier                                 |     Poids          |
|-----------------------------------------------|---------------------|
|    Poids total d’un appareil à nœud unique + boîtier avec bouchons     |    ~ 52 k.          |

<!--#### For the 4-node system

|     Enclosure                                 |     Weight          |
|-----------------------------------------------|---------------------|
|   Approximate weight of fully populated 4 devices + heater in 5U case     |    ~200 lbs.          |
|   Approximate weight of fully populated 4 UPS in 4U case    |    ~145 lbs.          |
-->

## <a name="enclosure-environment-specifications"></a>Caractéristiques ambiantes pour le boîtier

Cette section présente les spécifications liées à l’environnement du boîtier, telles que la température, les vibrations, les chocs et l’altitude.


|     Caractéristique              |     Valeur    |
|--------------------------------|-------------------------------------------------------------------|
|     Plage de températures          |     De 0 à 43 °C (en fonctionnement)    |
|     Vibration                  |     Méthode MIL-STD-810 514.7*<br>Procédure I CAT 4, 20                  |
|     Choc                      |     Méthode MIL-STD-810 516.7*<br>Procédure IV, logistique                 |
|     Altitude                   |     En fonctionnement :   3 000 mètres<br>Hors fonctionnement : 12 000 mètres          |

**Toutes les références ont trait à la norme MIL-STD-810G Change 1 (2014)*

## <a name="next-steps"></a>Étapes suivantes

- [Déployer votre Azure Stack Edge](azure-stack-edge-placeholder.md)

---
title: Spécifications techniques et conformité de Microsoft Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Découvrez les spécifications techniques et la conformité de votre appareil avec GPU Azure Stack Edge Pro
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: conceptual
ms.date: 10/07/2020
ms.author: alkohli
ms.openlocfilehash: 225cb9a31b73f330d8b4ed5790caacc4fa729477
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91839943"
---
# <a name="technical-specifications-and-compliance-for-azure-stack-edge-pro-with-gpu"></a>Spécifications techniques et conformité pour Azure Stack Edge Pro avec GPU 

Les composants matériels de votre appareil Azure Stack Edge Pro avec unité de traitement graphique (Graphics Processing Unit, GPU) intégrée sont conformes aux spécifications techniques et normes réglementaires décrites dans cet article. Les spécifications techniques décrivent des normes de blocs d’alimentation (PSU), de capacité de stockage, de boîtiers et environnementales.

## <a name="compute-and-memory-specifications"></a>Spécifications de calcul et de mémoire

L’appareil Azure Stack Edge Pro présente les spécifications suivantes en matière de calcul et de mémoire :

| Caractéristique           | Valeur                  |
|-------------------------|----------------------------|
| UC                     | 2 UC Intel Xeon Silver 4214 (Cascade Lake)            |
| Mémoire                  | 128 (8x16 Go) Go de RAM                     |


## <a name="compute-acceleration-specifications"></a>Spécifications de l’accélération de calcul

Une GPU est incluse sur chaque appareil Azure Stack Edge Pro qui active Kubernetes, l’apprentissage profond et des scénarios d’apprentissage automatique.

| Caractéristique           | Valeur                  |
|-------------------------|----------------------------|
| GPU   | Une ou deux GPU nVidia T4 <br> Pour plus d’informations, consultez [NVIDIA T4](https://www.nvidia.com/en-us/data-center/tesla-t4/).| 


## <a name="power-supply-unit-specifications"></a>Spécifications des blocs d’alimentation

L’appareil Azure Stack Edge Pro a deux blocs d’alimentation de 100-240 V avec des ventilateurs hautes performances. Les deux blocs d’alimentation fournissent une alimentation redondante. En cas de défaillance d’un blocs d’alimentation, l’appareil continue à fonctionner normalement sur l’autre bloc jusqu’au remplacement du module défaillant. Le tableau suivant liste les spécifications techniques des blocs d’alimentation.

| Caractéristique           | Bloc d’alimentation de 750 W                  |
|-------------------------|----------------------------|
| Puissance de sortie maximale    | 750 W                     |
| Fréquence               | 50/60 Hz                   |
| Sélection de la plage de tension | Détermination automatique : 100-240 V AC |
| Enfichable à chaud           | Oui                        |


## <a name="network-interface-specifications"></a>Spécifications de l’interface réseau

Votre appareil Azure Stack Edge Pro dispose de 6 interfaces réseau : PORT1 à PORT6.

| Caractéristique           | Description                 |
|-------------------------|----------------------------|
|  Interfaces réseau    | **2 interfaces 1 GbE** : le port d’interface de gestion 1 est utilisé pour la configuration initiale et est statique par défaut. Une fois l’installation initiale terminée, vous pouvez l’utiliser pour les données avec n’importe quelle adresse IP. Toutefois, après réinitialisation, l’interface revient à l’adresse IP statique. <br>L’autre port d’interface 2 peut être configuré par l’utilisateur, utilisé pour le transfert de données et est défini en mode DHCP par défaut. <br>**4 interfaces 25 GbE** : ces interfaces de données, Ports 3 à 6, peuvent être configurées par l’utilisateur en mode DHCP (par défaut) ou statique. Ils peuvent également opérer en tant qu’interfaces 10 GbE.  | 

Votre appareil Azure Stack Edge Pro est équipé des composants matériels réseau suivants :

* **Carte Microsoft Qlogic Cavium 25G NDC personnalisée** : Ports 1 à 4.
* **Carte réseau Mellanox Dual Port 25G ConnectX - 4 canaux** : Ports 5 et 6.

Voici les détails de la carte Mellanox :

| Paramètre           | Description                 |
|-------------------------|----------------------------|
| Modèle    | Carte d’interface réseau ConnectX®-4 Lx EN                      |
| Description du modèle               | 25 GbE double port SFP28 ; PCIe 3.0 x8 ; ROHS R6                    |
| Numéro de référence de l’appareil (R640) | MCX4121A-ACAT  |
| PSID (R640)           | MT_2420110034                         |

Pour obtenir la liste complète des câbles, commutateurs et transmetteurs pris en charge pour ces cartes réseau, consultez le document :

- [Matrice d’interopérabilité de la carte Qlogic Cavium 25G NDC](https://www.marvell.com/documents/xalflardzafh32cfvi0z/).
- [Produits compatibles avec la carte réseau Mellanox Dual Port 25G ConnectX - 4 canaux](https://docs.mellanox.com/display/ConnectX4LxFirmwarev14271016/Firmware+Compatible+Products).  

## <a name="storage-specifications"></a>Spécifications de stockage

Les appareils Azure Stack Edge Pro ont 5 disques SSD NVMe DC P4610 de 2,5", chacun d’une capacité de 1,6 To. Le lecteur de démarrage est un disque SSD SATA de 240 Go. La capacité utilisable totale pour l’appareil est d’environ 4,19 To. Le tableau suivant répertorie la capacité de stockage de l’appareil.

|     Caractéristique                          |     Valeur             |
|--------------------------------------------|-----------------------|
|    Nombre de SSD NVMe                     |    5                  |
|    Capacité d’un SSD NVMe                |    1,6 To             |
|    SSD SATA de démarrage      |    1                  |
|    Capacité de SSD de démarrage                       |    240 Go             |
|    Capacité totale                          |    8,0 To             |
|    Capacité utilisable totale*                  |    ~4,19 To          |
|    Contrôleur SAP                          |    HBA330 12 Gbits/s     |


**Après la résilience de parité et la réservation d’espace pour un usage interne.*

<!--Remove based on feedback from Ravi
## Other hardware specifications

Your Azure Stack Edge Pro device also contains the following hardware:

* iDRAC baseboard management
* Performance fans
* Custom ID module-->

## <a name="enclosure-dimensions-and-weight-specifications"></a>Dimensions et poids des boîtiers

Les tableaux suivants répertorient les dimensions et les caractéristiques de poids des différents boîtiers.

### <a name="enclosure-dimensions"></a>Dimensions de boîtier

Le tableau suivant répertorie les dimensions du boîtier 1U en millimètres et en pouces.

|     Boîtier     |     Millimètres     |     Pouces     |
|-------------------|---------------------|----------------|
|    Hauteur         |    44,45            |    1,75"       |
|    Largeur          |    434,1            |    17,09"      |
|    Longueur         |    740,4            |    29,15"      |

Le tableau suivant présente les dimensions du boîtier livré en millimètres et en pouces.

|     Package     |     Millimètres     |     Pouces     |
|-------------------|---------------------|----------------|
|    Hauteur         |    311,2            |    12,25"          |
|    Largeur          |    642,8            |    25,31"          |
|    Longueur         |    1 051,1          |    41,38"          |

### <a name="enclosure-weight"></a>Poids du boîtier

Le package de l’appareil pèse 30 kilogrammes et deux personnes sont nécessaires pour le manipuler. Le poids de l’appareil dépend de la configuration du boîtier.

|     Boîtier                                 |     Poids          |
|-----------------------------------------------|---------------------|
|    Poids total, emballage compris       |    27,7 Kg          |
|    Poids de l’appareil                       |    15,5 Kg          |

## <a name="enclosure-environment-specifications"></a>Caractéristiques ambiantes pour le boîtier

Cette section présente les caractéristiques liées à l’environnement du boîtier, telles que la température, l’humidité et l’altitude.

### <a name="temperature-and-humidity"></a>Température et humidité

|     Boîtier         |     Plage de températures ambiantes     |     Taux d’humidité ambiante     |     Point de rosée maximal     |
|-----------------------|--------------------------------------|--------------------------------------|---------------------------|
|    En fonctionnement        |    10°C à 35°C (50°F - 86°F)         |    10 % à 80 % sans condensation         |    29 °C (84 °F)            |
|    Hors fonctionnement    |    -40°C à 65°C (-40°F - 149°F)     |    5 % à 95 % sans condensation          |    33°C (91°F)            |

### <a name="airflow-altitude-shock-vibration-orientation-safety-and-emc"></a>Ventilation, altitude, chocs, vibrations, orientation, sécurité et CEM

|     Boîtier                           |     Caractéristiques en fonctionnement                                                                                                                                                                                         |
|-----------------------------------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|    Ventilation                              |    La ventilation du système s’effectue de l’avant vers l’arrière. Le système doit être utilisé avec une installation basse pression à échappement vers l’arrière. <!--Back pressure created by rack doors and obstacles should not exceed 5 pascals (0.5 mm water gauge).-->    |
|    Altitude maximale, en fonctionnement        |    3048 mètres (10 000 pieds) avec température maximale de fonctionnement en allègement de régime déterminée par les [Spécifications de température de fonctionnement en allègement de régime](#operating-temperature-de-rating-specifications).                                                                                |
|    Altitude maximale, hors fonctionnement    |    12 000 mètres (39 370 pieds)                                                                                                                                                                                         |
|    Chocs, en fonctionnement                   |    6 G pendant 11 millisecondes dans les six orientations                                                                                                                                                                         |
|    Chocs, hors fonctionnement               |    71 G pendant deux millisecondes dans les six orientations                                                                                                                                                                           |
|    Vibrations, en fonctionnement               |    0,26 G<sub>RMS</sub> 5 Hz à 350 Hz aléatoire                                                                                                                                                                                     |
|    Vibrations, hors fonctionnement           |    1,88 G<sub>RMS</sub> 10 Hz à 500 Hz pendant 15 minutes (six faces testées.)                                                                                                                                                  |
|    Orientation et montage             |    Montage en rack standard 19 pouces (1U)                                                                                                                                                                                       |
|    Sécurité et homologations                 |    EN 60950-1:2006 +A1:2010 +A2:2013 +A11:2009 +A12:2011/IEC 60950-1:2005 ed2 +A1:2009 +A2:2013 EN 62311:2008                                                                                                                                                                       |
|    CEM                                  |    FCC A, ICES-003 <br>EN 55032:2012/CISPR 32:2012  <br>EN 55032:2015/CISPR 32:2015  <br>EN 55024:2010 +A1:2015/CISPR 24:2010 +A1:2015  <br>EN 61000-3-2:2014/IEC 61000-3-2:2014 (Classe D)   <br>EN 61000-3-3:2013/IEC 61000-3-3:2013                                                                                                                                                                                         |
|    Énergie             |    Commission de régulation (UE) N° 617/2013                                                                                                                                                                                        |
|    RoHS           |    EN 50581:2012                                                                                                                                                                                        |

### <a name="operating-temperature-de-rating-specifications"></a>Spécifications de température de fonctionnement en allègement de régime

|     Température de fonctionnement en allègement de régime     |     Plage de températures ambiantes                                                         |
|--------------------------------------------|------------------------------------------------------------------------------------------|
|    Jusqu’à 35°C (95°F)                       |    La température maximale est réduite de 1°C/300 mètres (1°F/547 ft) au-dessus de 950 m (3117 ft).    |
|    35°C à 40°C (95°F à 104°F)            |    La température maximale est réduite de 1°C/175 mètres (1°F/319 ft) au-dessus de 950 m (3117 ft).    |
|    40°C à 45°C (104°F à 113°F)           |    La température maximale est réduite de 1°C/125 mètres (1°F/228 ft) au-dessus de 950 m (3117 ft).    |

## <a name="next-steps"></a>Étapes suivantes

[Déployer votre Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-prep.md)

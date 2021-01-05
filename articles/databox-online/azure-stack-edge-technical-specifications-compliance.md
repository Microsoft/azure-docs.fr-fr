---
title: Spécifications techniques et conformité de Microsoft Azure Stack Edge Pro | Microsoft Docs
description: Découvrir les spécifications techniques et la conformité de votre ressource Azure Stack Edge Pro
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: article
ms.date: 03/02/2020
ms.author: alkohli
ms.openlocfilehash: 8b36bb34f4c9081d807998cb8287797443625a6b
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460301"
---
# <a name="azure-stack-edge-pro-technical-specifications"></a>Spécifications techniques d’Azure Stack Edge Pro

Les composants matériels de l’appareil Microsoft Azure Stack Edge Pro sont conformes aux caractéristiques techniques et normes réglementaires présentées dans cet article. Les spécifications techniques décrivent les blocs d’alimentation (PSU), la capacité de stockage, les boîtiers et les normes environnementales.

## <a name="compute-memory-specifications"></a>Spécifications de calcul et de mémoire

L’appareil Azure Stack Edge Pro présente les spécifications suivantes en matière de calcul et de mémoire :

| Caractéristique           | Valeur                  |
|-------------------------|----------------------------|
| UC    | 2 processeurs 10 cœurs Intel Xeon Silver 4114 2.2G                    |
| Mémoire              | 128 Go de RAM (8 x16 Go RDIMM)                 |

## <a name="fpga-specifications"></a>Spécifications FPGA

Pour les scénarios de Machine Learning (ML), chaque appareil Azure Stack Edge Pro dispose d’un FPGA (Field Programmable Gate Array).

| Caractéristique           | Valeur                  |
|-------------------------|----------------------------|
| FPGA   | Intel Arria 10 <br> Les modèles à réseau neuronal profond disponibles sont les mêmes que ceux [pris en charge par les instances cloud FPGA](../machine-learning/how-to-deploy-fpga-web-service.md#fpga-support-in-azure).|

## <a name="power-supply-unit-specifications"></a>Spécifications des blocs d’alimentation

L’appareil Azure Stack Edge Pro est équipé de deux blocs d’alimentation de 100-240 V avec des ventilateurs hautes performances. Les deux blocs d’alimentation fournissent une alimentation redondante. En cas de défaillance d’un blocs d’alimentation, l’appareil continue à fonctionner normalement sur l’autre bloc jusqu’au remplacement du module défaillant. Le tableau suivant liste les spécifications techniques des blocs d’alimentation.

| Caractéristique           | Bloc d’alimentation de 750 W                  |
|-------------------------|----------------------------|
| Puissance de sortie maximale    | 750 W                     |
| Fréquence               | 50/60 Hz                   |
| Sélection de la plage de tension | Détermination automatique : 100-240 V AC |
| Enfichable à chaud           | Oui                        |

### <a name="azure-stack-edge-pro-power-cord-specifications-by-region"></a>Spécifications du cordon d’alimentation d’Azure Stack Edge Pro par région

Votre appareil Azure Stack Edge Pro a besoin d’un cordon d’alimentation différent en fonction de votre région Azure.
Pour voir les spécifications techniques de tous les cordons d’alimentation pris en charge, consultez [Spécifications du cordon d’alimentation d’Azure Stack Edge Pro par région](azure-stack-edge-technical-specifications-power-cords-regional.md).

<!--## Power consumption statistics

The following table lists the typical power consumption data (actual values may vary from the published) for the Azure Stack Edge Pro device.-->

## <a name="network-interface-specifications"></a>Spécifications de l’interface réseau

Votre appareil Azure Stack Edge Pro dispose de 6 interfaces réseau : PORT1 à PORT6.

| Caractéristique           | Description                 |
|-------------------------|----------------------------|
|  Interfaces réseau    | 2 interfaces 1 GbE : une pour la gestion, non configurable par l’utilisateur, utilisée pour l’installation initiale. L’autre interface peut être configurée par l’utilisateur, utilisée pour le transfert de données et est définie en mode DHCP par défaut. <br>2 interfaces 25 GbE : peuvent également fonctionner comme interfaces 10 GbE. Ces interfaces de données peuvent être configurées par l’utilisateur en mode statique ou DHCP (par défaut). <br> 2 interfaces 25 GbE : ces interfaces de données peuvent être configurées par l’utilisateur en mode statique ou DHCP (par défaut).                  |

Les cartes réseau utilisées sont les suivantes : 

| Caractéristique           | Description                 |
|-------------------------|----------------------------|
|Carte réseau fille (rNDC) |QLogic FastLinQ 41264 double port 25GbE SFP+, double port 1GbE, rNDC|
|Carte réseau PCI |Carte QLogic FastLinQ 41262 double port 25 Gbit/s SFP28|

Consultez la liste de compatibilité matérielle d’Intel QLogic pour connaître le convertisseur d’interface Gigabit compatible (GBIC). Le convertisseur d’interface Gigabit (GBIC) n’est pas inclus dans la distribution d’Azure Stack Edge. 

## <a name="storage-specifications"></a>Spécifications de stockage

Les appareils Azure Stack Edge Pro sont dotés de 9 disques SSD NVMe de 2,5", chacun d’une capacité de 1,6 To. Un de ces disques SSD est dédié au système d’exploitation, les huit autres étant des disques de données. La capacité utilisable totale pour l’appareil est d’environ 12,5 To. Le tableau suivant présente les détails de la capacité de stockage de l’appareil.

|     Caractéristique                          |     Valeur             |
|--------------------------------------------|-----------------------|
|    Nombre de disques SSD     |    8                  |
|    Capacité de disque SSD unique                     |    1,6 To             |
|    Capacité totale                          |    12,8 To            |
|    Capacité utilisable totale*                  |    ~ 12,5 To            |

**Une partie de l’espace est réservé à un usage interne.*

## <a name="enclosure-dimensions-and-weight-specifications"></a>Dimensions et poids des boîtiers

Les tableaux suivants répertorient les dimensions et les caractéristiques de poids des différents boîtiers.

### <a name="enclosure-dimensions"></a>Dimensions de boîtier

Le tableau suivant répertorie les dimensions du boîtier en millimètres et en pouces.

|     Boîtier     |     Millimètres     |     Pouces     |
|-------------------|---------------------|----------------|
|    Hauteur         |    44,45            |    1,75"          |
|    Largeur          |    434,1           |    17,09"          |
|    Longueur          |    740,4           |    29,15"          |

Le tableau suivant présente les dimensions du boîtier livré en millimètres et en pouces.

|     Package     |     Millimètres     |     Pouces     |
|-------------------|---------------------|----------------|
|    Hauteur         |    311,2            |    12,25"          |
|    Largeur          |    642,8          |    25,31"          |
|    Longueur          |   1 051,1          |    41,38"          |

### <a name="enclosure-weight"></a>Poids du boîtier

Le package de l’appareil pèse 28 kilogrammes et deux personnes sont nécessaires pour le manipuler. Le poids de l’appareil dépend de la configuration du boîtier.

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
|    Orientation et montage             |    Montage en rack 19"                                                                                                                                                                                        |
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

- [Déployer votre Azure Stack Edge Pro](azure-stack-edge-deploy-prep.md)
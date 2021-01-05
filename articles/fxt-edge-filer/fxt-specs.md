---
title: Spécifications Microsoft Azure FXT Edge Filer | Microsoft Docs
description: Découvrez les spécifications physiques et environnementales applicables au matériel Azure FXT Edge Filer.
author: ekpgh
ms.service: fxt-edge-filer
ms.topic: conceptual
ms.date: 06/20/2019
ms.author: rohogue
ms.openlocfilehash: b1a172adb54593421ef56e09c131e0ddcaee9f6e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92215822"
---
# <a name="azure-fxt-edge-filer-specifications"></a>Spécifications Azure FXT Edge Filer

Cet article explique les spécifications matérielles des nœuds physiques Azure FXT Edge Filer. En pratique, trois nœuds ou plus sont configurés ensemble pour fournir le système de cache en cluster.

## <a name="hardware-specifications"></a>Spécifications matérielles

| Composant | FXT 6600 | FXT 6400 |
|----------|-----------|-----------|
| Cœurs d’unité centrale |  16 | 16 |
| DRAM  | 1536 Go | 768 Go |
| Ports réseau | 6 x 25/10 Go + 2 x 1 Go | 6 x 25/10 Go + 2 x 1 Go |
| Capacité de disque SSD NVMe | 25,6 To | 12,8 To |

## <a name="drive-specifications"></a>Caractéristiques des lecteurs

Le système dispose de dix baies de disques accessibles en façade. Chaque lecteur rempli est étiqueté à droite avec des informations de capacité.

Les numéros de disque figurent dans l’espace entre les disques. Dans Azure FXT Edge Filer, le disque 0 est le disque supérieur gauche, et le disque 1 est directement en dessous.

![photo d’une baie de disque dur dans le châssis FXT, montrant les numéros de disque et les étiquettes de capacité](media/fxt-drives-photo.png)

| Numéros de lecteur    |  Utilisation   |  Spécifications |
|------------------|--------|-----------------|
| 0, 1             | Système d''exploitation     | SSD SATA de 480 Go |
| 2, 3, 4, 5, 6, 7, 8, 9 | Données   | FXT 6600 : Disque SSD NVMe de 3,2 To <br> FXT 6400 : Disque SSD NVMe de 1,6 To |

## <a name="dimensions-and-weight"></a>Dimensions et poids

Azure FXT Edge Filer est conçu pour s’adapter dans un rack d’équipement standard de 19" et mesure une unité de rack en hauteur (1U).

<!-- 10x2.5 inches version -->

| Dimensions du serveur de fichiers            | Valeur                    |
|-----------------------------|--------------------------|
| Hauteur                      | 42,8 mm (1,68 pouce)    |
| Largeur (y compris les pattes) | 482,0 mm (18,97 pouces)  |
| Largeur - boîtier principal      | 434,0 mm (17,08 pouces) |
| Profondeur - entre les pattes et l’arrière du boîtier principal                   | 733,82 mm (29,61 pouces) |
| Profondeur - entre les pattes et la protrusion arrière la plus éloignée                 | 772,67 mm (30,42 pouces) |
| Profondeur - entre les pattes et la protrusion avant la plus éloignée, sans cadre | 22,0 mm (0,87 pouce)  |
| Profondeur - entre les pattes et la protrusion avant la plus éloignée, cadre compris    | 35,84 mm (1,41 pouce) |

| Poids | Valeur |
|-----------------|----------------------|
| Poids du nœud (sans emballage, ni accessoires) | 40 lb (18,1 kg) |
| Poids net (sans emballage, avec les accessoires) | 51 lb (23,1 kg)|
| Poids brut (inclut tous les emballages d’expédition) |  64 lb (29,0 kg) |

### <a name="shipping-dimensions"></a>Dimensions de livraison

| Dimensions du package | Millimètres | Pouces |
|-------------------|-------------|--------|
| Hauteur            | 311,2       | 12,25" |
| Largeur             | 642,8       | 25,31" |
| Longueur            | 1 051,1     | 41,38" |

## <a name="power-and-thermal-specifications"></a>Caractéristiques du module d’alimentation et thermiques

Cette section fournit les puissances nominales et les mesures de l’appareil Azure FXT Edge Filer.

### <a name="nameplate-ratings"></a>Plaque signalétique

| Plaque signalétique des modèles de la série FXT 6000 |
|----------------|
| 100 - 240 V~    |
| 10 A - 5 A (X2)  |
| 50/60 Hz         |

<!-- matches the Dell regulatory label exactly -->

### <a name="power-and-thermal-measurements"></a>Mesures électriques et thermiques

Les nœuds Azure FXT Edge Filer utilisent des ventilateurs à vitesse variable ; la puissance dépend donc de la température et de la charge. Les vitesses maximales des ventilateurs peuvent être atteintes à certaines combinaisons de charges élevées et de températures ambiantes élevées.

Ces graphiques fournissent les mesures de consommation d’énergie et de puissance thermique pour les combinaisons tension-fréquence courantes.

| Alimentation FXT 6600 à température ambiante <br />(22 °C, 71,6 °F) | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz |
|---------|---|---|---|---|---|
| Tension (V) | 100 | 120 | 208 | 230 | 240 |
| Fréquence (Hz) | 60 | 60 | 60 | 50 | 50 |
| Intensité (A) | 5.02 | 4.16 |2.40 | 2.20 | 2.16 |
| Puissance apparente (VA) | 502 | 499 | 499 | 506 | 518|
| Facteur de puissance | 0,99 | 0,99 |0.98 | 0.98 | 0.98 |
| Puissance réelle (W) | 497 |494 | 489 | 496 | 508 |
| Dissipation thermique (BTU/h) |1696 | 1686 | 1669 | 1692 | 1733 |

| Alimentation FXT 6600, ventilateurs à pleine vitesse | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz |
|---------|---|---|---|---|---|
| Tension (V) | 100 |120 | 208 | 230 | 240|
| Fréquence (Hz) | 60 | 60 | 60 | 50 | 50 |
| Intensité (A) | 5,98 | 5,01 | 2.81 | 2.55 | 2.48 |
| Puissance apparente (VA) | 598 | 601 | 584 | 587 | 595 |
| Facteur de puissance | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 |
| Puissance réelle (W) | 592 | 595 | 573 | 575 | 583 |
| Dissipation thermique (BTU/h) | 2020 |2031 | 1954 | 1961 | 1990 |

| Alimentation FXT 6400 à température ambiante <br />(22 °C, 71,6 °F) | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz |
|---------|---|---|---|---|---|
| Tension (V) | 100 | 120 | 208 | 230 | 240 |
| Fréquence (Hz) |60 | 60 | 60 | 50 | 50 |
| Intensité (A) | 4.63 | 3,86 | 2.24 | 2,04 | 1,94 |
| Puissance apparente (VA) | 463 | 463 | 466 | 469 | 466 |
| Facteur de puissance | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 |
| Puissance réelle (W) | 458 | 459 | 457 | 460 | 456 |
| Dissipation thermique (BTU/h) | 1564 | 1565 | 1558 | 1569 | 1557 |

| Alimentation FXT 6400, ventilateurs à pleine vitesse | 100 V, 60 Hz | 120 V, 60 Hz | 208 V, 60 Hz | 230 V, 50 Hz | 240 V, 50 Hz |
|---------|---|---|---|---|---|
| Tension (V) | 100 | 120 | 208 | 230 | 240 |
| Fréquence (Hz) | 60 | 60 | 60 | 50 | 50 |
| Intensité (A) | 5.15 | 4.28 | 2.48 | 2.28 | 2.13 |
| Puissance apparente (VA) | 515 | 514 | 516 | 524 | 511 |
| Facteur de puissance | 0,99 | 0,99 | 0.98 | 0.98 | 0.98 |
| Puissance réelle (W) | 510 | 508 | 506 | 514 | 501 |
| Dissipation thermique (BTU/h) | 1740 | 1735 | 1725 | 1753 | 1709 |

## <a name="environmental-requirements"></a>Configuration requise de l’environnement

Cette section fournit des spécifications pour l’environnement ambiant du matériel.

### <a name="temperature-and-humidity"></a>Température et humidité

| Facteur environnemental   | Plage de fonctionnement                   | Plage hors exploitation         |
|---------------------------|-----------------------------------|-----------------------------|
| Plage de températures ambiantes | 10 °C à 35 °C (50-86 °F)          | -40 °C à 65 °C (-40 °F - 149 °F) |
| Taux d’humidité ambiante | 10 % à 80 % sans condensation          | 5 % à 95 % sans condensation     |
| Point de rosée maximal         | 29 °C (84 °F)                       | 33°C (91°F)                 |
| Altitude                  | Jusqu’à 3 048 mètres (10 000 pieds), soumis aux températures en allègement de régime ci-dessous | Jusqu’à 12 000 mètres (39 370 pieds) |

> [!NOTE]
> **Température de fonctionnement en allègement de régime selon l’altitude :** La température maximale est réduite de 1 °C/300 mètres (1 °F/547 ft) au-dessus de 950 m (3 117 ft).

### <a name="airflow-shock-and-vibration"></a>Ventilation, chocs et vibrations

| Attribut         | Caractéristique |
|-------------------|---------------|
| Ventilation                    | La ventilation du système s’effectue de l’avant vers l’arrière. Le système doit être utilisé avec une installation basse pression à échappement vers l’arrière. |
| Chocs, en fonctionnement         | 6 G pendant 11 millisecondes (testé dans les six orientations) |
| Chocs, hors fonctionnement     | 71 G pendant deux millisecondes (testé dans les six orientations) |
| Vibrations, en fonctionnement     | 0,26 G<sub>RMS</sub> 5 Hz à 350 Hz aléatoire         |
| Vibrations, hors fonctionnement | 1,88 G<sub>RMS</sub> 10 Hz à 500 Hz pendant 15 minutes (six faces testées)  |

## <a name="safety-regulation-compliance"></a>Conformité aux réglementations de sécurité

Azure FXT Edge Filer est conforme aux réglementations en vigueur.

| Category       | Spécification réglementaire |
|----------------|--------------------------|
| Sécurité générale | EN 60950-1:2006 +A1:2010 +A2:2013 +A11:2009 +A12:2011/IEC 60950-1:2005 ed2 +A1:2009 +A2:2013 <br>EN 62311:2008 |
| CEM            | FCC A, ICES-003  <br>EN 55032:2012/CISPR 32:2012  <br>EN 55032:2015/CISPR 32:2015  <br>EN 55024:2010 +A1:2015/CISPR 24:2010 +A1:2015  <br>EN 61000-3-2:2014/IEC 61000-3-2:2014 (Classe D)   <br>EN 61000-3-3:2013/IEC 61000-3-3:2013 |
| Énergie         | Commission de régulation (UE) N° 617/2013  |
| RoHS           |    EN 50581:2012   |

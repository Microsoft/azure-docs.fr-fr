---
title: Disponibilité de StorSimple par région
description: Décrit les régions Azure dans lesquelles les différents modèles d’appareil StorSimple sont disponibles.
author: alkohli
ms.service: storsimple
ms.topic: conceptual
ms.date: 11/16/2017
ms.author: alkohli
ms.openlocfilehash: 5efb87ad674474b2e21b86b5cdc48785753a51dc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90564149"
---
# <a name="available-regions-for-your-storsimple"></a>Régions disponibles pour votre appareil StorSimple

[!INCLUDE [storsimple-8000-eol-banner](../../includes/storsimple-8000-eol-banner.md)]

## <a name="overview"></a>Vue d’ensemble

Les centres de données Azure fonctionnent dans différentes zones géographiques du monde pour répondre aux besoins de performances, aux obligations et aux préférences des clients pour ce qui est de l’emplacement des données. Une zone géographique Azure est une zone définie du monde contenant au moins une région Azure. Une région Azure est une zone géographique contenant un ou plusieurs centres de données.

Le choix d’une région Azure est très important. Il est influencé par des facteurs comme la résidence et la souveraineté des données, la disponibilité du service, les performances, le coût et la redondance. Pour plus d’informations sur le choix d’une région, accédez à [Quelle région Azure est appropriée pour moi ?](https://azure.microsoft.com/overview/datacenters/how-to-choose/)

Pour la solution StorSimple, le choix de la région est particulièrement déterminé par les facteurs suivants :

- Régions dans lesquelles le service StorSimple Device Manager est disponible.
- Pays/régions où l’appareil physique, cloud ou virtuel StorSimple est disponible.
- Régions où les comptes de stockage qui stockent les données StorSimple doivent se trouver pour des performances optimales.

Ce didacticiel décrit la disponibilité des régions pour le service StorSimple Device Manager, ainsi que les appareils physiques locaux et cloud. Les informations contenues dans cet article s’appliquent aux appareils des gammes StorSimple 8000 et 1200.

## <a name="region-availability-for-storsimple-device-manager-service"></a>Disponibilité des régions pour le service StorSimple Device Manager

Le service StorSimple Device Manager est actuellement pris en charge dans 12 régions publiques et 2 régions Azure Government.

Vous définissez une région ou un emplacement au moment où vous créez le service StorSimple Device Manager. De manière générale, choisissez l’emplacement le plus proche de la région géographique dans laquelle vous déployez l’appareil. Toutefois, l’appareil et le service peuvent également être déployés à différents emplacements.

Voici la liste des régions où StorSimple Device Manager est disponible pour le cloud public Azure et peut être déployé.

![Capture d’écran affichant StorSimple Device Manager avec la liste d’emplacements mise en surbrillance.](./media/storsimple-region/storsimple-device-manager-service-regions.png)

Pour le cloud Azure Government, le service StorSimple Device Manager est disponible dans les centres de données US Gov Iowa et US Gov Virginie.

## <a name="region-availability-for-data-stored-in-storsimple"></a>Disponibilité des régions pour les données stockées dans StorSimple

Les données StorSimple sont physiquement stockées dans des comptes de stockage Azure et ces comptes sont disponibles dans toutes les régions Azure. Lorsque vous créez un compte de stockage Azure, vous choisissez l’emplacement principal du compte de stockage et ce choix détermine la région dans laquelle résident les données.

Quand vous commencez par créer un service StorSimple Device Manager et lui associez un compte de stockage, votre service StorSimple Device Manager et le stockage Azure peuvent être à deux endroits distincts. Dans ce cas, vous devez créer séparément le gestionnaire d’appareil StorSimple et le compte de stockage Azure.

En général, choisissez la région la plus proche de votre service pour votre compte de stockage. Toutefois, la région Microsoft Azure la plus proche peut ne pas être la région dont la latence est la plus faible. Or, c’est la latence qui dicte les performances du service réseau et par conséquent, les performances de la solution. Donc, si vous choisissez un compte de stockage dans une autre région, il est important de connaître les latences entre votre service et la région associée à votre compte de stockage.

Si vous utilisez StorSimple Cloud Appliance, nous recommandons que le service et le compte de stockage associé soient dans la même région. Des comptes de stockage placés dans des régions différentes peuvent entraîner une dégradation des performances.

## <a name="availability-of-storsimple-device"></a>Disponibilité de l’appareil StorSimple

Selon le modèle, les appareils StorSimple peuvent être disponibles dans des zones géographiques ou des pays/régions différent(e)s.

### <a name="storsimple-physical-device-models-81008600"></a>Appareil physique StorSimple (modèles 8100/8600)

Si vous utilisez un appareils physique StorSimple 8100 ou 8600, il est disponible dans les pays/régions suivant(e)s.

| #  | Pays/région        | #  | Pays/région     | #  | Pays/région      | #  | Pays/région             |
|----|-----------------------|----|--------------------|----|---------------------|----|----------------------------|
| 1  | Australie             | 16 | Hong Kong (R.A.S.)      | 31 | Nouvelle-Zélande         | 46 | Afrique du Sud               |
| 2  | Autriche               | 17 | Hongrie            | 32 | Nigeria             | 47 | Corée du Sud                |
| 3  | Bahreïn               | 18 | Islande            | 33 | Norvège              | 48 | Espagne                      |
| 4  | Belgique               | 19 | Inde              | 34 | Pérou                | 49 | Sri Lanka                  |
| 5  | Brésil                | 20 | Indonésie          | 35 | Philippines         | 50 | Suède                     |
| 6  | Canada                | 21 | Irlande            | 36 | Pologne              | 51 | Suisse                |
| 7  | Chili                 | 22 | Israël             | 37 | Portugal            | 52 | Taïwan                     |
| 8  | Colombie              | 23 | Italie              | 38 | Porto Rico         | 53 | Thaïlande                   |
| 9  | République tchèque        | 24 | Japon              | 39 | Qatar               | 54 | Turquie                     |
| 10 | Danemark               | 25 | Kenya              | 40 | Roumanie             | 55 | Ukraine                    |
| 11 | Égypte                 | 26 | Koweït             | 41 | Russie              | 56 | Émirats Arabes Unis       |
| 12 | Finlande               | 27 | Macao R.A.S.          | 42 | Arabie Saoudite        | 57 | United Kingdom             |
| 13 | France                | 28 | Malaisie           | 43 | Singapour           | 58 | États-Unis              |
| 14 | Allemagne               | 29 | Mexique             | 44 | Slovaquie            | 59 | Vietnam                    |
| 15 | Grèce                | 30 | Pays-bas        | 45 | Slovénie            | 60 | Croatie                    |

Cette liste évolue au fur et à mesure que de nouveaux pays/régions sont ajouté(e)s. Pour obtenir la liste actualisée des zones géographiques, accédez à l’annexe sur les conditions des groupes de stockage dans les [conditions du produit](https://www.microsoft.com/en-us/licensing/product-licensing/products).

Microsoft peut livrer le matériel physique et fournir des pièces matérielles de rechange pour StorSimple dans les zones géographiques indiquées dans la liste précédente.

> [!IMPORTANT]
> Ne placez pas d’appareil physique StorSimple dans une région où StorSimple n’est pas pris en charge. Microsoft ne pourra pas être en mesure d’expédier des pièces de rechange dans les pays/régions où StorSimple n’est pas pris en charge.

### <a name="storsimple-cloud-appliance-models-80108020"></a>StorSimple Cloud Appliance (modèles 8010/8020)

Si vous utilisez StorSimple Cloud Appliance 8010 ou 8020, l’appareil est pris en charge et disponible dans toutes les régions où la machine virtuelle sous-jacente est prise en charge. La gamme 8010 utilise une machine virtuelle _Standard_A3_ prise en charge dans toutes les régions Azure.

La gamme 8020 utilise un stockage Premium et une machine virtuelle _Standard_DS3_ pour créer une appliance cloud. La gamme 8020 est prise en charge dans les régions Azure qui prennent en charge le stockage Premium et les machines virtuelles Azure _Standard_DS3_. Utilisez [cette liste](https://azure.microsoft.com/regions/services/) afin de déterminer si **Machines virtuelles > Série DS** et **Stockage > Stockage sur disque** sont disponibles dans votre région.

### <a name="storsimple-virtual-array-model-1200"></a>StorSimple Virtual Array (modèle 1200)

Si vous utilisez la gamme 1200 de StorSimple Virtual Array, l’image de disque virtuelle est prise en charge dans toutes les régions Azure.

## <a name="next-steps"></a>Étapes suivantes

* Découvrez les [tarifs de différents modèles StorSimple](https://azure.microsoft.com/pricing/calculator/#storsimple2).
* En savoir plus sur la [gestion de votre compte de stockage StorSimple](storsimple-8000-manage-storage-accounts.md).
* En savoir plus sur [l’utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

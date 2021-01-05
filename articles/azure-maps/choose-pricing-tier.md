---
title: Choisir le bon niveau tarifaire pour Microsoft Azure Maps
description: En savoir plus sur les niveaux tarifaires de Azure Maps. Découvrez les fonctionnalités proposées aux niveaux et les principales considérations à prendre en compte pour le choix d’un niveau tarifaire.
author: anastasia-ms
ms.author: v-stharr
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: philmea
ms.openlocfilehash: cb57e424642141e73588a61d026d1ff2f1a8b096
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96905296"
---
# <a name="choose-the-right-pricing-tier-in-azure-maps"></a>Choix du bon niveau tarifaire dans Azure Maps

Azure Maps propose deux niveaux tarifaires : S0 et S1. Cet article a pour objectif de vous aider à choisir le niveau tarifaire qui répond le mieux à vos besoins. Pour choisir le niveau tarifaire correct, posez-vous les deux questions suivantes.

## <a name="how-many-concurrent-users-do-i-plan-to-support"></a>Combien d’utilisateurs simultanés dois-je pouvoir prendre en charge ?

Les niveaux tarifaires S0 et S1 gèrent différents débits de données. Le niveau tarifaire S0 gère jusqu’à **50 requêtes par seconde**. Le niveau tarifaire S1 gère **plus de 50 requêtes par seconde**.

## <a name="what-geospatial-capabilities-do-i-plan-to-use"></a>Quelles fonctionnalités géospatiales ai-je besoin d’utiliser ?

Si les principales API géospatiales répondent aux besoins de votre service, choisissez le niveau tarifaire S0. Si vous souhaitez des capacités plus avancées pour votre application, le niveau tarifaire S1 vous conviendra sans doute le mieux. Les fonctionnalités avancées sont les suivantes : imagerie aérienne et hybride, obtention de zone d’itinéraire et géocodage par lot. Pour sélectionner le niveau tarifaire le plus adapté pour votre application, passez en revue le tableau **Capacités des niveaux tarifaires** ci-dessous :

### <a name="pricing-tier-capabilities"></a>Fonctionnalités des niveaux tarifaires

| Fonctionnalité                              |        S0           |  S1      |
|-----------------------------------------|:-------------------:|:--------:|
| Rendu de carte                              | ✓                   | ✓       |
| Imagerie satellite                       |                     | ✓        |
| Recherche                                  | ✓                    | ✓        |
| Recherche par lot                            |                     | ✓        |
| Routage                                   | ✓                    |✓        |
| Routage par lot                            |                    | ✓        |
| Routage par matrice                          |                     | ✓        |
| Zone d’itinéraire (Isochrones)                |                     | ✓        |
| Trafic                                |✓                    |✓        |
| Time Zone (Fuseau horaire)                               |✓                    |✓        |
| Géolocalisation (préversion)                    |✓                   |✓        |
| Opérations spatiales                        |                    |✓        |
| Geofencing                                |                    |✓        |
| Données Azure Maps (préversion)                |                     | ✓        |
| Mobilité (préversion)                       |                     | ✓        |
| Météo (préversion)                        |✓                    |✓        |
|  Créateur (préversion)                         |                   |✓        |
|  Elevation (préversion)                        |                   |✓        |

Prenez en compte ces autres points :

* Quel est le type de votre entreprise ?
* Quelle est l’importance de votre application ?

### <a name="pricing-tier-targeted-customers"></a>Niveau tarifaire et clients ciblés

Pour mieux comprendre les niveaux tarifaires S0 et S1, consultez le tableau **Niveau tarifaire et clients ciblés**. Pour plus d’informations, consultez la page [Tarification Azure Maps](https://azure.microsoft.com/pricing/details/azure-maps/). 

| Niveau tarifaire  |     Clients ciblés                                                                |
|-----------------|:-----------------------------------------------------------------------------------------|
| S0            |    Le niveau tarifaire S0 convient pout toutes les phases de production des applications : du développement « preuve de concept » à la phase de test initiale, en passant par la mise en production et le déploiement. Toutefois, ce niveau est conçu pour un développement à petite échelle, des clients avec peu d’utilisateurs simultanés, ou les deux. 
| S1            |    Le niveau tarifaire S1 convient aux clients qui ont des applications d’entreprise à grande échelle, des applications stratégiques ou un grand nombre d’utilisateurs simultanés. Il convient également aux clients qui ont besoin de services géospatiaux avancés.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur l’affichage et la modification des niveaux tarifaires :

> [!div class="nextstepaction"]
> [Gérer un niveau tarifaire](how-to-manage-pricing-tier.md)

---
title: Fonctionnalités de la solution Usine connectée - Azure | Microsoft Docs
description: Cet article propose une vue d’ensemble des fonctionnalités de la solution préconfigurée Usine connectée, comme le tableau de bord cloud, les règles et les alertes.
author: dominicbetts
manager: timlt
ms.service: iot-accelerators
services: iot-accelerators
ms.topic: conceptual
ms.date: 06/10/2019
ms.author: dobett
ms.openlocfilehash: e492ebf70f69c985691852a1c1f2351d1e09578e
ms.sourcegitcommit: 6906980890a8321dec78dd174e6a7eb5f5fcc029
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92425013"
---
# <a name="what-is-connected-factory-iot-solution-accelerator"></a>Qu’est-ce que l’accélérateur de solution IoT Usine connectée ?

> [!IMPORTANT]
> Pendant la mise à jour de cet article, consultez [Azure Industrial IoT](https://azure.github.io/Industrial-IoT/) pour obtenir le contenu le plus récent.

Usine connectée est une implémentation de l’architecture de référence de Microsoft Azure Industrial IoT, sous la forme d’une solution open source. Vous pouvez l’utiliser comme point de départ pour un produit commercial. Vous pouvez déployer une version prédéfinie de la solution Usine connectée dans votre abonnement Azure à partir des [accélérateurs de solution Azure IoT Suite](https://www.azureiotsolutions.com/#solutions/types/CF).

![Tableau de bord de la solution Usine connectée](./media/iot-accelerators-connected-factory-features/dashboard.png)

Le [code de l’accélérateur de solution Usine connectée est disponible sur GitHub](https://github.com/Azure/azure-iot-connected-factory).

Usine connectée propose les fonctionnalités suivantes :

## <a name="industrial-device-interoperability"></a>Interopérabilité des appareils industriels

- Connectez-vous aux ressources industrielles avec une interface OPC UA.
- Utilisez les lignes de production simulées (serveurs OPC UA en cours d’exécution dans des conteneurs Docker) pour voir leurs données de télémétrie en direct.
- Parcourez le modèle d’information OPC UA des serveurs OPC UA à partir d’un tableau de bord cloud.

## <a name="remote-management"></a>Gestion à distance

- Configurez vos ressources OPC UA à partir du tableau de bord cloud (appel de méthodes, lecture et écriture de données).
- Lancez ou annulez la publication des données de télémétrie de vos ressources OPC UA à partir d’un tableau de bord cloud.

## <a name="cloud-dashboard"></a>Tableau de bord cloud

- Affichez les aperçus de données de télémétrie directement dans un tableau de bord cloud.
- Affichez les tendances dans les données de télémétrie et créez des corrélations à l’aide du tableau de bord Explorateur de Time Series Insights.
- Consultez le taux de rendement global (TRG) calculé et les indicateurs de performance clé (KPI) à partir d’un tableau de bord cloud.
- Affichez les hiérarchies de ressources industrielles dans une topologie arborescente, ainsi que sur une carte interactive.
- Affichez, accusez la réception et fermez les alertes à partir d’un tableau de bord cloud.

## <a name="azure-time-series-insights"></a>Azure Time Series Insights

- [Azure Time Series Insights](../time-series-insights/time-series-insights-overview.md) est conçu pour stocker, visualiser et interroger de grandes quantités de données de série chronologique. Usine connectée tire parti de ce service.
- Usine connectée s’intègre dans ce service pour vous permettre d’analyser en profondeur et en temps réel les données de votre appareil.

## <a name="rules-and-alerts"></a>Règles et alertes

[Configurez des règles de seuil des alertes](iot-accelerators-connected-factory-configure.md).

## <a name="end-to-end-security"></a>Sécurité de bout en bout

- Configurez les autorisations de sécurité des utilisateurs à l'aide du contrôle d'accès en fonction du rôle (RBAC).
- Le chiffrement de bout en bout est implémenté à l’aide de l’authentification OPC UA (avec des certificats X.509) et des jetons de sécurité.

## <a name="customizability"></a>Possibilités de personnalisation

- Personnalisez la solution en fonction de vos besoins professionnels.
- Code source complet de la solution disponible sur GitHub. Consultez le dépôt sur la [solution préconfigurée Usine connectée](https://github.com/Azure/azure-iot-connected-factory).

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur l’accélérateur de solution de la fabrique connectée, consultez le guide de démarrage rapide [Essayer une solution cloud pour gérer mes appareils IoT industriels](quickstart-connected-factory-deploy.md).

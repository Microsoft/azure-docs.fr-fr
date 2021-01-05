---
title: Observabilité dans Gestion des API Azure | Microsoft Docs
description: Vue d’ensemble de toutes les options d’observabilité dans Gestion des API Azure.
services: api-management
documentationcenter: ''
author: begim
manager: alberts
editor: ''
ms.service: api-management
ms.workload: mobile
ms.tgt_pltfrm: na
ms.topic: article
ms.date: 06/01/2020
ms.author: apimpm
ms.openlocfilehash: 1ebebed465952bbb5d3e8f82ae1c7776c441c6b0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87094117"
---
# <a name="observability-in-azure-api-management"></a>Observabilité dans Gestion des API Azure

L’observabilité est la capacité à comprendre l’état interne d’un système à partir des données qu’il produit et la capacité à explorer ces données pour répondre à des questions sur ce qui s’est produit et pourquoi. 

Gestion des API Azure aide les organisations à centraliser la gestion de toutes les API. Dans la mesure où il sert de point d’entrée unique pour le trafic de toutes les API, c’est l’emplacement idéal pour observer ces dernières. 

## <a name="observability-tools"></a>Outils d’observabilité

Le tableau ci-dessous récapitule tous les outils pris en charge par Gestion des API permettant d’observer les API, chacun d’eux étant utile pour un ou plusieurs scénarios :

| Outil        | Utile pour    | Retard des données | Rétention | échantillonnage | Type de données | activé|
|:------------- |:-------------|:---- |:----|:---- |:--- |:---- 
| **[Inspecteur d’API](api-management-howto-api-inspector.md)** | Test et débogage | Immédiat | 100 dernières traces | Activé par requête | Traces des requêtes | Toujours
| **Analytique intégrée** | Création de rapports et surveillance | Minutes | Durée de vie | 100 % | Rapports et journaux | Toujours |
| **[Azure Monitor Metrics](api-management-howto-use-azure-monitor.md)** | Création de rapports et surveillance | Minutes | 93 jours (mettre à niveau pour prolonger) | 100 % | Mesures | Toujours |
| **[Journaux d’activité Azure Monitor](api-management-howto-use-azure-monitor.md)** | Création de rapports, surveillance et débogage | Minutes | 31 jours/5 Go (mettre à niveau pour prolonger/étendre) | 100% (ajustable) | Journaux d’activité | Facultatif |
| **[Azure Application Insights](api-management-howto-app-insights.md)** | Création de rapports, surveillance et débogage | Secondes | 90 jours/5 Go (mettre à niveau pour prolonger/étendre) | Custom | Journaux, métriques | Facultatif |
| **[Journalisation via Azure Event Hubs](api-management-howto-log-event-hubs.md)** | Scénarios personnalisés | Secondes | Géré par l’utilisateur | Custom | Custom | Facultatif |

### <a name="self-hosted-gateway"></a>Passerelle auto-hébergée

Tous les outils mentionnés ci-dessus sont pris en charge par la passerelle gérée dans le cloud. La [passerelle auto-hébergée](self-hosted-gateway-overview.md) n’envoie actuellement aucun journal de diagnostic à Azure Monitor. Toutefois, il est possible de configurer et de conserver des journaux localement, à l’endroit où la passerelle auto-hébergée est déployée. Pour plus d’informations, consultez [la configuration des journaux et des métriques cloud pour une passerelle auto-hébergée](how-to-configure-cloud-metrics-logs.md) et [la configuration des métriques et journaux locaux pour une passerelle auto-hébergée](how-to-configure-local-metrics-logs.md).

## <a name="next-steps"></a>Étapes suivantes

* [Suivez les didacticiels pour en savoir plus sur Gestion des API.](import-and-publish.md)

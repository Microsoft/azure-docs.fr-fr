---
title: Utiliser l’analytique des API dans Gestion des API Azure | Microsoft Docs
description: Utilisez l’analytique dans Gestion des API Azure pour mieux comprendre et catégoriser les différentes utilisations des API, ainsi que leurs performances.
author: dlepow
ms.service: api-management
ms.topic: article
ms.date: 11/24/2020
ms.author: apimpm
ms.openlocfilehash: ca7bd70bbf99a6d0079717a7a02328b11528d2e0
ms.sourcegitcommit: 8b4b4e060c109a97d58e8f8df6f5d759f1ef12cf
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/07/2020
ms.locfileid: "96839668"
---
# <a name="get-api-analytics-in-azure-api-management"></a>Obtenir l’analytique des API dans Gestion des API Azure

La Gestion des API Azure comprend une analytique intégrée pour vos API. Analysez l’utilisation et les performances des API dans votre instance de la Gestion des API sous différentes dimensions, notamment :

* Temps
* Geography
* API
* Opérations d’API
* Produits
* Abonnements
* Utilisateurs
* Demandes

:::image type="content" source="media/howto-use-analytics/analytics-report-portal.png" alt-text="Chronologie de l’analytique dans le portail":::

Utilisez l’analytique pour une supervision générale de vos API et leur dépannage. Pour obtenir des fonctionnalités de supervision supplémentaires, y compris des métriques en quasi-temps réel et des journaux de ressources pour les diagnostics et l’audit, consultez [Tutoriel : Surveiller les API publiées](api-management-howto-use-azure-monitor.md).

[!INCLUDE [premium-dev-standard-basic.md](../../includes/api-management-availability-premium-dev-standard-basic.md)]

## <a name="analytics---portal"></a>Analytique - Portail

Utilisez le portail Azure pour avoir un aperçu rapide des données d’analytique de votre instance Gestion des API.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre instance APIM. 
1. Dans le menu de gauche, sous **Supervision**, sélectionnez **Analytique**.

    :::image type="content" source="media/howto-use-analytics/monitoring-menu-analytics.png" alt-text="Sélectionnez Analytique pour l’instance Gestion des API dans le portail":::  
1. Sélectionnez un intervalle de temps pour les données, ou entrez un intervalle personnalisé.
1. Sélectionnez une catégorie de rapport pour les données d’analytique, par exemple **Chronologie** ou **Géographie**.
1. Si vous le souhaitez, vous pouvez filtrer le rapport selon une ou plusieurs catégories supplémentaires.

## <a name="analytics---rest-api"></a>Analytique - API REST

Utilisez les opérations [Rapports](/rest/api/apimanagement/2019-12-01/reports) de l’API REST Gestion des API pour récupérer et filtrer les données d’analytique de votre instance Gestion des API.

Les opérations disponibles retournent des enregistrements de rapport filtrés par API, Géographie, Opérations d’API, Produit, Demande, Abonnement, Date ou Utilisateur.

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation des fonctionnalités Azure Monitor dans Gestion des API, consultez [Tutoriel : Surveiller les API publiées](api-management-howto-use-azure-monitor.md).
* Pour plus d’informations sur la supervision et la journalisation HTTP, consultez [Superviser vos API avec la Gestion des API Azure, Event Hubs et Moesif](api-management-log-to-eventhub-sample.md).
* Découvrez comment [intégrer la Gestion des API Azure à Azure Application Insights](api-management-howto-app-insights.md).
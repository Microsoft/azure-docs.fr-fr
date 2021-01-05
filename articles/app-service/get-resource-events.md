---
title: Récupérer des événements de ressource dans Azure App Service
description: Découvrez comment obtenir des événements de ressources par le biais des journaux d’activité et d’Event Grid sur votre application App Service.
ms.topic: article
ms.date: 04/24/2020
ms.author: msangapu
ms.openlocfilehash: c5c84891187c540c0b24162cf5c8c7f96e9e731a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88962296"
---
# <a name="get-resource-events-in-azure-app-service"></a>Récupérer des événements de ressource dans Azure App Service

Azure App Service fournit des outils intégrés pour surveiller l’état et l’intégrité de vos ressources. Les événements de ressource vous aident à comprendre les modifications apportées aux ressources de votre application web sous-jacente et à agir en fonction des besoins. Voici quelques exemples d’événements : mise à l’échelle des instances, mises à jour des paramètres d’application, redémarrage de l’application web et bien plus encore. Dans cet article, vous allez apprendre à afficher les [journaux d’activité Azure](../azure-monitor/platform/activity-log.md#view-the-activity-log) et à activer [Event Grid](../event-grid/index.yml) pour surveiller les événements de ressources liés à votre application web App Service.

> [!NOTE]
> L’intégration d’App Service à Event Grid est dans la **version préliminaire**. [Pour plus d’informations, lisez l’annonce](https://aka.ms/app-service-event-grid-announcement).
>

## <a name="view-azure-activity-logs"></a>Afficher des journaux d’activité Azure
Les journaux d’activité Azure contiennent des événements de ressource émis par des opérations effectuées sur les ressources de votre abonnement. Les deux actions de l’utilisateur dans les modèles du portail Azure et Azure Resource Manager contribuent aux événements capturés par le journal d’activité. 

Les journaux d’activité Azure pour obtenir des détails App Service tels que :
- les opérations qui ont été effectuées sur les ressources ( Plans App Service)
- Qui a démarré l'opération
- Le moment où a eu lieu l’opération
- L’état de l’opération
- les valeurs de propriété pour vous aider à rechercher l’opération

### <a name="what-can-you-do-with-azure-activity-logs"></a>Que pouvez-vous faire avec les journaux d’activité Azure Monitor ?

Les journaux d’activité Azure peuvent être interrogés à l’aide du Portail Azure, de PowerShell, de l’API REST ou de l’interface CLI. Vous pouvez envoyer les journaux à un compte de stockage, Event Hub et Log Analytics. Vous pouvez également les analyser dans Power BI ou créer des alertes pour rester à jour sur les événements de ressource.

[Affichez et récupérez les événements du journal d’activité Azure.](../azure-monitor/platform/activity-log.md#view-the-activity-log)

## <a name="ship-activity-logs-to-event-grid"></a>Acheminer les journaux d’activité vers Event Grid

Tandis que les journaux d’activité sont basés sur l’utilisateur, il existe un nouveau [Event Grid](../event-grid/index.yml) intégration avec App Service (version préliminaire) qui journalise les actions utilisateur et les événements automatisés. Avec Event Grid, vous pouvez configurer un gestionnaire pour réagir auxdits événements. Par exemple, utilisez la grille d’événement pour déclencher instantanément une fonction sans serveur afin d’exécuter une analyse d’image chaque fois qu’une nouvelle photo est ajoutée à un conteneur de stockage d’objets blob.

Vous pouvez également utiliser Event Grid avec Logic Apps pour traiter des données en tout lieu sans écrire de code. La grille d’événement connecte des sources de données et des gestionnaires d’événements. Par exemple, utilisez la grille d’événement pour déclencher instantanément une fonction sans serveur afin d’exécuter une analyse d’image chaque fois qu’une nouvelle photo est ajoutée à un conteneur de stockage d’objets blob.

[Affichez les propriétés et le schéma pour les événements Azure App Service.](../event-grid/event-schema-app-service.md)

## <a name="next-steps"></a><a name="nextsteps"></a>Étapes suivantes
* [Interrogation de journaux d’activité grâce à Azure Monitor](../azure-monitor/log-query/log-query-overview.md)
* [Surveillance des applications dans Azure App Service](web-sites-monitor.md)
* [Dépannage d’une application web dans le Service d’application Microsoft Azure à l’aide de Visual Studio](troubleshoot-dotnet-visual-studio.md)
* [Analyse des journaux d’activité d’application dans HDInsight (en anglais)](https://gallery.technet.microsoft.com/scriptcenter/Analyses-Windows-Azure-web-0b27d413)
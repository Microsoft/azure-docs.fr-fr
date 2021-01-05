---
title: Présentation d’Azure Scheduler
description: Créer, planifier et exécuter des travaux automatisés qui appellent des services à l’intérieur ou à l’extérieur d’Azure
services: scheduler
ms.service: scheduler
ms.suite: infrastructure-services
author: derek1ee
ms.author: deli
ms.reviewer: klam, estfan
ms.topic: conceptual
ms.date: 02/17/2020
ms.openlocfilehash: 66ec285554299214122a4093837d3506bf642b13
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92368126"
---
# <a name="what-is-azure-scheduler"></a>Présentation d’Azure Scheduler

> [!IMPORTANT]
> [Azure Logic Apps](../logic-apps/logic-apps-overview.md) remplace Azure Scheduler, qui est en phase de [mise hors service](../scheduler/migrate-from-scheduler-to-logic-apps.md#retire-date). Pour poursuivre les travaux que vous avez configurés dans Scheduler, veuillez [migrer vers Azure Logic Apps](../scheduler/migrate-from-scheduler-to-logic-apps.md) dès que possible. 
>
> Scheduler n’est plus disponible dans le portail Azure, mais l’[API REST](/rest/api/scheduler) et les [applets de commande Azure Scheduler PowerShell](scheduler-powershell-reference.md) restent disponibles pour vous permettre de gérer vos travaux et collections de travaux.

[Azure Scheduler](https://azure.microsoft.com/services/scheduler/) vous permet de créer des [travaux](../scheduler/scheduler-concepts-terms.md) qui s’exécutent dans le cloud en décrivant les actions de manière déclarative. Le service planifie et exécute ensuite ces actions automatiquement. Par exemple, vous pouvez appeler des services à l’intérieur et à l’extérieur d’Azure, notamment des points de terminaison HTTP ou HTTPS, et poster des messages sur des files d’attente Stockage Azure, ainsi que sur des files d’attente ou des rubriques Azure Service Bus. Vous pouvez exécuter des travaux immédiatement ou plus tard. Scheduler prend facilement en charge les [planifications complexes et la périodicité avancée](../scheduler/scheduler-advanced-complexity.md). Scheduler spécifie le moment où les travaux doivent être exécutés, conserve un historique des résultats des travaux pour que vous puissiez le consulter, et planifie l’exécution des charges de travail de manière prévisible et fiable.

D’autres fonctionnalités de planification Azure utilisent également Scheduler en arrière-plan, par exemple, [Azure WebJobs](../app-service/webjobs-create.md), qui est une fonctionnalité [Web Apps](https://azure.microsoft.com/services/app-service/web/) dans Azure App Service. Vous pouvez gérer la communication pour ces actions à l’aide de l’[API REST Scheduler](/rest/api/scheduler/).

Voici quelques scénarios où Scheduler peut vous aider :

* Exécuter des actions d’application récurrentes : par exemple, collecter périodiquement des données de Twitter dans un flux.

* Effectuer une maintenance quotidienne : par exemple, nettoyer quotidiennement les journaux d’activité, faire des sauvegardes et effectuer d’autres tâches de maintenance.

  Par exemple, en tant qu’administrateur, vous pouvez être amené à sauvegarder votre base de données à 1 h 00 tous les jours, pendant les neuf prochains mois.

Bien que vous puissiez utiliser Scheduler pour créer, gérer et exécuter des charges de travail planifiées, Scheduler n’héberge pas les charges de travail et n’exécute pas le code. Le service *appelle* uniquement les services ou le code hébergé ailleurs, par exemple, dans Azure, localement ou chez un autre fournisseur. Scheduler peut appeler via HTTP et HTTPS, une file d’attente de stockage, une file d’attente Service Bus ou une rubrique Service Bus.

Pour créer, planifier, gérer, mettre à jour ou supprimer des travaux et des [collections de travaux](../scheduler/scheduler-concepts-terms.md), vous pouvez utiliser du code, l’[API REST Scheduler](/rest/api/scheduler/) ou les [applets de commande PowerShell d’Azure Scheduler](scheduler-powershell-reference.md).

## <a name="next-steps"></a>Étapes suivantes

* [Concepts, terminologie et hiérarchie d’entités d’Azure Scheduler](scheduler-concepts-terms.md)
* [Plans et facturation pour Azure Scheduler](scheduler-plans-billing.md)
* [Créer des planifications complexes et une périodicité avancée avec Azure Scheluler](scheduler-advanced-complexity.md)
* [Informations de référence sur l’API REST d’Azure Scheluler](/rest/api/scheduler)
* [Informations de référence sur les applets de commande PowerShell d’Azure Scheluler](scheduler-powershell-reference.md)
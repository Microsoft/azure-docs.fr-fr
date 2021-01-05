---
title: Mettre à l’échelle Azure Cosmos DB selon une planification à l’aide du minuteur Azure Functions
description: Apprenez à mettre à l’échelle les modifications du débit dans Azure Cosmos DB à l’aide de PowerShell et d’Azure Functions.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 01/13/2020
ms.author: mjbrown
ms.openlocfilehash: c60f3fc6b4ce4a1aead273fedb81e39de697f576
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339255"
---
# <a name="scale-azure-cosmos-db-throughput-by-using-azure-functions-timer-trigger"></a>Mettre à l’échelle le débit Azure Cosmos DB à l’aide du déclencheur de minuteur Azure Functions
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Les performances d’un compte Azure Cosmos sont basées sur le débit provisionné exprimé en unités de requête par seconde (RU/s). Le provisionnement s’effectue avec une précision d’une seconde et est facturé en fonction de la valeur RU/s par heure la plus élevée. Ce modèle de capacité approvisionnée permet au service de fournir un débit prévisible et constant, une faible latence garantie et une haute disponibilité. La plupart des charges de travail de production utilisent ces fonctionnalités. Toutefois, dans les environnements de développement et de test où Azure Cosmos DB n’est utilisé que pendant les heures de travail, vous pouvez augmenter le débit le matin et le réduire à nouveau le soir après les heures de travail.

Vous pouvez définir le débit via les [modèles Azure Resource Manager](./templates-samples-sql.md), [Azure CLI](cli-samples.md) ou [PowerShell](powershell-samples.md), pour les comptes d’API Core (SQL), ou en utilisant les kits SDK Azure Cosmos DB spécifiques au langage. L’avantage de l’utilisation des modèles Resource Manager, d’Azure CLI ou de PowerShell tient au fait qu’ils prennent en charge toutes les API du modèle Azure Cosmos DB.

## <a name="throughput-scheduler-sample-project"></a>Exemple de projet de planificateur de débit

Pour simplifier le processus de mise à l’échelle d’Azure Cosmos DB selon une planification, nous avons créé un exemple de projet appelé [Planificateur de débit Cosmos Azure](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler). Ce projet est une application Azure Functions avec deux déclencheurs de minuteur : « ScaleUpTrigger » et « ScaleDownTrigger ». Les déclencheurs exécutent un script PowerShell qui définit le débit sur chaque ressource tel qu’il est défini dans le fichier `resources.json` de chaque déclencheur. ScaleUpTrigger est configuré pour s’exécuter à 8 h UTC et ScaleDownTrigger est configuré pour s’exécuter à 6 h UTC, et ces heures peuvent être facilement mises à jour dans le fichier `function.json` pour chaque déclencheur.

Vous pouvez cloner ce projet localement et le modifier pour spécifier les ressources Azure Cosmos DB à augmenter ou réduire, ainsi que la planification à exécuter. Par la suite, vous pouvez le déployer dans un abonnement Azure et le sécuriser en utilisant Managed Service Identity avec des autorisations de [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](role-based-access-control.md) avec le rôle « Opérateur Azure Cosmos DB » pour définir le débit sur vos comptes Azure Cosmos.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus et télécharger l’exemple à partir de [Planificateur de débit Azure Cosmos DB](https://github.com/Azure-Samples/azure-cosmos-throughput-scheduler).
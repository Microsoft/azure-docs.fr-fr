---
title: Opérations du plan de contrôle et du plan de données
description: Décrit la différence entre les opérations du plan de contrôle et du plan de données. Les opérations du plan de contrôle sont gérées par Azure Resource Manager. Les opérations de plan de données sont gérées par un service.
ms.topic: conceptual
ms.date: 09/10/2020
ms.openlocfilehash: f478037c3e83c1d28cc900d64512a41619628dd7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91371267"
---
# <a name="azure-control-plane-and-data-plane"></a>Plan de contrôle et plan de données Azure

Les opérations Azure peuvent être divisées en deux catégories : le plan de contrôle et le plan de données. Cet article décrit les différences entre ces deux types d’opérations.

Vous utilisez le plan de contrôle pour gérer les ressources de votre abonnement. Le plan de données vous permet d’utiliser les fonctionnalités exposées par votre instance d’un type de ressource.

Par exemple :

* Vous créez une machine virtuelle via le plan de contrôle. Une fois la machine virtuelle créée, vous interagissez avec elle via des opérations de plan de données, comme le protocole RDP (Remote Desktop Protocol).

* Vous créez un compte de stockage via le plan de contrôle. Le plan de données vous permet de lire et d’écrire des données dans le compte de stockage.

* Vous créez une base de données Cosmos via le plan de contrôle. Pour interroger des données dans la base de données Cosmos, vous utilisez le plan de données.

## <a name="control-plane"></a>Plan de contrôle

Toutes les requêtes d’opérations de plan de contrôle sont envoyées à l’URL d’Azure Resource Manager. Cette URL varie en fonction de l’environnement Azure.

* Pour Azure Global, l’URL est `https://management.azure.com`.
* Pour Azure Government, l’URL est `https://management.usgovcloudapi.net/`.
* Pour Azure Allemagne, l’URL est `https://management.microsoftazure.de/`.
* Pour Microsoft Azure China 21Vianet, l’URL est `https://management.chinacloudapi.cn`.

Pour découvrir quelles opérations utilisent l’URL Azure Resource Manager, consultez [l’API REST Azure](/rest/api/azure/). Par exemple, [l’opération de création ou de mise à jour](/rest/api/mysql/databases/createorupdate) pour MySQL est une opération de plan de contrôle, car l’URL de la demande est :

```http
PUT https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroupName}/providers/Microsoft.DBforMySQL/servers/{serverName}/databases/{databaseName}?api-version=2017-12-01
```

Azure Resource Manager gère toutes les requêtes du plan de contrôle. Il applique automatiquement les fonctionnalités Azure que vous avez implémentées pour gérer vos ressources, comme :

* [Contrôle d’accès en fonction du rôle Azure (RBAC Azure)](../../role-based-access-control/overview.md)
* [Azure Policy](../../governance/policy/overview.md)
* [Verrous de gestion](lock-resources.md)
* [Journaux d’activité](view-activity-logs.md)

Une fois la demande authentifiée, Azure Resource Manager l’envoie au fournisseur de ressources, qui termine l’opération.

Le plan de contrôle comprend deux scénarios pour la gestion des requêtes : « Champ vert » et « Champ marron ». Le champ vert fait référence aux nouvelles ressources. Le champ marron fait référence aux ressources existantes. Lorsque vous déployez des ressources, Azure Resource Manager comprend quand créer de nouvelles ressources et quand mettre à jour les ressources existantes. Vous n’avez pas à vous soucier de la création de ressources identiques.

## <a name="data-plane"></a>Plan de données

Les demandes d’opérations du plan de données sont envoyées à un point de terminaison spécifique à votre instance. Par exemple, [Détecter l’opération de langage](/rest/api/cognitiveservices/textanalytics/detect%20language/detect%20language) dans Cognitive Services est une opération du plan de données, car l’URL de la demande est :

```http
POST {Endpoint}/text/analytics/v2.0/languages
```

Les opérations du plan de données ne sont pas limitées à l’API REST. Elles peuvent nécessiter des informations d’identification supplémentaires, comme la connexion à une machine virtuelle ou un serveur de base de données.

Les fonctionnalités qui appliquent la gestion et la gouvernance peuvent ne pas être appliquées aux opérations du plan de données. Vous devez prendre en compte les différentes façons dont les utilisateurs interagissent avec vos solutions. Par exemple, un verrou qui empêche les utilisateurs de supprimer une base de données n’empêche pas les utilisateurs de supprimer des données via des requêtes.

Vous pouvez utiliser certaines stratégies pour régir les opérations du plan de données. Pour plus d’informations, consultez [Modes Fournisseur de ressources (préversion) dans Azure Policy](../../governance/policy/concepts/definition-structure.md#resource-provider-modes).

## <a name="next-steps"></a>Étapes suivantes

* Pour une présentation d’Azure Resource Manager, consultez [Qu’est-ce qu’Azure Resource Manager ?](overview.md)

* Pour en savoir plus sur l’effet des définitions de stratégie sur les nouvelles ressources et les ressources existantes, consultez [Évaluer l’impact d’une nouvelle définition Azure Policy](../../governance/policy/concepts/evaluate-impact.md).

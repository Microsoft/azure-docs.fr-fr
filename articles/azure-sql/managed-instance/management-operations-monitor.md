---
title: Surveiller les opérations de gestion
titleSuffix: Azure SQL Managed Instance
description: Découvrez différentes méthodes de surveillance des opérations de gestion d'Azure SQL Managed Instance.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: ''
ms.topic: how-to
author: urosmil
ms.author: urmilano
ms.reviewer: sstein, bonova, MashaMSFT
ms.date: 09/03/2020
ms.openlocfilehash: 0f76d2079b7ed5aacbf835540ea92febd034e2d0
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92782448"
---
# <a name="monitoring-azure-sql-managed-instance-management-operations"></a>Surveiller les opérations de gestion d'Azure SQL Managed Instance
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Azure SQL Managed Instance permet de surveiller les [opérations de gestion](management-operations-overview.md) auxquelles vous avez recours pour déployer de nouvelles instances gérées, mettre à jour les propriétés des instances ou supprimer les instances dont vous n'avez plus besoin. 

## <a name="overview"></a>Vue d’ensemble

Toutes les opérations de gestion peuvent être classées comme suit :

- Déploiement d’instance (création d’une nouvelle instance)
- Mise à jour d’instance (modification des propriétés, par exemple, vCore ou stockage réservé)
- Suppression d’instance

La plupart des opérations de gestion sont des [opérations durables](management-operations-overview.md#duration). Il est donc nécessaire de surveiller leur état ou de suivre la progression des différentes étapes qui les composent. 

Différentes méthodes sont disponibles pour surveiller les opérations de gestion des instances gérées :

- [Déploiements de groupes de ressources](../../azure-resource-manager/templates/deployment-history.md)
- [Journal d’activité](../../azure-monitor/platform/activity-log.md)
- [API Opérations d'instance gérée](#managed-instance-operations-api)


Le tableau suivant compare les options de surveillance des opérations de gestion : 

| Option | Rétention | Prend en charge l'annulation | Créer | Update | Supprimer | Annuler | Étapes |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Déploiements de groupes de ressources | Infinis<sup>1</sup> | Non <sup>2</sup> | Visible | Visible | Non visible | Visible | Non visible |
| Journal d’activité | 90 jours | Non | Visible | Visible | Visible | Visible |  Non visible |
| API Opérations d'instance gérée | 24 heures | [Oui](management-operations-cancel.md) | Visible | Visible | Visible | Visible | Visible |
|  |  |  |  |  |  |  | |

<sup>1</sup> L'historique de déploiement de chaque groupe de ressources est limité à 800 déploiements.

<sup>2</sup> Les déploiements de groupes de ressources prennent en charge l'opération d'annulation. Toutefois, compte tenu de la logique d'annulation, seule une opération dont le déploiement est prévu après l'exécution de l'action d'annulation sera annulée. Un déploiement en cours ne sera pas annulé au moment de l'annulation du déploiement du groupe de ressources. Dans la mesure où le déploiement d'une instance gérée se compose d'une étape « durable » (du point de vue d'Azure Resource Manager), l'annulation du déploiement du groupe de ressources n'annulera pas celui de l'instance gérée et l'opération se poursuivra jusqu'à son terme. 

## <a name="managed-instance-operations-api"></a>API Opérations d'instance gérée

Les API Opérations de gestion sont spécialement conçues pour surveiller les opérations. La surveillance des opérations d'instance gérée peut fournir des informations sur les paramètres et les étapes des opérations, ainsi que sur l'[annulation d'opérations spécifiques](management-operations-cancel.md). Outre les détails de l'opération et la commande d'annulation qu'elle fournit, cette API peut être utilisée dans des scripts d'automatisation avec des déploiements multi-ressources - en fonction de l'étape de progression, vous pouvez lancer le déploiement de certaines ressources dépendantes.

Les API disponibles sont les suivantes : 

| Commande | Description |
| --- | --- |
|[Opérations d'instance gérée - Obtenir](/rest/api/sql/managedinstanceoperations/get)|Permet d'obtenir une opération de gestion sur une instance gérée.|
|[Opérations d'instance gérée - Annuler](/rest/api/sql/managedinstanceoperations/cancel)|Permet d'annuler l'opération asynchrone sur l'instance gérée.|
|[Opérations d'instance gérée - Répertorier par instance gérée](/rest/api/sql/managedinstanceoperations/listbymanagedinstance)|Permet d'obtenir la liste des opérations effectuées sur l'instance gérée.|

> [!NOTE]
> Utilisez la version 2020-02-02 de l'API pour voir l'opération de création de l'instance gérée dans la liste des opérations. Il s'agit de la version par défaut utilisée sur le portail Azure, et elle contient les derniers packages PowerShell et Azure CLI.

## <a name="monitor-operations"></a>Surveiller les opérations

# <a name="portal"></a>[Portail](#tab/azure-portal)

Sur le portail Azure, utilisez la page de **présentation** de l'instance gérée pour surveiller les opérations d'instance gérée. 

Par exemple, l' **opération Créer** est visible au début du processus de création sur la page de **présentation**  : 

![Progression de la création d'une instance gérée](./media/management-operations-monitor/monitoring-create-operation.png)

Sélectionnez **Opération en cours** pour ouvrir la page **Opération en cours** et accéder aux opérations **Créer** ou **Mettre à jour** . Vous pouvez également [Annuler](management-operations-cancel.md) des opérations à partir de cette page.  

![Détails des opérations d'instance gérée](./media/management-operations-monitor/monitoring-operation-details.png)

> [!NOTE]
> Les opérations de création soumises via le portail Azure, PowerShell, Azure CLI ou d'autres outils à l'aide de l'API REST version 2020-02-02 [peuvent être annulées](management-operations-cancel.md). Les versions de l'API REST antérieures à la version 2020-02-02 utilisées pour soumettre une opération de création lanceront le déploiement de l'instance, mais celui-ci ne sera pas répertorié dans l'API Opérations et ne pourra donc pas être annulé.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

La cmdlet Get-AzSqlInstanceOperation permet d'obtenir des informations sur les opérations effectuées sur une instance gérée. Vous pouvez afficher toutes les opérations effectuées sur une instance gérée ou afficher une opération spécifique en fournissant son nom.

```powershell-interactive
$managedInstance = "yourInstanceName"
$resourceGroup = "yourResourceGroupName"

$managementOperations = Get-AzSqlInstanceOperation `
    -ManagedInstanceName $managedInstance  -ResourceGroupName $resourceGroup
```

Pour une description détaillée des commandes, consultez [Get-AzSqlInstanceOperation](/powershell/module/az.sql/get-azsqlinstanceoperation).

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

az sql mi op list permet d'obtenir la liste des opérations effectuées sur l'instance gérée. Si Azure CLI n’est pas encore installé, consultez [Installer l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).

```azurecli-interactive
az sql mi op list -g yourResourceGroupName --mi yourInstanceName 
```

Pour une explication détaillée des commandes, consultez [az sql mi op](/cli/azure/sql/mi/op).

---

## <a name="next-steps"></a>Étapes suivantes

- Pour savoir comment créer votre première instance managée, consultez le [Guide de démarrage rapide](instance-create-quickstart.md).
- Pour accéder à la liste des fonctionnalités et à leur comparaison, consultez [Fonctionnalités SQL courantes](../database/features-comparison.md).
- Pour plus d’informations sur la configuration du réseau virtuel, consultez [Configuration de réseau virtuel SQL Managed Instance](connectivity-architecture-overview.md).
- Pour obtenir un guide de démarrage rapide qui crée une instance managée et restaure une base de données à partir d’un fichier de sauvegarde, consultez [Créer une instance managée](instance-create-quickstart.md).
- Pour accéder à un tutoriel expliquant comment utiliser Azure Database Migration Service pour la migration, consultez [Migration SQL Managed Instance à l’aide d’Azure Database Migration Service](../../dms/tutorial-sql-server-to-managed-instance.md).
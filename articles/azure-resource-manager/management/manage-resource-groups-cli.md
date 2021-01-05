---
title: Gérer des groupes de ressources - Azure CLI
description: Utilisez Azure CLI pour gérer vos groupes de ressources dans Azure Resource Manager. Montre comment créer, lister et supprimer des groupes de ressources.
author: mumian
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: jgao
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4a9a4ed4ebba7f6f2470bb9e7000a899ebc26323
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185808"
---
# <a name="manage-azure-resource-manager-resource-groups-by-using-azure-cli"></a>Gérer des groupes de ressources Azure Resource Manager à l’aide d’Azure CLI

Découvrez comment utiliser Azure CLI avec [Azure Resource Manager](overview.md) pour gérer vos groupes de ressources Azure. Pour gérer des ressources Azure, voir [Gérer les ressources Azure à l’aide d’Azure CLI](manage-resources-cli.md).

Autres articles sur la gestion des groupes de ressources :

- [Gérer les groupes de ressources Azure à l’aide du portail Azure](manage-resources-portal.md)
- [Gérer les groupes de ressources Azure à l’aide d’Azure PowerShell](manage-resources-powershell.md)

## <a name="what-is-a-resource-group"></a>Qu’est-ce qu’un groupe de ressources

Un groupe de ressources est un conteneur réunissant les ressources associées d’une solution Azure. Le groupe de ressources peut inclure toutes les ressources de la solution, ou uniquement celles que vous souhaitez gérer en tant que groupe. Pour déterminer comment allouer des ressources aux groupes de ressources, choisissez l’approche la plus pertinente pour votre organisation. En règle générale, il convient d’ajouter des ressources qui partagent le même cycle de vie dans un même groupe de ressources afin de pouvoir facilement les déployer, les mettre à jour et les supprimer en tant que groupe.

Le groupe de ressources stocke des métadonnées sur les ressources. Par conséquent, lorsque vous spécifiez un emplacement pour le groupe de ressources, vous indiquez où stocker ces métadonnées. Pour des raisons de conformité, vous devrez peut-être vous assurer que vos données sont stockées dans une région particulière.

Le groupe de ressources stocke des métadonnées sur les ressources. Lorsque vous spécifiez un emplacement pour le groupe de ressources, vous indiquez où stocker ces métadonnées.

## <a name="create-resource-groups"></a>Créer des groupes de ressources

La commande CLI suivante crée un groupe de ressources.

```azurecli-interactive
az group create --name demoResourceGroup --location westus
```

## <a name="list-resource-groups"></a>Répertorier les groupes de ressources

Le script CLI suivant répertorie les groupes de ressources de votre abonnement.

```azurecli-interactive
az group list
```

Pour obtenir un groupe de ressources :

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group show --name $resourceGroupName
```

## <a name="delete-resource-groups"></a>Supprimer des groupes de ressources

Le script CLI suivant supprime un groupe de ressources :

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName
```

Pour plus d’informations sur l’ordre dans lequel Azure Resource Manager supprime des ressources, voir [Suppression d’un groupe de ressources par Azure Resource Manager](delete-resource-group.md).

## <a name="deploy-resources-to-an-existing-resource-group"></a>Déployer des ressources sur un groupe de ressources existant

Voir [Déployer des ressources sur un groupe de ressources existant](manage-resources-cli.md#deploy-resources-to-an-existing-resource-group).

## <a name="deploy-a-resource-group-and-resources"></a>Déployer un groupe de ressources et des ressources

Vous pouvez créer un groupe de ressources et déployer des ressources sur ce groupe à l'aide d'un modèle Resource Manager. Pour plus d'informations, consultez [Créer un groupe de ressources et déployer des ressources](../templates/deploy-to-subscription.md#resource-groups).

## <a name="redeploy-when-deployment-fails"></a>Redéploiement en cas d’échec du déploiement

Cette fonctionnalité est également appelée *Annulation en cas d’erreur*. Pour plus d'informations, consultez [Redéploiement en cas d'échec du déploiement](../templates/rollback-on-error.md).

## <a name="move-to-another-resource-group-or-subscription"></a>Déplacer vers un autre groupe de ressources ou abonnement

Vous pouvez déplacer les ressources du groupe vers un autre groupe de ressources. Pour plus d’informations, voir [Déplacer des ressources](manage-resources-cli.md#move-resources).

## <a name="lock-resource-groups"></a>Verrouiller des groupes de ressources

Le verrouillage empêche d’autres utilisateurs de votre organisation de supprimer ou de modifier accidentellement des ressources stratégiques, telles qu’une ressource, un groupe de ressources ou un abonnement Azure. 

Le script suivant verrouille un groupe de ressources pour empêcher sa suppression.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az lock create --name LockGroup --lock-type CanNotDelete --resource-group $resourceGroupName  
```

Le script suivant obtient tous les verrous relatifs à un groupe de ressources :

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az lock list --resource-group $resourceGroupName  
```

Le script suivant supprime un verrou :

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
echo "Enter the lock name:" &&
read lockName &&
az lock delete --name $lockName --resource-group $resourceGroupName
```

Pour plus d’informations, consultez [Verrouiller des ressources avec Azure Resource Manager](lock-resources.md).

## <a name="tag-resource-groups"></a>Baliser des groupes de ressources

Vous pouvez appliquer des balises à des groupes de ressources pour organiser logiquement vos ressources. Pour plus d’informations, voir [Organisation des ressources Azure à l’aide de balises](tag-resources.md#azure-cli).

## <a name="export-resource-groups-to-templates"></a>Exporter des groupes de ressources dans des modèles

Après avoir correctement configuré votre groupe de ressources, vous pouvez afficher le modèle Resource Manager correspondant. L’exportation du modèle offre deux avantages :

- Automatisez les futurs déploiements de la solution, car le modèle contient la totalité de l'infrastructure.
- Familiarisez-vous avec la syntaxe des modèles en consultant la notation JSON (JavaScript Object Notation) qui représente votre solution.

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group export --name $resourceGroupName  
```

Le script affiche le modèle sur la console.  Copiez le JSON et enregistrez-le dans un fichier.

La fonctionnalité d’exportation de modèle ne prend pas en charge l’exportation des ressources Azure Data Factory. Pour en savoir plus sur la façon dont vous pouvez exporter des ressources Data Factory, consultez [Copier ou cloner une fabrique de données dans Azure Data Factory](../../data-factory/copy-clone-data-factory.md).

Pour exporter des ressources créées par le biais du modèle de déploiement classique, vous devez [les migrer vers le modèle de déploiement de Resource Manager](../../virtual-machines/migration-classic-resource-manager-overview.md).

Pour plus d’informations, consultez [Export d’une ressource unique ou de plusieurs ressources vers un modèle dans le portail Azure](../templates/export-template-portal.md).

## <a name="manage-access-to-resource-groups"></a>Gérer l'accès aux groupes de ressources

Le [contrôle d’accès en fonction du rôle Azure (Azure RBAC)](../../role-based-access-control/overview.md) vous permet de gérer l’accès aux ressources dans Azure. Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôle Azure à l’aide d’Azure CLI](../../role-based-access-control/role-assignments-cli.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour vous familiariser avec Azure Resource Manager, consultez [Vue d’ensemble d’Azure Resource Manager](overview.md).
- Pour vous familiariser avec la syntaxe des modèles Resource Manager, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../templates/template-syntax.md).
- Pour apprendre à développer des modèles, consultez les [tutoriels pas à pas](../index.yml).
- Pour accéder aux schémas liés aux modèles Azure Resource Manager, consultez [Informations de référence sur les modèles](/azure/templates/).
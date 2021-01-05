---
title: Supprimer des ressources d’Azure
description: Supprimer des ressources d’Azure
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: twright-msft
ms.author: twright
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 4c76b2c0109637ce34681d2fa5c8b29e1ff800a1
ms.sourcegitcommit: 28c5fdc3828316f45f7c20fc4de4b2c05a1c5548
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/22/2020
ms.locfileid: "92364369"
---
# <a name="delete-resources-from-azure"></a>Supprimer des ressources d’Azure

> [!NOTE]
>  Les options permettant de supprimer des ressources décrites dans cet article sont irréversibles.

> [!NOTE]
>  Étant donné que le seul mode de connexion proposé pour les services de données activés par Azure Arc est actuellement le mode de connexion indirecte, la suppression d’une instance de Kubernetes n’a pas pour effet de supprimer celle-ci d’Azure, et la suppression d’une instance d’Azure a pour effet de supprimer celle-ci de Kubernetes.  Actuellement, la suppression d’une ressource est un processus en deux étapes qui sera amélioré dans le futur.  À l’avenir, Kubernetes sera la source fidèle et Azure sera mis à jour pour la refléter.

Dans certains cas, il se peut que vous deviez supprimer manuellement les ressources des services de données activés par Azure Arc dans Azure Resource Manager (ARM).  Vous pouvez supprimer ces ressources à l’aide de l’une des options suivantes.

- [Supprimer des ressources d’Azure](#delete-resources-from-azure)
  - [Supprimer un groupe de ressources entier](#delete-an-entire-resource-group)
  - [Supprimer des ressources spécifiques dans le groupe de ressources](#delete-specific-resources-in-the-resource-group)
  - [Supprimer des ressources à l’aide d’Azure CLI](#delete-resources-using-the-azure-cli)
    - [Supprimer des ressources d’instance gérée SQL à l’aide d’Azure CLI](#delete-sql-managed-instance-resources-using-the-azure-cli)
    - [Supprimer des ressources de groupe de serveurs PostgreSQL Hyperscale à l’aide d’Azure CLI](#delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli)
    - [Supprimer des ressources de contrôleur de données Azure Arc à l’aide d’Azure CLI](#delete-azure-arc-data-controller-resources-using-the-azure-cli)
    - [Supprimer un groupe de ressources à l’aide d’Azure CLI](#delete-a-resource-group-using-the-azure-cli)

[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="delete-an-entire-resource-group"></a>Supprimer un groupe de ressources entier
Si vous avez utilisé un groupe de ressources spécifique et dédié pour les services de données activés par Azure Arc et souhaitez supprimer *tout* le contenu du groupe de ressources, vous pouvez supprimer le groupe de ressources, ce qui a pour effet de supprimer tout son contenu.  

Vous pouvez supprimer un groupe de ressources dans le portail Azure en procédant comme suit :

- Accédez au groupe de ressources dans le portail Azure où les ressources de services de données activées par Azure Arc ont été créées.
- Cliquez sur le bouton **Supprimer le groupe de ressources** .
- Validez la suppression en entrant le nom du groupe de ressources, puis cliquez sur **Supprimer** .

## <a name="delete-specific-resources-in-the-resource-group"></a>Supprimer des ressources spécifiques dans le groupe de ressources

Vous pouvez supprimer des ressources de services de données activées par Azure Arc dans un groupe de ressources sur le portail Azure en procédant comme suit :

- Accédez au groupe de ressources dans le portail Azure où les ressources de services de données activées par Azure Arc ont été créées.
- Sélectionnez toutes les ressources à supprimer.
- Cliquez sur le bouton Supprimer.
- Pour confirmer la suppression, tapez Oui, puis cliquez sur **Supprimer** .

## <a name="delete-resources-using-the-azure-cli"></a>Supprimer des ressources à l’aide d’Azure CLI

Vous pouvez supprimer des ressources de services de données activées par Azure Arc à l’aide d’Azure CLI.

### <a name="delete-sql-managed-instance-resources-using-the-azure-cli"></a>Supprimer des ressources d’instance gérée SQL à l’aide d’Azure CLI

Pour supprimer des ressources d’instance gérée SQL d’Azure à l’aide d’Azure CLI remplacez les valeurs d’espace réservé dans la commande ci-dessous, puis exécutez celle-ci.

```azurecli
az resource delete --name <sql instance name> --resource-type Microsoft.AzureData/sqlManagedInstances --resource-group <resource group name>

#Example
#az resource delete --name sql1 --resource-type Microsoft.AzureData/sqlManagedInstances --resource-group rg1
```

### <a name="delete-postgresql-hyperscale-server-group-resources-using-the-azure-cli"></a>Supprimer des ressources de groupe de serveurs PostgreSQL Hyperscale à l’aide d’Azure CLI

Pour supprimer une ressource de groupe de serveurs PostgreSQL Hyperscale d’Azure à l’aide d’Azure CLI, remplacez les valeurs d’espace réservé dans la commande ci-dessous, puis exécutez-la.

```azurecli
az resource delete --name <postgresql instance name> --resource-type Microsoft.AzureData/postgresInstances --resource-group <resource group name>

#Example
#az resource delete --name pg1 --resource-type Microsoft.AzureData/postgresInstances --resource-group rg1
```

### <a name="delete-azure-arc-data-controller-resources-using-the-azure-cli"></a>Supprimer des ressources de contrôleur de données Azure Arc à l’aide d’Azure CLI

> [!NOTE]
> Avant de supprimer un contrôleur de données Azure Arc, vous devez supprimer toutes les ressources de l’instance de base de données qu’il gère.

Pour supprimer un contrôleur de données Azure Arc d’Azure à l’aide d’Azure CLI, remplacez les valeurs d’espace réservé dans la commande ci-dessous, puis exécutez-la.

```azurecli
az resource delete --name <data controller name> --resource-type Microsoft.AzureData/dataControllers --resource-group <resource group name>

#Example
#az resource delete --name dc1 --resource-type Microsoft.AzureData/dataControllers --resource-group rg1
```

### <a name="delete-a-resource-group-using-the-azure-cli"></a>Supprimer un groupe de ressources à l’aide Azure CLI

Pour [supprimer un groupe de ressources](../../azure-resource-manager/management/delete-resource-group.md), vous pouvez également utiliser Azure CLI.
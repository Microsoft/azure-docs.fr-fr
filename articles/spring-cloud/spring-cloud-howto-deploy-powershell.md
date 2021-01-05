---
title: Guide pratique pour déployer Azure Spring Cloud avec Azure PowerShell
description: Guide pratique pour déployer Azure Spring Cloud avec Azure PowerShell
author: bmitchell287
ms.author: brendm
ms.topic: conceptual
ms.service: spring-cloud
ms.devlang: azurepowershell
ms.date: 11/16/2020
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 3cb320a37818084f2fbcad22a3cc992655b19c3d
ms.sourcegitcommit: c95e2d89a5a3cf5e2983ffcc206f056a7992df7d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95547884"
---
# <a name="deploy-azure-spring-cloud-with-azure-powershell"></a>Déployer Azure Spring Cloud avec Azure PowerShell

Cet article explique comment vous pouvez créer une instance d’Azure Spring Cloud à l’aide du module PowerShell [Az.SpringCloud](/powershell/module/Az.SpringCloud).

## <a name="requirements"></a>Spécifications

* Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [azure-powershell-requirements-no-header.md](../../includes/azure-powershell-requirements-no-header.md)]

  > [!IMPORTANT]
  > Tant que le module PowerShell **Az.SpringCloud** est en préversion, vous devez l’installer séparément au moyen de l’applet de commande `Install-Module`. Une fois que ce module PowerShell sera en disponibilité générale, il fera partie intégrante des versions futures du module Az PowerShell et sera disponible par défaut dans Azure Cloud Shell.

  ```azurepowershell-interactive
  Install-Module -Name Az.SpringCloud
  ```

* Si vous avez plusieurs abonnements Azure, sélectionnez l’abonnement approprié dans lequel les ressources doivent être facturées. Sélectionnez un abonnement spécifique avec l’applet de commande [Set-AzContext](/powershell/module/az.accounts/set-azcontext).

  ```azurepowershell-interactive
  Set-AzContext -SubscriptionId 00000000-0000-0000-0000-000000000000
  ```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec l’applet de commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources avec un nom spécifié et à l’emplacement indiqué.

```azurepowershell-interactive
New-AzResourceGroup -Name <resource group name> -Location eastus
```

## <a name="provision-a-new-instance-of-azure-spring-cloud"></a>Provisionner une nouvelle instance d’Azure Spring Cloud

Pour créer une instance d’Azure Spring Cloud, vous utilisez l’applet de commande [New-AzSpringCloud](/powershell/module/az.springcloud/new-azspringcloud). L’exemple suivant crée un service Azure Spring Cloud avec le nom précisé, dans le groupe de ressources créé précédemment.

```azurepowershell-interactive
New-AzSpringCloud -ResourceGroupName <resource group name> -name <service instance name> -Location eastus
```

## <a name="create-a-new-azure-spring-cloud-app"></a>Créer une application Azure Spring Cloud

Pour créer une application, utilisez l’applet de commande [New-AzSpringCloudApp](/powershell/module/az.springcloud/new-azspringcloudapp). L’exemple suivant crée une application Azure Spring Cloud nommée `gateway`.

```azurepowershell-interactive
New-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="create-a-new-azure-spring-cloud-deployment"></a>Créer un déploiement Azure Spring Cloud

Pour créer un déploiement, utilisez l’applet de commande [New-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/new-azspringcloudappdeployment). L’exemple suivant crée un déploiement d’Azure Spring Cloud nommé `default` pour l’application `gateway`.

```azurepowershell-interactive
New-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -Name <service instance name> -AppName gateway -DeploymentName default
```

## <a name="get-an-azure-spring-cloud-service"></a>Obtenir un service Azure Spring Cloud

Pour bénéficier d’un service Azure Spring Cloud et de ses propriétés, utilisez l’applet de commande [Get-AzSpringCloud](/powershell/module/az.springcloud/get-azspringcloud). L’exemple suivant extrait les informations relatives au service Azure Spring Cloud spécifié.

```azurepowershell-interactive
Get-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

## <a name="get-an-azure-spring-cloud-app"></a>Obtenir une application Azure Spring Cloud

Pour bénéficier d’une application Azure Spring Cloud et de ses propriétés, utilisez l’applet de commande [Get-AzSpringCloudApp](/powershell/module/az.springcloud/get-azspringcloudapp). L’exemple suivant extrait les informations sur l’application Spring Cloud `gateway`.

```azurepowershell-interactive
Get-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

## <a name="get-an-azure-spring-cloud-deployment"></a>Obtenir un déploiement d’Azure Spring Cloud

Pour bénéficier d’un déploiement Azure Spring Cloud et de ses propriétés, utilisez l’applet de commande [Get-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/get-azspringcloudappdeployment). L’exemple suivant récupère des informations sur le déploiement de Spring Cloud `default`.

```azurepowershell-interactive
Get-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez créées dans cet article, vous pouvez les supprimer en exécutant l’exemple suivant.

### <a name="delete-an-azure-spring-cloud-deployment"></a>Supprimer un déploiement d’Azure Spring Cloud

Pour supprimer un déploiement d’Azure Spring Cloud, utilisez l’applet de commande [Remove-AzSpringCloudAppDeployment](/powershell/module/az.springcloud/remove-azspringcloudappdeployment). L’exemple suivant supprime un déploiement d’application Azure Spring Cloud nommé `default` pour le service et l’application indiqués.

```azurepowershell-interactive
Remove-AzSpringCloudAppDeployment -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway -DeploymentName default
```

### <a name="delete-an-azure-spring-cloud-app"></a>Supprimer une application Azure Spring Cloud

Pour supprimer une application Spring Cloud, vous utilisez l’applet de commande [Remove-AzSpringCloudApp](/powershell/module/Az.SpringCloud/remove-azspringcloudapp). L’exemple suivant supprime l’application `gateway` dans le groupe de ressources et le service indiqués.

```azurepowershell
Remove-AzSpringCloudApp -ResourceGroupName <resource group name> -ServiceName <service instance name> -AppName gateway
```

### <a name="delete-an-azure-spring-cloud-service"></a>Supprimer un service Azure Spring Cloud

Pour supprimer un service Azure Spring Cloud, utilisez l’applet de commande [Remove-AzSpringCloud](/powershell/module/Az.SpringCloud/remove-azspringcloud). L’exemple suivant supprime le service Azure Spring Cloud spécifié.

```azurepowershell
Remove-AzSpringCloud -ResourceGroupName <resource group name> -ServiceName <service instance name>
```

### <a name="delete-the-resource-group"></a>Supprimer le groupe de ressources

> [!CAUTION]
> L’exemple suivant supprime le groupe de ressources spécifié et toutes les ressources qu’il contient.
> Si des ressources en dehors du cadre de cet article existent dans le groupe de ressources spécifié, elles seront également supprimées.

```azurepowershell-interactive
Remove-AzResourceGroup -Name <resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

[Ressources du développeur Azure Spring Cloud](spring-cloud-resources.md).

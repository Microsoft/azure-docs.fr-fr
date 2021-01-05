---
title: Créer un espace de noms Azure Service Bus à l’aide d’un modèle
description: Utiliser un modèle Azure Resource Manager pour créer un espace de noms Service Bus Messaging
documentationcenter: .net
author: spelluru
ms.topic: article
ms.tgt_pltfrm: dotnet
ms.date: 06/23/2020
ms.author: spelluru
ms.openlocfilehash: 1b7aafca331170100ce99c084a11c96c97df7781
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88067390"
---
# <a name="create-a-service-bus-namespace-by-using-an-azure-resource-manager-template"></a>Créer un espace de noms Service Bus à l’aide d’un modèle Azure Resource Manager

Découvrez comment déployer un modèle Azure Resource Manager pour créer un espace de noms Service Bus. Vous pouvez utiliser ce modèle pour vos propres déploiements, ou le personnaliser afin qu’il réponde à vos besoins. Pour en savoir plus sur la création de modèles, consultez la [documentation Azure Resource Manager](../azure-resource-manager/index.yml).

Les modèles suivants sont également disponibles pour la création d’espaces de noms Service Bus :

* [Créer un espace de noms Service Bus avec file d’attente](./service-bus-resource-manager-namespace-queue.md)
* [Créer un espace de noms Service Bus par rubrique et abonnement](./service-bus-resource-manager-namespace-topic.md)
* [Créer un espace de noms Service Bus avec file d'attente et règle d’autorisation](./service-bus-resource-manager-namespace-auth-rule.md)
* [Créer un modèle d’espace de noms Service Bus avec rubrique, abonnement et règle](./service-bus-resource-manager-namespace-topic-with-rule.md)

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-a-service-bus-namespace"></a>Création d'un espace de noms Service Bus

Dans ce guide de démarrage rapide, vous utilisez un [modèle Resource Manager existant](https://github.com/Azure/azure-quickstart-templates/blob/master/101-servicebus-create-namespace/azuredeploy.json) issu de [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/) :

[!code-json[create-azure-service-bus-namespace](~/quickstart-templates/101-servicebus-create-namespace/azuredeploy.json)]

Pour obtenir d’autres exemples de modèles, voir [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Servicebus&pageNumber=1&sort=Popular).

Pour créer un espace de noms Service Bus à l’aide d’un modèle :

1. Sélectionnez **Essayer** à partir du bloc de code suivant, puis suivez les instructions permettant de vous connecter à Azure Cloud Shell.

    ```azurepowershell-interactive
    $serviceBusNamespaceName = Read-Host -Prompt "Enter a name for the service bus namespace to be created"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${serviceBusNamespaceName}rg"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-servicebus-create-namespace/azuredeploy.json"

    New-AzResourceGroup -Name $resourceGroupName -Location $location
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri -serviceBusNamespaceName $serviceBusNamespaceName

    Write-Host "Press [ENTER] to continue ..."
    ```

    Le nom du groupe de ressources correspond au nom du Service Bus auquel **rg** a été ajouté.

2. Sélectionnez **Copier** pour copier le script PowerShell.
3. Cliquez avec le bouton droit sur la console d’interpréteur de commandes, puis sélectionnez **Coller**.

Il faut quelques instants pour créer un Event Hub.

## <a name="verify-the-deployment"></a>Vérifier le déploiement

Pour afficher l’espace de noms Service Bus déployé, vous pouvez ouvrir le groupe de ressources à partir du Portail Azure ou utiliser le script Azure PowerShell suivant. Si Cloud Shell est toujours ouvert, vous n’avez pas besoin de copier/exécuter les première et deuxième lignes du script suivant.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Get-AzServiceBusNamespace -ResourceGroupName $resourceGroupName -Name $serviceBusNamespaceName

Write-Host "Press [ENTER] to continue ..."
```

Dans ce tutoriel, Azure PowerShell permet de déployer le modèle. Pour d’autres méthodes de déploiement de modèle, consultez :

* [En utilisant le Portail Azure](../azure-resource-manager/templates/deploy-portal.md).
* [En utilisant Azure CLI](../azure-resource-manager/templates/deploy-cli.md).
* [En utilisant l’API REST](../azure-resource-manager/templates/deploy-rest.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources. Si Cloud Shell est toujours ouvert, vous n’avez pas besoin de copier/exécuter les première et deuxième lignes du script suivant.

```azurepowershell-interactive
$serviceBusNamespaceName = Read-Host -Prompt "Enter the same service bus namespace name used earlier"
$resourceGroupName = "${serviceBusNamespaceName}rg"

Remove-AzResourceGroup -ResourceGroupName $resourceGroupName

Write-Host "Press [ENTER] to continue ..."
```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez créé un espace de noms Service Bus. Consultez les autres guides de démarrage rapide pour apprendre à créer et utiliser des files d'attente et des rubriques/abonnements :

* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Prise en main des rubriques Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
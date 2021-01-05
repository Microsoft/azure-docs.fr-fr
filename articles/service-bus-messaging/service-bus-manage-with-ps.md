---
title: Utiliser PowerShell pour gérer les ressources Azure Service Bus | Microsoft Docs
description: Cet article explique comment utiliser le module Azure PowerShell pour créer et gérer des entités Service Bus (espaces de noms, files d’attente, rubriques, abonnements).
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: b6439deb2b86c2ea5b50fe3bdbad89a0875b2acc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88065741"
---
# <a name="use-powershell-to-manage-service-bus-resources"></a>Utiliser PowerShell pour gérer les ressources Service Bus

Microsoft Azure PowerShell est un environnement de création de scripts vous permettant de contrôler et d'automatiser le déploiement et la gestion des services Azure. Cet article explique comment utiliser le [module PowerShell du Gestionnaire de ressources Service Bus](/powershell/module/az.servicebus) pour approvisionner et gérer les entités Service Bus (espaces de noms, files d’attente, rubriques et abonnements) à l’aide d’une console Azure PowerShell locale ou d’un script.

Vous pouvez également gérer les entités Service Bus à l’aide de modèles Azure Resource Manager. Pour plus d’informations, consultez l’article [Création de ressources Service Bus à l’aide de modèles Azure Resource Manager](service-bus-resource-manager-overview.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Conditions préalables requises

Avant de commencer, vérifiez que les prérequis suivants sont respectés :

* Un abonnement Azure. Pour plus d’informations sur la façon de se procurer un abonnement, consultez les [options d’achat][purchase options], les [offres spéciales membres][member offers] ou découvrez comment créer un [compte gratuit][free account].
* Un ordinateur sur lequel est installé Azure PowerShell. Pour obtenir des instructions, consultez la page [Bien démarrer avec les applets de commande Azure PowerShell](/powershell/azure/get-started-azureps).
* Des connaissances générales sur les scripts PowerShell, les packages NuGet et .NET Framework.

## <a name="get-started"></a>Bien démarrer

La première étape consiste à utiliser PowerShell pour vous connecter à votre compte Azure et à votre abonnement Azure. Suivez les instructions décrites dans [Prise en main des applets de commande Azure PowerShell](/powershell/azure/get-started-azureps) pour vous connecter à votre compte Azure, récupérer les ressources de votre abonnement Azure et accéder à celles-ci.

## <a name="provision-a-service-bus-namespace"></a>Approvisionner un espace de noms Service Bus

Lorsque vous travaillez avec des espaces de noms Service Bus, vous pouvez utiliser les cmdlets [Get-AzServiceBusNamespace](/powershell/module/az.servicebus/get-azservicebusnamespace), [New-AzServiceBusNamespace](/powershell/module/az.servicebus/new-azservicebusnamespace), [Remove-AzServiceBusNamespace](/powershell/module/az.servicebus/remove-azservicebusnamespace), et [Set-AzServiceBusNamespace](/powershell/module/az.servicebus/set-azservicebusnamespace).

Cet exemple crée quelques variables locales dans le script ; `$Namespace` et `$Location`.

* `$Namespace` est le nom de l’espace de noms Service Bus que nous allons utiliser.
* `$Location` identifie le centre de données dans lequel nous approvisionnons l’espace de noms.
* `$CurrentNamespace` stocke l'espace de noms de référence récupéré (ou créé).

Dans un script réel, les variables `$Namespace` et `$Location` peuvent être transmises en tant que paramètres.

Cette partie du script effectue les opérations suivantes :

1. Elle tente de récupérer un espace de noms Service Bus portant le nom spécifié.
2. S'il trouve l'espace de noms recherché, il signale qu'il l'a trouvé.
3. S'il ne trouve pas l'espace de noms recherché, il le crée, puis il récupère le nouvel espace de noms.
   
    ``` powershell
    # Query to see if the namespace currently exists
    $CurrentNamespace = Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
   
    # Check if the namespace already exists or needs to be created
    if ($CurrentNamespace)
    {
        Write-Host "The namespace $Namespace already exists in the $Location region:"
        # Report what was found
        Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
    }
    else
    {
        Write-Host "The $Namespace namespace does not exist."
        Write-Host "Creating the $Namespace namespace in the $Location region..."
        New-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace -Location $Location
        $CurrentNamespace = Get-AzServiceBusNamespace -ResourceGroup $ResGrpName -NamespaceName $Namespace
        Write-Host "The $Namespace namespace in Resource Group $ResGrpName in the $Location region has been successfully created."
                
    }
    ```

### <a name="create-a-namespace-authorization-rule"></a>Créer une règle d’autorisation d’espace de noms

L’exemple suivant montre comment gérer les règles d’autorisation d’espace de noms à l’aide des cmdlets [New-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/new-azservicebusauthorizationrule), [Get-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/get-azservicebusauthorizationrule), [Set-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/set-azservicebusauthorizationrule), and [Remove-AzServiceBusAuthorizationRule](/powershell/module/az.servicebus/remove-azservicebusauthorizationrule).

```powershell
# Query to see if rule exists
$CurrentRule = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

# Check if the rule already exists or needs to be created
if ($CurrentRule)
{
    Write-Host "The $AuthRule rule already exists for the namespace $Namespace."
}
else
{
    Write-Host "The $AuthRule rule does not exist."
    Write-Host "Creating the $AuthRule rule for the $Namespace namespace..."
    New-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule -Rights @("Listen","Send")
    $CurrentRule = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule
    Write-Host "The $AuthRule rule for the $Namespace namespace has been successfully created."

    Write-Host "Setting rights on the namespace"
    $authRuleObj = Get-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthorizationRuleName $AuthRule

    Write-Host "Remove Send rights"
    $authRuleObj.Rights.Remove("Send")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Add Send and Manage rights to the namespace"
    $authRuleObj.Rights.Add("Send")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj
    $authRuleObj.Rights.Add("Manage")
    Set-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -AuthRuleObj $authRuleObj

    Write-Host "Show value of primary key"
    $CurrentKey = Get-AzServiceBusKey -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
        
    Write-Host "Remove this authorization rule"
    Remove-AzServiceBusAuthorizationRule -ResourceGroup $ResGrpName -NamespaceName $Namespace -Name $AuthRule
}
```

## <a name="create-a-queue"></a>Créer une file d’attente

Pour créer une file d’attente ou une rubrique, vérifiez l’espace de noms à l’aide du script décrit dans la section précédente. Ensuite, créez la file d’attente :

```powershell
# Check if queue already exists
$CurrentQ = Get-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName

if($CurrentQ)
{
    Write-Host "The queue $QueueName already exists in the $Location region:"
}
else
{
    Write-Host "The $QueueName queue does not exist."
    Write-Host "Creating the $QueueName queue in the $Location region..."
    New-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -EnablePartitioning $True
    $CurrentQ = Get-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName
    Write-Host "The $QueueName queue in Resource Group $ResGrpName in the $Location region has been successfully created."
}
```

### <a name="modify-queue-properties"></a>Modifier les propriétés de la file d’attente

Après l’exécution du script de la manière décrite dans la section précédente, vous pouvez utiliser la cmdlet de commande [Set-AzServiceBusQueue](/powershell/module/az.servicebus/set-azservicebusqueue) pour mettre à jour les propriétés d’une file d’attente, comme dans l’exemple suivant :

```powershell
$CurrentQ.DeadLetteringOnMessageExpiration = $True
$CurrentQ.MaxDeliveryCount = 7
$CurrentQ.MaxSizeInMegabytes = 2048
$CurrentQ.EnableExpress = $True

Set-AzServiceBusQueue -ResourceGroup $ResGrpName -NamespaceName $Namespace -QueueName $QueueName -QueueObj $CurrentQ
```

## <a name="provisioning-other-service-bus-entities"></a>Approvisionne d'autres entités Service Bus

Vous pouvez utiliser le [module PowerShell de Service Bus](/powershell/module/az.servicebus) pour approvisionner d’autres entités, telles que des rubriques et abonnements. La syntaxe de ces applets de commande est similaire à celle des applets de commande de création de file d’attente décrites dans la section précédente.

## <a name="next-steps"></a>Étapes suivantes

- Consultez la documentation complète du module PowerShell du Gestionnaire de ressources Service Bus [ici](/powershell/module/az.servicebus). Cette page liste toutes les applets de commande disponibles.
- Pour plus d’informations sur l’utilisation des modèles Azure Resource Manager, consultez l’article [Création de ressources Service Bus à l’aide de modèles Azure Resource Manager](service-bus-resource-manager-overview.md).
- Informations sur les [bibliothèques de gestion Service Bus .NET](service-bus-management-libraries.md).

D’autres manières de gérer les entités Service Bus sont décrites dans les billets de blog suivants :

* [Comment créer des files d'attente, des rubriques et des abonnements Service Bus à l'aide d'un script PowerShell](/archive/blogs/paolos/how-to-create-service-bus-queues-topics-and-subscriptions-using-a-powershell-script)
* [Comment créer un espace de noms et un concentrateur d'événements Service Bus à l'aide d'un script PowerShell](/archive/blogs/paolos/how-to-create-a-service-bus-namespace-and-an-event-hub-using-a-powershell-script)
* [Scripts PowerShell pour Service Bus](https://code.msdn.microsoft.com/Service-Bus-PowerShell-a46b7059)

<!--Anchors-->

[purchase options]: https://azure.microsoft.com/pricing/purchase-options/
[member offers]: https://azure.microsoft.com/pricing/member-offers/
[free account]: https://azure.microsoft.com/pricing/free-trial/
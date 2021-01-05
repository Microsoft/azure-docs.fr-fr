---
title: 'Démarrage rapide : Configurer la haute disponibilité avec Azure Front Door - Azure PowerShell'
description: Ce guide de démarrage rapide vous montre comment utiliser Azure Front Door pour créer une application web globale hautement disponible et très performante à l’aide d’Azure PowerShell.
services: front-door
documentationcenter: na
author: duongau
manager: KumudD
Customer intent: As an IT admin, I want to direct user traffic to ensure high availability of web applications.
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/21/2020
ms.author: duau
ms.openlocfilehash: 8ccc50dfc4c53f1ee207a04d7d7775775f13cb65
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91347909"
---
# <a name="quickstart-create-a-front-door-for-a-highly-available-global-web-application-using-azure-powershell"></a>Démarrage rapide : Créer une porte d’entrée pour une application web globale hautement disponible à l’aide d’Azure PowerShell

Démarrez avec Azure Front Door en utilisant Azure PowerShell pour créer une application web globale hautement disponible et très performante.

La porte d’entrée dirige le trafic web vers des ressources spécifiques dans un pool de back-ends. Vous définissez le domaine front-end, ajoutez des ressources à un pool de back-ends et créer une règle de routage. Cet article utilise une configuration simple d’un pool de back-ends avec deux ressources d’application web et une règle de routage unique utilisant le chemin par défaut correspondant à « /* ».

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell installé localement ou Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-resource-group"></a>Créer un groupe de ressources

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Créez un groupe de ressources avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) :

```azurepowershell-interactive
New-AzResourceGroup -Name myResourceGroupFD -Location centralus
```

## <a name="create-two-instances-of-a-web-app"></a>Créer deux instances d’une application web

Ce guide de démarrage rapide demande deux instances d’une application web qui s’exécutent dans différentes régions Azure. Les deux instances de l’application web s’exécutent en mode Actif/Actif, donc l’une ou l’autre peut s’occuper du trafic. Cette configuration diffère d’une configuration Active/Veille où l’une sert de basculement.

Si vous n’avez pas encore d’application web, utilisez le script suivant pour configurer deux exemples d’applications web.

```azurepowershell-interactive
# Create first web app in Central US region.
$webapp1 = New-AzWebApp `
-Name "WebAppContoso-$(Get-Random)" `
-Location centralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanCentralUS

# Create second web app in South Central US region.
$webapp2 = New-AzWebApp `
-Name "WebAppContoso-$(Get-Random)" `
-Location southcentralus `
-ResourceGroupName myResourceGroupFD `
-AppServicePlan myAppServicePlanSouthCentralUS
```

## <a name="create-a-front-door"></a>Créer une Front Door

Cette section explique en détail comment vous pouvez créer et configurer les composants suivants de la porte d’entrée :
    
* Un objet front-end contient le domaine par défaut de la porte d’entrée.
* Un pool de back-ends est un ensemble de back-ends équivalents sur lequel Front Door équilibre la charge de votre demande de client.
* Une règle de routage mappe l’hôte front-end et le modèle de chemin d’URL correspondant à un pool de back-ends spécifique.

### <a name="create-a-frontend-object"></a>Créer un objet front-end

L’objet front-end configure le nom d’hôte pour la porte d’entrée. Par défaut, le nom d’hôte a pour suffixe * *.azurefd.net*.

```azurepowershell-interactive
# Create a unique name
$fdname = "contoso-frontend-$(Get-Random)"

#Create the frontend object
$FrontendEndObject = New-AzFrontDoorFrontendEndpointObject `
-Name "frontendEndpoint1" `
-HostName $fdname".azurefd.net"
```

### <a name="create-the-backend-pool"></a>Créer le pool principal

Le pool de back-ends est constitué des deux applications web créées au début de ce guide de démarrage rapide. Les paramètres de sonde d’intégrité et d’équilibrage de charge définis dans cette étape utilisent des valeurs par défaut.

```azurepowershell-interactive
# Create backend objects that points to the hostname of the web apps
$backendObject1 = New-AzFrontDoorBackendObject `
-Address $webapp1.DefaultHostName
$backendObject2 = New-AzFrontDoorBackendObject `
-Address $webapp2.DefaultHostName

# Create a health probe object
$HealthProbeObject = New-AzFrontDoorHealthProbeSettingObject `
-Name "HealthProbeSetting"

# Create the load balancing setting object
$LoadBalancingSettingObject = New-AzFrontDoorLoadBalancingSettingObject `
-Name "Loadbalancingsetting" `
-SampleSize "4" `
-SuccessfulSamplesRequired "2" `
-AdditionalLatencyInMilliseconds "0"

# Create a backend pool using the backend objects, health probe, and load balancing settings
$BackendPoolObject = New-AzFrontDoorBackendPoolObject `
-Name "myBackendPool" `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-Backend $backendObject1,$backendObject2 `
-HealthProbeSettingsName "HealthProbeSetting" `
-LoadBalancingSettingsName "Loadbalancingsetting"
```

### <a name="create-a-routing-rule"></a>Créer une règle de routage

La règle de routage mappe le pool de back-ends au domaine front-end et définit la valeur du chemin par défaut correspondant sur « /* ».

```azurepowershell-interactive
# Create a routing rule mapping the frontend host to the backend pool
$RoutingRuleObject = New-AzFrontDoorRoutingRuleObject `
-Name LocationRule `
-FrontDoorName $fdname `
-ResourceGroupName myResourceGroupFD `
-FrontendEndpointName "frontendEndpoint1" `
-BackendPoolName "myBackendPool" `
-PatternToMatch "/*"
```
### <a name="create-the-front-door"></a>Créer la porte d’entrée

Vous pouvez à présent créer la porte d’entrée :

```azurepowershell-interactive
# Creates the Front Door
New-AzFrontDoor `
-Name $fdname `
-ResourceGroupName myResourceGroupFD `
-RoutingRule $RoutingRuleObject `
-BackendPool $BackendPoolObject `
-FrontendEndpoint $FrontendEndObject `
-LoadBalancingSetting $LoadBalancingSettingObject `
-HealthProbeSetting $HealthProbeObject
```

Une fois le déploiement réussi, vous pouvez le tester en suivant les étapes décrites dans la section suivante.

## <a name="test-the-front-door"></a>Tester la porte d’entrée

Exécutez les commandes suivantes pour obtenir le nom d’hôte de la porte d’entrée.

```azurepowershell-interactive
# Gets Front Door in resource group and output the hostname of the frontend domain.
$fd = Get-AzFrontDoor -ResourceGroupName myResourceGroupFD
$fd.FrontendEndpoints[0].Hostname
```

Ouvrez un navigateur web et entrez le nom d’hôte obtenu à partir des commandes. La porte d’entrée dirige votre demande vers l’une des ressources back-end. 

:::image type="content" source="./media/quickstart-create-front-door-powershell/front-door-test-page.png" alt-text="Page de test de la porte d’entrée":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la porte d’entrée, supprimez le groupe de ressources. Quand vous supprimez le groupe de ressources, vous supprimez aussi la porte d’entrée et toutes ses ressources associées. 

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name myResourceGroupFD
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :
* Front Door
* Deux applications web

Pour savoir comment ajouter un domaine personnalisé à votre porte d’entrée, passez aux tutoriels Front Door.

> [!div class="nextstepaction"]
> [Ajouter un domaine personnalisé](front-door-custom-domain.md)

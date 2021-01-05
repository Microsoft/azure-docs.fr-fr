---
title: 'Démarrage rapide : Créer un gestionnaire de trafic en utilisant un modèle Azure Resource Manager (modèle ARM)'
description: Ce guide de démarrage rapide décrit comment créer un profil Azure Traffic Manager en utilisant un modèle Azure Resource Manager (modèle ARM).
services: traffic-manager
author: duongau
ms.service: traffic-manager
ms.topic: quickstart
ms.custom: subject-armqs
ms.author: duau
ms.date: 09/01/2020
ms.openlocfilehash: ec569781a6318062810358c2c5e17ba71efc4f71
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92676010"
---
# <a name="quickstart-create-a-traffic-manager-profile-using-an-arm-template"></a>Démarrage rapide : Créer un profil Traffic Manager en utilisant un modèle ARM

Ce guide de démarrage rapide explique comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer un profil Traffic Manager avec des points de terminaison externes en utilisant la méthode de routage des performances.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-traffic-manager-external-endpoint%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-traffic-manager-external-endpoint).

:::code language="json" source="~/quickstart-templates/101-traffic-manager-external-endpoint/azuredeploy.json":::

Une seule ressource Azure est définie dans le modèle :

* [**Microsoft.Network/trafficManagerProfiles**](/azure/templates/microsoft.network/trafficmanagerprofiles)

Pour trouver d’autres modèles liés à Azure Traffic Manager, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez **Essayer** à partir du bloc de code suivant pour ouvrir Azure Cloud Shell, puis suivez les instructions pour vous connecter à Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-traffic-manager-external-endpoint/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Patientez jusqu'à ce que vous voyiez l’invite de la console.

1. Sélectionnez **Copier** dans le bloc de code précédent pour copier le script PowerShell.

1. Cliquez sur le volet de la console shell, puis sélectionnez **Coller**.

1. Entrez les valeurs.

    Le déploiement du modèle crée un profil avec deux points de terminaison externes. **Endpoint1** utilise un point de terminaison cible de `www.microsoft.com` avec comme emplacement **Europe Nord**. **Endpoint2** utilise un point de terminaison cible de `docs.microsoft.com` avec comme localisation **USA Centre Sud**.

    Le nom du groupe de ressources est le nom du projet avec **rg** ajouté.

    > [!NOTE]
    > **uniqueDNSname** doit être un nom global unique pour que le modèle puisse être déployé correctement. Si le déploiement échoue, recommencez à l’Étape 1.

    Le déploiement du modèle prend quelques minutes. Une fois l’opération terminée, le résultat ressemble à ce qui suit :

    :::image type="content" source="./media/quickstart-create-traffic-manager-profile/traffic-manager-arm-powershell-output.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager Azure Traffic Manager":::

Azure PowerShell est utilisé pour déployer le modèle. Outre Azure PowerShell, vous pouvez également utiliser le portail Azure, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

1. Vous pouvez déterminer le nom DNS du profil Traffic Manager en avec [Get-AzTrafficManagerProfile](/powershell/module/az.trafficmanager/get-aztrafficmanagerprofile).

    ```azurepowershell-interactive
    Get-AzTrafficManagerProfile -Name ExternalEndpointExample -ResourceGroupName $resourceGroupName | Select RelativeDnsName
    ```

    Copiez la valeur de **RelativeDnsName**. Le nom DNS de votre profil Traffic Manager est `<relativednsname>.trafficmanager.net`.

1. À partir d’un PowerShell local, exécutez la commande suivante en remplaçant la variable **{relativeDNSname}** par `<relativednsname>.trafficmanager.net`.

    ```powershell
    Resolve-DnsName -Name {relativeDNSname} | Select-Object NameHost | Select -First 1
    ```

    Vous devez obtenir un NameHost `www.microsoft.com` ou `docs.microsoft.com`, en fonction de la région qui est la plus proche de vous.

1. Pour vérifier si vous pouvez résoudre l’autre point de terminaison, désactivez le point de terminaison pour la cible que vous avez obtenu lors de la dernière étape. Remplacez **{endpointName}** par **endpoint1** ou **endpoint2** pour désactiver la cible pour `www.microsoft.com` ou `docs.microsoft.com`, respectivement.

    ```azurepowershell-interactive
    Disable-AzTrafficManagerEndpoint -Name {endpointName} -Type ExternalEndpoints -ProfileName ExternalEndpointExample -ResourceGroupName $resourceGroupName -Force
    ```

1. Réexécutez la commande de l’Étape 2 dans un PowerShell local. Cette fois, vous devez obtenir l’autre NameHost pour l’autre point de terminaison.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du profil Traffic Manager, supprimez le groupe de ressources. Cette action supprime le profil Traffic Manager et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un profil Traffic Manager.

Pour plus d’informations sur le routage du trafic, passez aux tutoriels Traffic Manager.

> [!div class="nextstepaction"]
> [Didacticiels Traffic Manager](tutorial-traffic-manager-improve-website-response.md)

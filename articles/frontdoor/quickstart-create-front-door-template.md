---
title: 'Démarrage rapide : Créer un service Azure Front Door avec un modèle Azure Resource Manager'
description: Ce guide de démarrage rapide décrit comment utiliser un modèle Azure Resource Manager pour créer un service Azure Front Door.
services: front-door
documentationcenter: ''
author: duongau
editor: ''
ms.assetid: ''
ms.service: frontdoor
ms.devlang: na
ms.topic: quickstart
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/14/2020
ms.author: duau
ms.custom: subject-armqs
ms.openlocfilehash: 223006193219afe4179f3161d5e60e6439207b22
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92896053"
---
# <a name="quickstart-create-a-front-door-using-an-arm-template"></a>Démarrage rapide : Créer un service Front Door à l’aide d’un modèle Resource Manager

Ce guide de démarrage rapide explique comment utiliser un modèle Azure Resource Manager pour créer un service Front Door afin de configurer la haute disponibilité d’un point de terminaison web.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-front-door-create-basic%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

* Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.
* Adresse IP ou nom de domaine complet (FQDN) d’un site web ou d’une application web.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-front-door-create-basic).

Dans ce guide de démarrage rapide, vous allez créer une configuration Front Door avec un seul back-end et un seul chemin par défaut correspondant à `/*`.

:::code language="json" source="~/quickstart-templates/101-front-door-create-basic/azuredeploy.json":::

Une seule ressource Azure est définie dans le modèle :

* [**Microsoft.Network/frontDoors**](/azure/templates/microsoft.network/frontDoors)

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez **Essayer** à partir du bloc de code suivant pour ouvrir Azure Cloud Shell, puis suivez les instructions pour vous connecter à Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-front-door-create-basic/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Patientez jusqu'à ce que vous voyiez l’invite de la console.

1. Sélectionnez **Copier** dans le bloc de code précédent pour copier le script PowerShell.

1. Cliquez sur le volet de la console shell, puis sélectionnez **Coller**.

1. Entrez les valeurs.

    Le déploiement du modèle crée un service Front Door avec un seul back-end. Dans cet exemple, `microsoft.com` est utilisé comme **backendAddress**.

    Le nom du groupe de ressources est le nom du projet avec **rg** ajouté.

    > [!NOTE]
    > **frontDoorName** doit être un nom global unique pour que le modèle se déploie correctement. Si le déploiement échoue, recommencez à l’Étape 1.

    Le déploiement du modèle prend quelques minutes. Une fois l’opération terminée, le résultat ressemble à ce qui suit :

    :::image type="content" source="./media/quickstart-create-front-door-template/front-door-template-deployment-powershell-output.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager Front Door":::

Azure PowerShell est utilisé pour déployer le modèle. Outre Azure PowerShell, vous pouvez également utiliser le portail Azure, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Groupes de ressources** dans le volet gauche.

1. Sélectionnez le groupe de ressources que vous avez créé dans la section précédente. Le nom du groupe de ressources par défaut est le nom du projet avec **rg** ajouté.

1. Sélectionnez le service Front Door que vous avez créé précédemment et cliquez sur le lien **Hôte front-end**. Le lien ouvre un navigateur web qui vous redirige vers le nom de domaine complet de votre back-end que vous avez défini lors de la création.

    :::image type="content" source="./media/quickstart-create-front-door-template/front-door-overview.png" alt-text="Vue d’ensemble du portail Front Door":::

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin du service Front Door, supprimez le groupe de ressources. Vous supprimez ainsi le service Front Door et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé un service Front Door.

Pour savoir comment ajouter un domaine personnalisé à votre service Front Door, passez aux tutoriels Front Door.

> [!div class="nextstepaction"]
> [Tutoriels Front Door](front-door-custom-domain.md)

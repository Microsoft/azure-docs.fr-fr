---
title: Créer une zone et un enregistrement Azure DNS – Modèle Azure Resource Manager (modèle ARM)
titleSuffix: Azure DNS
description: Découvrez comment créer une zone et un enregistrement DNS dans Azure DNS. Ce démarrage rapide pas à pas vous aide à créer et gérer votre première zone et votre premier enregistrement DNS à l’aide d’un modèle Azure Resource Manager (modèle ARM).
services: dns
author: duongau
ms.service: dns
ms.topic: quickstart
ms.date: 09/8/2020
ms.author: duau
ms.custom: subject-armqs
ms.openlocfilehash: 24460167e2279e7d3001d0bc16d050beb5b55289
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790999"
---
# <a name="quickstart-create-an-azure-dns-zone-and-record-using-an-arm-template"></a>Créer une zone et un enregistrement Azure DNS à l’aide d’un modèle ARM

Ce guide de démarrage rapide explique comment utiliser un modèle Azure Resource Manager (modèle ARM) pour créer une zone DNS contenant un enregistrement `A`.

[!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-azure-dns-new-zone%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="review-the-template"></a>Vérifier le modèle

Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-azure-dns-new-zone).

Dans ce guide de démarrage rapide, vous allez créer une zone DNS unique avec un suffixe `azurequickstart.org`. Un enregistrement `A` pointant vers deux adresses IP sera également placé dans la zone.

:::code language="json" source="~/quickstart-templates/101-azure-dns-new-zone/azuredeploy.json":::

Deux ressources Azure ont été définies dans le modèle :

- [Microsoft.Network**/dnsZones/*
- [**Microsoft.Network/dnsZones/A**](/azure/templates/microsoft.network/dnsZones/A) : utilisé pour créer un enregistrement `A` dans la zone.

Pour trouver d’autres modèles liés à Azure Traffic Manager, consultez [Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Network&pageNumber=1&sort=Popular).

## <a name="deploy-the-template"></a>Déployer le modèle

1. Sélectionnez **Essayer** à partir du bloc de code suivant pour ouvrir Azure Cloud Shell, puis suivez les instructions pour vous connecter à Azure.

    ```azurepowershell-interactive
    $projectName = Read-Host -Prompt "Enter a project name that is used for generating resource names"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $templateUri = "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-azure-dns-new-zone/azuredeploy.json"

    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateUri $templateUri

    Read-Host -Prompt "Press [ENTER] to continue ..."
    ```

    Patientez jusqu'à ce que vous voyiez l’invite de la console.

1. Sélectionnez **Copier** dans le bloc de code précédent pour copier le script PowerShell.

1. Cliquez sur le volet de la console shell, puis sélectionnez **Coller**.

1. Entrez les valeurs.

    Le déploiement du modèle a pour effet de créer une zone avec un enregistrement `A` pointant vers deux adresses IP. Le nom du groupe de ressources est le nom du projet avec **rg** ajouté.

    Le déploiement du modèle prend 2 à 3 secondes. Une fois l’opération terminée, le résultat ressemble à ce qui suit :

    :::image type="content" source="./media/dns-getstarted-template/create-dns-zone-powershell-output.png" alt-text="Sortie du déploiement PowerShell du modèle Resource Manager Azure DNS":::

Azure PowerShell est utilisé pour déployer le modèle. Outre Azure PowerShell, vous pouvez également utiliser le portail Azure, l’interface Azure CLI et l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-portal.md).

## <a name="validate-the-deployment"></a>Valider le déploiement

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Sélectionnez **Groupes de ressources** dans le volet gauche.

1. Sélectionnez le groupe de ressources que vous avez créé dans la section précédente. Le nom du groupe de ressources par défaut est le nom du projet avec **rg** ajouté.

1. Le groupe de ressources doit contenir les ressources indiquées ci-dessous :

    :::image type="content" source="./media/dns-getstarted-template/resource-group-dns-zone.png" alt-text="Groupe de ressources de déploiement de la zone DNS":::

1. Sélectionnez la zone DNS avec le suffixe `azurequickstart.org` pour vérifier que la zone est créée correctement avec un enregistrement `A` référençant la valeur `1.2.3.4` et `1.2.3.5`.

    :::image type="content" source="./media/dns-getstarted-template/dns-zone-overview.png" alt-text="Déploiement de la zone DNS":::

1. Copiez un des noms de serveur de noms de l’étape précédente.

1. Ouvrez une invite de commandes et exécutez la commande suivante :

   ```cmd
   nslookup www.<dns zone name> <name server name>
   ```

   Par exemple :

   ```cmd
   nslookup www.2lwynbseszpam.azurequickstart.org ns1-09.azure-dns.com.
   ```

   Vous devez voir quelque chose de semblable à la capture d’écran suivante :

    :::image type="content" source="./media/dns-getstarted-template/dns-zone-validation.png" alt-text="Nslookup de zone DNS":::

Le nom d’hôte `www.2lwynbseszpam.azurequickstart.org` se résout en `1.2.3.4` et `1.2.3.5`, tel que vous l’avez configuré. Ce résultat confirme que la résolution de noms fonctionne correctement.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’avez plus besoin des ressources que vous avez créées avec la zone DNS, supprimez le groupe de ressources. Vous supprimez ainsi la zone DNS et toutes les ressources associées.

Pour supprimer le groupe de ressources, appelez l’applet de commande `Remove-AzResourceGroup` :

```azurepowershell-interactive
Remove-AzResourceGroup -Name <your resource group name>
```

## <a name="next-steps"></a>Étapes suivantes

Dans le cadre de ce guide de démarrage rapide, vous avez créé les éléments suivants :

- Zone DNS
- Enregistrement `A`

Maintenant que vous avez créé votre premier enregistrement et votre première zone DNS à l’aide d’un modèle ARM, vous pouvez créer des enregistrements pour une application web dans un domaine personnalisé.

> [!div class="nextstepaction"]
> [Créer des enregistrements DNS pour une application web dans un domaine personnalisé](./dns-web-sites-custom-domain.md)

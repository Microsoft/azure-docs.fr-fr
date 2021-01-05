---
title: 'Démarrage rapide : Créer un compte Azure Purview en utilisant Azure PowerShell ou Azure CLI (préversion)'
description: Ce guide de démarrage rapide détaille la création d’un compte Azure Purview avec Azure PowerShell ou Azure CLI.
author: hophanms
ms.author: hophan
ms.service: purview
ms.subservice: purview-data-catalog
ms.topic: quickstart
ms.date: 11/23/2020
ms.openlocfilehash: 7db546ec9038403bd9e34f637e9d9de368277d52
ms.sourcegitcommit: cc13f3fc9b8d309986409276b48ffb77953f4458
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/14/2020
ms.locfileid: "97400300"
---
# <a name="quickstart-create-an-azure-purview-account-using-azure-powershellazure-cli"></a>Démarrage rapide : Créer un compte Azure Purview en utilisant Azure PowerShell ou Azure CLI

> [!IMPORTANT]
> Azure Purview est actuellement disponible en PRÉVERSION. L’[Avenant aux conditions d’utilisation pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/) contient des conditions légales supplémentaires qui s’appliquent aux fonctionnalités Azure en version bêta, en préversion ou pas encore en disponibilité générale.

Dans ce guide de démarrage rapide, vous créez un compte Azure Purview en utilisant Azure PowerShell ou Azure CLI.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

* Le compte d’utilisateur que vous utilisez pour vous connecter à Azure doit être membre du rôle Contributeur ou Propriétaire, ou être administrateur de l’abonnement Azure.

* Votre propre [locataire Azure Active Directory](https://docs.microsoft.com/azure/active-directory/fundamentals/active-directory-access-create-new-tenant).

* Installez Azure PowerShell ou Azure CLI dans votre machine cliente pour déployer le modèle : [Déploiement en ligne de commande](https://docs.microsoft.com/azure/azure-resource-manager/templates/template-tutorial-create-first-template?tabs=azure-cli#command-line-deployment)

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous au [portail Azure](https://portal.azure.com) avec votre compte Azure.

## <a name="configure-your-subscription"></a>Configurer votre abonnement

Si nécessaire, suivez ces étapes pour configurer votre abonnement en vue d’autoriser l’exécution d’Azure Purview dans votre abonnement :

   1. Dans le portail Azure, recherchez et sélectionnez **Abonnements**.

   1. Dans la liste des abonnements, sélectionnez l’abonnement que vous souhaitez utiliser. L’autorisation d’accès administratif pour l’abonnement est nécessaire.

      :::image type="content" source="./media/create-catalog-portal/select-subscription.png" alt-text="Capture d’écran illustrant la sélection d’un abonnement dans le portail Azure.":::

   1. Sur votre abonnement, sélectionnez **Fournisseurs de ressources**. Dans le volet **Fournisseurs de ressources**, recherchez et inscrivez les trois fournisseurs de ressources : 
       1. **Microsoft.Purview**
       1. **Microsoft.Storage**
       1. **Microsoft.EventHub** 
      
      S’ils ne sont pas inscrits, inscrivez-les en sélectionnant **Inscrire**.

      :::image type="content" source="./media/create-catalog-portal/register-purview-resource-provider.png" alt-text="Capture d’écran illustrant l’inscription du fournisseur de ressources Microsoft.Azure Purview dans le portail Azure.":::

## <a name="create-an-azure-purview-account-instance"></a>Créer une instance de compte Azure Purview

1. Se connecter avec des informations d’identification Azure

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Connect-AzAccount
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az login
    ```
    
    ---

1. Si vous avez plusieurs abonnements Azure, sélectionnez celui que vous souhaitez utiliser :

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    Set-AzContext [SubscriptionID/SubscriptionName]
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az account set --subscription [SubscriptionID/SubscriptionName]
    ```
    
    ---

1. Créez un groupe de ressources pour votre compte Purview. Vous pouvez ignorer cette étape si vous en avez déjà un :

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroup `
      -Name myResourceGroup `
      -Location "East US"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    ```azurecli
    az group create \
      --name myResourceGroup \
      --location "East US"
    ```
    
    ---

1. Créez un fichier de modèle Purview, tel que `purviewtemplate.json`. Vous pouvez mettre à jour `name`, `location` et `capacity` (`4` ou `16`) :

    ```json
    {
      "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "resources": [
        {
          "name": "<yourPurviewAccountName>",
          "type": "Microsoft.Purview/accounts",
          "apiVersion": "2020-12-01-preview",
          "location": "EastUs",
          "identity": {
            "type": "SystemAssigned"
          },
          "properties": {
            "networkAcls": {
              "defaultAction": "Allow"
            }
          },
          "dependsOn": [],
          "sku": {
            "name": "Standard",
            "capacity": "4"
          },
          "tags": {}
        }
      ],
      "outputs": {}
    }
    ```

1. Déployer le modèle Purview

    # <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
    
    ```azurepowershell
    New-AzResourceGroupDeployment -ResourceGroupName "<myResourceGroup>" -TemplateFile "<PATH TO purviewtemplate.json>"
    ```
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    
    Pour exécuter cette commande de déploiement, vous devez disposer de la [dernière version](/cli/azure/install-azure-cli) d’Azure CLI.
    
    ```azurecli
    az deployment group create --resource-group "<myResourceGroup>" --template-file "<PATH TO purviewtemplate.json>"
    ```
    
    ---

1. La commande de déploiement retourne des résultats. Recherchez `ProvisioningState` pour voir si le déploiement s’est déroulé correctement.
    
## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez vu comment créer un compte Azure Purview.

Passez à l’article suivant pour savoir comment autoriser les utilisateurs à accéder à votre compte Azure Purview. 

> [!div class="nextstepaction"]
> [Ajouter des utilisateurs à votre compte Azure Purview](catalog-permissions.md)
---
title: Utiliser les informations de référence des modèles
description: Utilisez les informations de référence sur le modèle Azure Resource Manager (modèle ARM) pour créer un modèle.
author: mumian
ms.date: 04/23/2020
ms.topic: tutorial
ms.author: jgao
ms.custom: seodec18
ms.openlocfilehash: a44852fb2f491dd949b58217eca3e4f3e392cf17
ms.sourcegitcommit: d2d1c90ec5218b93abb80b8f3ed49dcf4327f7f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97584134"
---
# <a name="tutorial-utilize-the-arm-template-reference"></a>Tutoriel : Utiliser les informations de référence sur le modèle ARM

Découvrez comment trouver les informations concernant le schéma de modèle et vous en servir pour créer des modèles Azure Resource Manager (modèles ARM).

Dans ce tutoriel, vous utiliserez un modèle de base issu des modèles de démarrage rapide Azure. Vous allez personnaliser le modèle en vous appuyant sur la documentation de référence sur le modèle.

![Resource Manager modèle référence déploiement compte de stockage](./media/template-tutorial-use-template-reference/resource-manager-template-tutorial-deploy-storage-account.png)

Ce tutoriel décrit les tâches suivantes :

> [!div class="checklist"]
> * Ouvrir un modèle de démarrage rapide
> * Comprendre le modèle
> * Trouver la référence du modèle
> * Modifier le modèle
> * Déployer le modèle

Si vous ne disposez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="prerequisites"></a>Prérequis

Pour effectuer ce qui est décrit dans cet article, vous avez besoin des éléments suivants :

* Visual Studio Code avec l’extension Outils Resource Manager. Consultez [Démarrage rapide : Créer des modèles ARM avec Visual Studio Code](quickstart-create-templates-use-visual-studio-code.md).

## <a name="open-a-quickstart-template"></a>Ouvrir un modèle de démarrage rapide

[Modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/) est un référentiel de modèles ARM. Au lieu de créer un modèle à partir de zéro, vous pouvez chercher un exemple de modèle et le personnaliser. Le modèle utilisé dans ce démarrage rapide se nomme [Créer un compte de stockage standard](https://azure.microsoft.com/resources/templates/101-storage-account-create/). Le modèle définit une ressource de compte de stockage Azure.

1. À partir de Visual Studio Code, sélectionnez **Fichier** > **Ouvrir un fichier**.
1. Collez l’URL suivante dans **Nom de fichier** :

    ```url
    https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-storage-account-create/azuredeploy.json
    ```

1. Sélectionnez **Ouvrir** pour ouvrir le fichier.
1. Sélectionnez **Fichier** > **Enregistrer sous** pour enregistrer le fichier sous le nom _azuredeploy.json_ sur votre ordinateur local.

## <a name="understand-the-schema"></a>Comprendre le schéma

1. Dans Visual Studio Code, réduisez le modèle jusqu’au niveau de la racine. Vous avez la structure la plus simple avec les éléments suivants :

    ![Structure la plus simple du modèle Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-simplest-structure.png)

    * `$schema` : spécifiez l’emplacement du fichier de schéma JSON qui décrit la version du langage du modèle.
    * `contentVersion` : spécifiez n’importe quelle valeur pour que cet élément documente les modifications importantes dans votre modèle.
    * `parameters` : spécifiez les valeurs fournies lors de l'exécution du déploiement pour personnaliser le déploiement des ressources.
    * `variables` : spécifiez les valeurs utilisées en tant que fragments JSON dans le modèle pour simplifier les expressions du langage du modèle.
    * `resources` : spécifiez les types de ressource déployés ou mis à jour dans un groupe de ressources.
    * `outputs` : spécifiez les valeurs retournées après le déploiement.

1. Développez `resources`. Une ressource `Microsoft.Storage/storageAccounts` est définie. Le nom de la référence SKU utilise une valeur de paramètre. Le paramètre est appelé `storageAccountType`.

    ![Définition du modèle de compte de stockage Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resource.png)

1. Développez la section `parameters` pour voir la définition de `storageAccountType`. Le paramètre a quatre valeurs autorisées. Vous allez rechercher les autres valeurs autorisées, puis modifier la définition du paramètre.

    ![Références SKU des ressources de compte de stockage du modèle Resource Manager](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus-old.png)

## <a name="find-the-template-reference"></a>Trouver la référence du modèle

1. Accédez à la [documentation de référence du modèle Azure](/azure/templates/).
1. Dans la zone **Filtrer par titre**, entrez **comptes de stockage**, puis sélectionnez la première occurrence de **Comptes de stockage** sous **Informations de référence > Stockage**.

    ![Resource Manager modèle référence compte de stockage](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts.png)

    Un fournisseur de ressources propose généralement plusieurs versions d’API :

    ![Resource Manager modèle référence versions compte de stockage](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-versions.png)

1. Sélectionnez **Toutes les ressources** en dessous de **Stockage** dans le volet de gauche. Cette page liste les types de ressources et les versions du fournisseur de ressources de stockage. Il est recommandé d’utiliser les dernières versions d’API pour les types de ressources définis dans votre modèle.

    ![Resource Manager modèle référence compte de stockage types versions](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-types-versions.png)

1. Sélectionnez la dernière version du type de ressource `storageAccount`. La dernière version était **2019-06-01** au moment où cet article a été rédigé. Assurez-vous que cette version correspond à la version utilisée pour la ressource de compte de stockage dans votre modèle. Si vous mettez à jour la version de l’API, vérifiez que la définition de ressource correspond à la référence de modèle.

1. Cette page liste les détails du type de ressource storageAccount. Par exemple, elle présente les valeurs autorisées pour l’objet **Sku**. Il existe davantage de références SKU que celles listées dans le modèle de démarrage rapide ouvert précédemment. Vous pouvez personnaliser le modèle de démarrage rapide pour inclure tous les types de stockage disponibles.

    ![Resource Manager modèle référence compte de stockage références sku](./media/template-tutorial-use-template-reference/resource-manager-template-resources-reference-storage-accounts-skus.png)

## <a name="edit-the-template"></a>Modifier le modèle

À partir de Visual Studio Code, ajoutez les types de compte de stockage supplémentaires indiqués dans la capture d’écran suivante :

![Resource Manager modèle compte de stockage ressources](./media/template-tutorial-use-template-reference/resource-manager-template-storage-resources-skus.png)

## <a name="deploy-the-template"></a>Déployer le modèle

1. Se connecter à [Azure Cloud Shell](https://shell.azure.com)

1. Choisissez votre environnement préféré en sélectionnant **PowerShell** ou **Bash** (pour CLI) en haut à gauche.  Il est nécessaire de redémarrer l’interpréteur de commandes lors d’un tel changement.

    ![Fichier de chargement du Cloud Shell du portail Azure](./media/template-tutorial-use-template-reference/azure-portal-cloud-shell-upload-file.png)

1. Sélectionnez **Charger/Télécharger des fichiers**, puis **Charger**. Consultez la capture d’écran précédente. Sélectionnez le fichier que vous avez enregistré dans la section précédente. Après avoir chargé le fichier, vous pouvez utiliser la commande `ls` et la commande `cat` pour vérifier que le chargement a été correctement effectué.

1. Dans le Cloud Shell, exécutez les commandes suivantes. Sélectionnez l’onglet pour afficher le code PowerShell ou CLI.

   Au moment de déployer le modèle, spécifiez le paramètre `storageAccountType` avec une valeur nouvellement ajoutée, par exemple **Standard_RAGRS**. Si vous utilisez le modèle de démarrage rapide d’origine, le déploiement échoue car **Standard_RAGRS** n’était pas une valeur autorisée.

    # <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

    ```azurecli
    echo "Enter a project name that is used to generate resource group name:" &&
    read projectName &&
    echo "Enter the location (i.e. centralus):" &&
    read location &&
    resourceGroupName="${projectName}rg" &&
    az group create --name $resourceGroupName --location "$location" &&
    az deployment group create --resource-group $resourceGroupName --template-file "$HOME/azuredeploy.json" --parameters storageAccountType='Standard_RAGRS'
    ```

    # <a name="powershell"></a>[PowerShell](#tab/PowerShell)

    ```azurepowershell
    $projectName = Read-Host -Prompt "Enter a project name that is used to generate resource group name"
    $location = Read-Host -Prompt "Enter the location (i.e. centralus)"
    $resourceGroupName = "${projectName}rg"

    New-AzResourceGroup -Name $resourceGroupName -Location "$location"
    New-AzResourceGroupDeployment -ResourceGroupName $resourceGroupName -TemplateFile "$HOME/azuredeploy.json" -storageAccountType "Standard_RAGRS"
    ```

    ---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Lorsque vous n’en avez plus besoin, nettoyez les ressources Azure que vous avez déployées en supprimant le groupe de ressources.

1. Dans le portail Azure, sélectionnez **Groupe de ressources** dans le menu de gauche.
1. Entrez le nom du groupe de ressources dans le champ **Filtrer par nom**.
1. Sélectionnez le nom du groupe de ressources.  Vous devriez voir six ressources au total dans le groupe de ressources.
1. Sélectionnez **Supprimer le groupe de ressources** dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez appris à utiliser la référence de modèle pour personnaliser un modèle existant. Pour apprendre à créer plusieurs instances de compte de stockage, voir :

> [!div class="nextstepaction"]
> [Créer plusieurs instances](./template-tutorial-create-multiple-instances.md)

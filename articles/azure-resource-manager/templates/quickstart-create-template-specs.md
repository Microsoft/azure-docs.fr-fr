---
title: Créer et déployer une spec de modèle
description: Découvrez comment créer une spec de modèle à partir d’un modèle ARM. Ensuite, déployez la spec de modèle vers un groupe de ressources dans votre abonnement.
author: tfitzmac
ms.date: 12/14/2020
ms.topic: quickstart
ms.author: tomfitz
ms.openlocfilehash: 1d3e7c6ee6d19d4d2cd3828d5abf95ccb5457e76
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97511346"
---
# <a name="quickstart-create-and-deploy-template-spec-preview"></a>Démarrage rapide : Créer et déployer une spec de modèle (préversion)

Ce guide de démarrage rapide montre comment empaqueter un modèle Azure Resource Manager (modèle ARM) dans une [spec de modèle](template-specs.md). Ensuite, vous déployez cette spec de modèle. Votre spec de modèle contient un modèle ARM qui déploie un compte de stockage.

## <a name="prerequisites"></a>Prérequis

Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).

> [!NOTE]
> La fonctionnalité Specs de modèle est actuellement en préversion. Pour l’utiliser avec Azure PowerShell, vous devez installer la [version 5.0.0 ou ultérieure](/powershell/azure/install-az-ps). Pour l’utiliser avec Azure CLI, utilisez la [version 2.14.2 ou ultérieure](/cli/azure/install-azure-cli).

## <a name="create-template"></a>Créer un modèle

Vous créez une spec de modèle à partir d’un modèle local. Copiez le modèle suivant et enregistrez-le localement dans un fichier nommé **azuredeploy.json**. Ce démarrage rapide part du principe que vous avez enregistré dans le chemin d’accès **c:\Templates\azuredeploy.json**, mais vous pouvez utiliser n’importe quel chemin.

:::code language="json" source="~/quickstart-templates/101-storage-account-create/azuredeploy.json":::

## <a name="create-template-spec"></a>Créer une spec de modèle

La spec de modèle est un type de ressource nommé `Microsoft.Resources/templateSpecs`. Pour créer une spec de modèle, utilisez PowerShell, Azure CLI, le portail ou un modèle ARM.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Créez un groupe de ressources pour contenir la spec de modèle.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

1. Créez la spec de modèle dans ce groupe de ressources. Attribuez le nom **storageSpec** à la nouvelle spec de modèle.

    ```azurepowershell
    New-AzTemplateSpec `
      -Name storageSpec `
      -Version "1.0" `
      -ResourceGroupName templateSpecRG `
      -Location westus2 `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

1. Créez un groupe de ressources pour contenir la spec de modèle.

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Créez la spec de modèle dans ce groupe de ressources. Attribuez le nom **storageSpec** à la nouvelle spec de modèle.

    ```azurecli
    az ts create \
      --name storageSpec \
      --version "1.0" \
      --resource-group templateSpecRG \
      --location "westus2" \
      --template-file "c:\Templates\azuredeploy.json"
    ```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Recherchez **specs de modèle**. Sélectionnez **Specs de modèle** parmi les options disponibles.

    :::image type="content" source="./media/quickstart-create-template-specs/search-template-spec.png" alt-text="recherche de specs de modèle":::

1. Sélectionnez **Importer le modèle**.

    :::image type="content" source="./media/quickstart-create-template-specs/import-template.png" alt-text="importation du modèle":::

1. Sélectionnez l’icône du dossier.

    :::image type="content" source="./media/quickstart-create-template-specs/open-folder.png" alt-text="ouverture du dossier":::

1. Accédez au modèle local que vous avez enregistré et sélectionnez-le. Sélectionnez **Ouvrir**.
1. Sélectionnez **Importer**.

    :::image type="content" source="./media/quickstart-create-template-specs/select-import.png" alt-text="sélection de l’option d’importation":::

1. Renseignez les valeurs suivantes :

    - **Nom** : entrez un nom pour la spec de modèle.  Par exemple, **storageSpec**.
    - **Abonnement** : sélectionnez l’abonnement Azure utilisé pour créer la spec de modèle.
    - **Groupe de ressources** : sélectionnez **Créer nouveau**, puis entrez un nouveau nom de groupe de ressources.  Par exemple, **templateSpecRG**.
    - **Emplacement** : sélectionnez un emplacement pour le groupe de ressources. Par exemple, **USA Ouest 2**.
    - **Version** : entrez une version pour la spec de modèle. Utilisez **1.0**.

1. Sélectionnez **Vérifier + créer**.
1. Sélectionnez **Create** (Créer).

# <a name="arm-template"></a>[Modèle ARM](#tab/azure-resource-manager)

> [!NOTE]
> Au lieu d’utiliser un modèle ARM, nous vous conseillons d’utiliser PowerShell ou l’interface CLI pour créer votre spec de modèle. Ces outils convertissent automatiquement les modèles liés en artefacts connectés à votre modèle principal. Lorsque vous utilisez un modèle ARM pour créer la spec de modèle, vous devez ajouter manuellement ces modèles liés en tant qu’artefacts, ce qui peut s’avérer compliqué.

1. Lorsque vous utilisez un modèle ARM pour créer la spec de modèle, le modèle est incorporé dans la définition de la ressource. Vous devez apporter certaines modifications à votre modèle local. Copiez le modèle suivant et enregistrez-le localement sous **azuredeploy.json**.

    > [!NOTE]
    > Dans le modèle incorporé, toutes les [expressions de modèle](template-expressions.md) doivent être échappées au moyen d’un deuxième crochet gauche. Utilisez `"[[` au lieu de `"[`. Les tableaux JSON utilisent toujours un seul crochet gauche.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2019-06-01-preview",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2019-06-01-preview",
              "name": "1.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "template": {
                  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                  "contentVersion": "1.0.0.0",
                  "parameters": {
                    "storageAccountType": {
                      "type": "string",
                      "defaultValue": "Standard_LRS",
                      "allowedValues": [
                        "Standard_LRS",
                        "Standard_GRS",
                        "Standard_ZRS",
                        "Premium_LRS"
                      ],
                      "metadata": {
                        "description": "Storage Account type"
                      }
                    },
                    "location": {
                      "type": "string",
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat('store', uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Utilisez Azure CLI ou PowerShell pour créer un groupe de ressources.

    ```azurepowershell
    New-AzResourceGroup `
      -Name templateSpecRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name templateSpecRG \
      --location westus2
    ```

1. Déployez votre modèle avec Azure CLI ou PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

---

## <a name="deploy-template-spec"></a>Déployer une spec de modèle

Pour déployer une spec de modèle, utilisez les mêmes commandes de déploiement que celles que vous utiliseriez pour déployer un modèle. Transmettez l’ID de ressource de la spec de modèle à déployer.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Créez un groupe de ressources pour accueillir le nouveau compte de stockage.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

1. Obtenez l’ID de ressource de la spec de modèle.

    ```azurepowershell
    $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "1.0").Versions.Id
    ```

1. Déployez la spec de modèle.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG
    ```

1. Vous fournissez les paramètres exactement comme vous le feriez pour un modèle ARM. Redéployez la spec de modèle avec un paramètre pour le type de compte de stockage.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -TemplateSpecId $id `
      -ResourceGroupName storageRG `
      -storageAccountType Standard_GRS
    ```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

1. Créez un groupe de ressources pour accueillir le nouveau compte de stockage.

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Obtenez l’ID de ressource de la spec de modèle.

    ```azurecli
    id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "1.0" --query "id")
    ```

    > [!NOTE]
    > Il existe un problème connu lié à l’obtention de l’ID de la spec de modèle et à son attribution à une variable dans Windows PowerShell.

1. Déployez la spec de modèle.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id
    ```

1. Vous fournissez les paramètres exactement comme vous le feriez pour un modèle ARM. Redéployez la spec de modèle avec un paramètre pour le type de compte de stockage.

    ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters storageAccountType='Standard_GRS'
    ```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Sélectionnez la spec de modèle que vous avez créée.

    :::image type="content" source="./media/quickstart-create-template-specs/select-template-spec.png" alt-text="sélection de specs de modèle":::

1. Sélectionnez **Déployer**.

    :::image type="content" source="./media/quickstart-create-template-specs/deploy-template-spec.png" alt-text="déploiement de specs de modèle":::

1. Renseignez les valeurs suivantes :

    - **Abonnement** : sélectionnez un abonnement Azure pour créer la ressource.
    - **Groupe de ressources**  : sélectionnez **Créer nouveau**, puis entrez **storageRG**.
    - **Type de compte de stockage** : sélectionnez **Standard_GRS**.

1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

# <a name="arm-template"></a>[Modèle ARM](#tab/azure-resource-manager)

1. Copiez le modèle suivant et enregistrez-le localement dans un fichier nommé **storage.json**.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-06-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '1.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Utilisez Azure CLI ou PowerShell pour créer un groupe de ressources pour le compte de stockage.

    ```azurepowershell
    New-AzResourceGroup `
      -Name storageRG `
      -Location westus2
    ```

    ```azurecli
    az group create \
      --name storageRG \
      --location westus2
    ```

1. Déployez votre modèle avec Azure CLI ou PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="grant-access"></a>Accorder l'accès

Si vous voulez permettre à d’autres utilisateurs de votre organisation de déployer votre spec de modèle, vous devez leur accorder un accès en lecture. Vous pouvez attribuer le rôle Lecteur à un groupe de Azure AD pour le groupe de ressources contenant les specs de modèle que vous souhaitez partager. Pour plus d’informations, consultez le [Tutoriel : Accorder à un groupe l’accès aux ressources Azure à l’aide d’Azure PowerShell](../../role-based-access-control/tutorial-role-assignments-group-powershell.md).

## <a name="update-template"></a>Mettre à jour le modèle

Supposons que vous avez identifié une modification que vous aimeriez apporter au modèle dans votre spec de modèle. Le modèle suivant est similaire à votre précédent modèle, à ceci près qu’il ajoute un préfixe pour le nom du compte de stockage. Copiez le modèle suivant et mettez à jour votre fichier azuredeploy.json.

:::code language="json" source="~/resourcemanager-templates/azure-resource-manager/templatespecs/azuredeploy2.json":::

## <a name="update-template-spec-version"></a>Mettre à jour la version de spec de modèle

Au lieu de créer une spec de modèle pour le modèle révisé, ajoutez une nouvelle version nommée `2.0` à la spec de modèle existante. Les utilisateurs peuvent choisir l’une ou l’autre version à déployer.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

1. Créez une nouvelle version du spec de modèle.

   ```powershell
   New-AzTemplateSpec `
     -Name storageSpec `
     -Version "2.0" `
     -ResourceGroupName templateSpecRG `
     -Location westus2 `
     -TemplateFile "c:\Templates\azuredeploy.json"
   ```

1. Pour déployer la nouvelle version, récupérez l’ID de ressource pour la version `2.0`.

   ```powershell
   $id = (Get-AzTemplateSpec -ResourceGroupName templateSpecRG -Name storageSpec -Version "2.0").Versions.Id
   ```

1. Déployez cette version. Indiquez le préfixe du nom du compte de stockage.

   ```powershell
   New-AzResourceGroupDeployment `
     -TemplateSpecId $id `
     -ResourceGroupName storageRG `
     -namePrefix "demoaccount"
   ```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/azure-cli)

1. Créez une nouvelle version du spec de modèle.

   ```azurecli
   az ts create \
     --name storageSpec \
     --version "2.0" \
     --resource-group templateSpecRG \
     --location "westus2" \
     --template-file "c:\Templates\azuredeploy.json"
   ```

1. Pour déployer la nouvelle version, récupérez l’ID de ressource pour la version `2.0`.

   ```azurecli
   id=$(az ts show --name storageSpec --resource-group templateSpecRG --version "2.0" --query "id")
   ```

1. Déployez cette version. Indiquez le préfixe du nom du compte de stockage.

   ```azurecli
    az deployment group create \
      --resource-group storageRG \
      --template-spec $id \
      --parameters namePrefix='demoaccount'
    ```

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans votre spec de modèle, sélectionnez **Créer une nouvelle version**.

   :::image type="content" source="./media/quickstart-create-template-specs/select-versions.png" alt-text="création d’une nouvelle version":::

1. Nommez la nouvelle version `2.0` et ajoutez éventuellement des notes. Sélectionnez **Modifier un modèle**.

   :::image type="content" source="./media/quickstart-create-template-specs/add-version-name.png" alt-text="nommage de la nouvelle version":::

1. Remplacez le contenu du modèle par votre modèle mis à jour. Sélectionnez **Vérifier + enregistrer**.
1. Sélectionnez **Enregistrer les modifications**.

1. Pour déployer la nouvelle version, sélectionnez **Versions**.

   :::image type="content" source="./media/quickstart-create-template-specs/see-versions.png" alt-text="liste des versions":::

1. Pour la version que vous souhaitez déployer, sélectionnez les trois points, puis **Déployer**.

   :::image type="content" source="./media/quickstart-create-template-specs/deploy-version.png" alt-text="sélection de la version à déployer":::

1. Renseignez les champs comme vous l’avez fait lors du déploiement de la version antérieure.
1. Sélectionnez **Revoir + créer**.
1. Sélectionnez **Create** (Créer).

# <a name="arm-template"></a>[Modèle ARM](#tab/azure-resource-manager)

1. Là encore, vous devez apporter quelques modifications à votre modèle local pour qu’il fonctionne avec des specs de modèle. Copiez le modèle suivant et enregistrez-le localement sous azuredeploy.json.

   ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/templateSpecs",
          "apiVersion": "2019-06-01-preview",
          "name": "storageSpec",
          "location": "westus2",
          "properties": {
            "displayName": "Storage template spec"
          },
          "tags": {},
          "resources": [
            {
              "type": "versions",
              "apiVersion": "2019-06-01-preview",
              "name": "2.0",
              "location": "westus2",
              "dependsOn": [ "storageSpec" ],
              "properties": {
                "template": {
                  "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
                  "contentVersion": "1.0.0.0",
                  "parameters": {
                    "storageAccountType": {
                      "type": "string",
                      "defaultValue": "Standard_LRS",
                      "allowedValues": [
                        "Standard_LRS",
                        "Standard_GRS",
                        "Standard_ZRS",
                        "Premium_LRS"
                      ],
                      "metadata": {
                        "description": "Storage Account type"
                      }
                    },
                    "location": {
                      "type": "string",
                      "defaultValue": "[[resourceGroup().location]",
                      "metadata": {
                        "description": "Location for all resources."
                      }
                    },
                    "namePrefix": {
                      "type": "string",
                      "maxLength": 11,
                      "defaultValue": "store",
                      "metadata": {
                        "description": "Prefix for storage account name"
                      }
                    }
                  },
                  "variables": {
                    "storageAccountName": "[[concat(parameters('namePrefix'), uniquestring(resourceGroup().id))]"
                  },
                  "resources": [
                    {
                      "type": "Microsoft.Storage/storageAccounts",
                      "apiVersion": "2019-04-01",
                      "name": "[[variables('storageAccountName')]",
                      "location": "[[parameters('location')]",
                      "sku": {
                        "name": "[[parameters('storageAccountType')]"
                      },
                      "kind": "StorageV2",
                      "properties": {}
                    }
                  ],
                  "outputs": {
                    "storageAccountName": {
                      "type": "string",
                      "value": "[[variables('storageAccountName')]"
                    }
                  }
                }
              },
              "tags": {}
            }
          ]
        }
      ],
      "outputs": {}
    }
    ```

1. Pour ajouter la nouvelle version à votre spec de modèle, déployez votre modèle avec Azure CLI ou PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName templateSpecRG `
      -TemplateFile "c:\Templates\azuredeploy.json"
    ```

    ```azurecli
    az deployment group create \
      --name templateSpecRG \
      --template-file "c:\Templates\azuredeploy.json"
    ```

1. Copiez le modèle suivant et enregistrez-le localement dans un fichier nommé **storage.json**.

    ```json
    {
      "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
      "contentVersion": "1.0.0.0",
      "parameters": {},
      "functions": [],
      "variables": {},
      "resources": [
        {
          "type": "Microsoft.Resources/deployments",
          "apiVersion": "2020-06-01",
          "name": "demo",
          "properties": {
            "templateLink": {
              "id": "[resourceId('templateSpecRG', 'Microsoft.Resources/templateSpecs/versions', 'storageSpec', '2.0')]"
            },
            "parameters": {
            },
            "mode": "Incremental"
          }
        }
      ],
      "outputs": {}
    }
    ```

1. Déployez votre modèle avec Azure CLI ou PowerShell.

    ```azurepowershell
    New-AzResourceGroupDeployment `
      -ResourceGroupName storageRG `
      -TemplateFile "c:\Templates\storage.json"
    ```

    ```azurecli
    az deployment group create \
      --name storageRG \
      --template-file "c:\Templates\storage.json"
    ```

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour nettoyer la ressource que vous avez déployée dans le cadre de ce démarrage rapide, supprimez les deux groupes de ressources que vous avez créés.

1. Dans le portail Azure, sélectionnez Groupe de ressources dans le menu de gauche.

1. Entrez le nom du groupe de ressources (templateSpecRG et storageRG) dans le champ Filtrer par nom.

1. Sélectionnez le nom du groupe de ressources.

1. Sélectionnez Supprimer le groupe de ressources dans le menu supérieur.

## <a name="next-steps"></a>Étapes suivantes

Pour découvrir la création d’une spec de modèle incluant des modèles liés, consultez [Créer une spec de modèle d’un modèle lié](template-specs-create-linked.md).

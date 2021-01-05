---
title: 'Démarrage rapide : Créer un blueprint avec Azure CLI'
description: Dans ce guide de démarrage rapide, vous allez utiliser Azure Blueprints pour créer, définir et déployer des artefacts avec Azure CLI.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: 88ccd3fe6271a79d201007954e7997b68d2dc69b
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94920557"
---
# <a name="quickstart-define-and-assign-an-azure-blueprint-with-azure-cli"></a>Démarrage rapide : Définir et affecter un blueprint Azure avec Azure CLI

Un utilisateur qui sait comment créer et affecter des blueprints peut définir des modèles courants et développer des configurations réutilisables et rapides à déployer en fonction de modèles Azure Resource Manager (modèles ARM), de stratégies, d’exigences en matière de sécurité, etc. Dans ce tutoriel, vous allez découvrir comment utiliser Azure Blueprint pour effectuer des tâches courantes liées à la création, à la publication et à l’affectation d’un blueprint dans votre organisation, notamment :

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-blueprint-extension"></a>Ajouter l’extension Blueprint

Pour permettre à Azure CLI de gérer les affectations et définitions de blueprints, vous devez ajouter l’extension.
Cette extension fonctionne avec Azure CLI quel que soit l’endroit où vous l’utilisez : [bash sur Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (en version autonome et à l’intérieur du portail), [image Docker Azure CLI](https://hub.docker.com/_/microsoft-azure-cli) ou installation locale.

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.0.76** au minimum). S’il n’est pas installé, suivez [ces instructions](/cli/azure/install-azure-cli-windows).

1. Dans l’environnement Azure CLI de choix, vous devez l’importer avec la commande suivante :

   ```azurecli-interactive
   # Add the Blueprint extension to the Azure CLI environment
   az extension add --name blueprint
   ```

1. Vérifiez que l’extension a été installée et qu’il s’agit de la version attendue (au moins **0.1.0**) :

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for extension options
   az blueprint -h
   ```

## <a name="create-a-blueprint"></a>Créer un blueprint

La première étape de la définition d’un modèle standard à des fins de conformité est de composer un blueprint à partir des ressources disponibles. Nous allons créer un blueprint nommé « MyBlueprint » pour configurer les attributions de rôle et de stratégie pour l’abonnement. Ensuite, nous allons ajouter un groupe de ressources, un modèle Resource Manager et une attribution de rôle sur le groupe de ressources.

> [!NOTE]
> Quand Azure CLI est utilisé, l’objet _blueprint_ est créé en premier. Pour chaque _artefact_ à ajouter contenant des paramètres, vous devez définir ces paramètres à l’avance sur le _blueprint_ initial.

1. Créez l’objet _blueprint_ initial. Le paramètre **parameters** prend un fichier JSON qui inclut tous les paramètres de niveau blueprint. Les paramètres sont définis durant l’affectation et utilisés par les artefacts ajoutés aux étapes ultérieures.

   - Fichier JSON - blueprintparms.json

     ```json
     {
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
                "displayName": "storage account type.",
                "description": null
            }
        },
        "tagName": {
            "type": "string",
            "metadata": {
                "displayName": "The name of the tag to provide the policy assignment.",
                "description": null
            }
        },
        "tagValue": {
            "type": "string",
            "metadata": {
                "displayName": "The value of the tag to provide the policy assignment.",
                "description": null
            }
        },
        "contributors": {
            "type": "array",
            "metadata": {
                "description": "List of AAD object IDs that is assigned Contributor role at the subscription",
                "strongType": "PrincipalId"
            }
        },
        "owners": {
            "type": "array",
            "metadata": {
                "description": "List of AAD object IDs that is assigned Owner role at the resource group",
                "strongType": "PrincipalId"
            }
        }
     }
     ```

   - Commande Azure CLI

     ```azurecli-interactive
     # Login first with az login if not using Cloud Shell

     # Create the blueprint object
     az blueprint create \
        --name 'MyBlueprint' \
        --description 'This blueprint sets tag policy and role assignment on the subscription, creates a ResourceGroup, and deploys a resource template and role assignment to that ResourceGroup.' \
        --parameters blueprintparms.json
     ```

     > [!NOTE]
     > Utilisez le nom de fichier _blueprint.json_ quand vous importez vos définitions de blueprint.
     > Ce nom de fichier est utilisé lors de l’appel de [az blueprint import](/cli/azure/ext/blueprint/blueprint#ext_blueprint_az_blueprint_import).

     Par défaut, l’objet blueprint est créé dans l’abonnement par défaut. Pour spécifier le groupe d’administration, utilisez le paramètre **managementgroup**. Pour spécifier l’abonnement, utilisez le paramètre **subscription**.

1. Ajoutez le groupe de ressources pour les artefacts de stockage à la définition.

   ```azurecli-interactive
   az blueprint resource-group add \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'storageRG' \
      --description 'Contains the resource template deployment and a role assignment.'
   ```

1. Ajoutez une attribution de rôle au niveau de l’abonnement. Dans l’exemple ci-dessous, les identités de principal ayant reçu le rôle spécifié sont configurées avec un paramètre qui est défini durant l’affectation du blueprint. Cet exemple utilise le rôle intégré _Contributeur_ avec le GUID `b24988ac-6180-42a0-ab88-20f7382dd24c`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleContributor' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/b24988ac-6180-42a0-ab88-20f7382dd24c' \
      --principal-ids "[parameters('contributors')]"
   ```

1. Ajoutez une affectation de stratégie au niveau de l’abonnement. Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Fichier JSON – artifacts\policyTags.json

     ```json
     {
        "tagName": {
           "value": "[parameters('tagName')]"
        },
        "tagValue": {
           "value": "[parameters('tagValue')]"
        }
     }
     ```

   - Commande Azure CLI

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply tag and its default value to resource groups' \
        --description 'Apply tag and its default value to resource groups' \
        --parameters artifacts\policyTags.json
     ```

1. Ajoutez une autre affectation de stratégie pour l’étiquette Storage (réutilisation du paramètre _storageAccountType_) au niveau de l’abonnement. Cet artefact d’affectation de stratégie supplémentaire montre qu’un paramètre défini sur le blueprint peut être utilisé par plusieurs artefacts. Dans l’exemple, l’élément **storageAccountType** sert à définir une étiquette sur le groupe de ressources. Cette valeur fournit des informations sur le compte de stockage créé à l’étape suivante. Cet exemple utilise la stratégie intégrée _Appliquer l’étiquette et sa valeur par défaut aux groupes de ressources_ avec le GUID `49c88fc8-6fd1-46fd-a676-f12d1d3a4c71`.

   - Fichier JSON – artifacts\policyStorageTags.json

     ```json
     {
        "tagName": {
           "value": "StorageType"
        },
        "tagValue": {
           "value": "[parameters('storageAccountType')]"
        }
     }
     ```

   - Commande Azure CLI

     ```azurecli-interactive
     az blueprint artifact policy create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'policyStorageTags' \
        --policy-definition-id '/providers/Microsoft.Authorization/policyDefinitions/49c88fc8-6fd1-46fd-a676-f12d1d3a4c71' \
        --display-name 'Apply storage tag to resource group' \
        --description 'Apply storage tag and the parameter also used by the template to resource groups' \
        --parameters artifacts\policyStorageTags.json
     ```

1. Ajoutez le modèle sous le groupe de ressources. Le paramètre **template** d’un modèle Resource Manager inclut le composant JSON normal du modèle. Le modèle réutilise également les paramètres de blueprint **storageAccountType**, **tagName** et **tagValue** en transmettant chacun au modèle. Les paramètres de blueprint sont accessibles au modèle en utilisant le paramètre **parameters** et dans le fichier JSON, cette paire clé/valeur est utilisée pour injecter la valeur. Les noms de paramètres de blueprint et de modèle peuvent être les mêmes.

   - Fichier de modèle JSON Resource Manager – artifacts\templateStorage.json

     ```json
     {
         "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
         "contentVersion": "1.0.0.0",
         "parameters": {
             "storageAccountTypeFromBP": {
                 "type": "string",
                 "metadata": {
                     "description": "Storage Account type"
                 }
             },
             "tagNameFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag name from blueprint"
                 }
             },
             "tagValueFromBP": {
                 "type": "string",
                 "defaultValue": "NotSet",
                 "metadata": {
                     "description": "Tag value from blueprint"
                 }
             }
         },
         "variables": {
             "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
         },
         "resources": [{
             "type": "Microsoft.Storage/storageAccounts",
             "name": "[variables('storageAccountName')]",
             "apiVersion": "2016-01-01",
             "tags": {
                 "[parameters('tagNameFromBP')]": "[parameters('tagValueFromBP')]"
             },
             "location": "[resourceGroup().location]",
             "sku": {
                 "name": "[parameters('storageAccountTypeFromBP')]"
             },
             "kind": "Storage",
             "properties": {}
         }],
         "outputs": {
             "storageAccountSku": {
                 "type": "string",
                 "value": "[variables('storageAccountName')]"
             }
         }
     }
     ```

   - Fichier de paramètres de modèle JSON Resource Manager – artifacts\templateStorageParams.json

     ```json
     {
        "storageAccountTypeFromBP": {
           "value": "[parameters('storageAccountType')]"
        },
        "tagNameFromBP": {
           "value": "[parameters('tagName')]"
        },
        "tagValueFromBP": {
           "value": "[parameters('tagValue')]"
        }
     }
     ```

   - Commande Azure CLI

     ```azurecli-interactive
     az blueprint artifact template create \
        --blueprint-name 'MyBlueprint' \
        --artifact-name 'templateStorage' \
        --template artifacts\templateStorage.json \
        --parameters artifacts\templateStorageParams.json \
        --resource-group-art 'storageRG'
     ```

1. Ajoutez une attribution de rôle sous le groupe de ressources. Comme pour l’entrée d’attribution de rôle précédente, l’exemple ci-dessous utilise l’identificateur de définition pour le rôle **Propriétaire** et lui fournit un paramètre différent à partir du blueprint. Cet exemple utilise le rôle intégré _Propriétaire_ avec le GUID `8e3af657-a8ff-443c-a75c-2fe8c4bcb635`.

   ```azurecli-interactive
   az blueprint artifact role create \
      --blueprint-name 'MyBlueprint' \
      --artifact-name 'roleOwner' \
      --role-definition-id '/providers/Microsoft.Authorization/roleDefinitions/8e3af657-a8ff-443c-a75c-2fe8c4bcb635' \
      --principal-ids "[parameters('owners')]" \
      --resource-group-art 'storageRG'
   ```

## <a name="publish-a-blueprint"></a>Publier un blueprint

Maintenant que les artefacts ont été ajoutés au blueprint, il est temps de le publier. Une fois un blueprint publié, il peut être affecté à un abonnement.

```azurecli-interactive
az blueprint publish --blueprint-name 'MyBlueprint' --version '{BlueprintVersion}'
```

La valeur de `{BlueprintVersion}` est une chaîne de lettres, de chiffres et de traits d’union qui ne doit pas dépasser 20 caractères et qui ne doit pas contenir d’espaces ou d’autres caractères spéciaux. Utilisez un nom unique et informatif tel que **v20200605-135541**.

## <a name="assign-a-blueprint"></a>Affecter un blueprint

Une fois qu’un blueprint a été publié à l’aide d’Azure CLI, il peut être affecté à un abonnement. Affectez le blueprint que vous avez créé à l’un des abonnements sous votre hiérarchie de groupes d’administration. Si le blueprint est enregistré dans un abonnement, il ne peut être attribué qu’à cet abonnement. Le paramètre **blueprint-name** spécifie le blueprint à affecter. Pour fournir les paramètres name, location, identity, lock et blueprint, utilisez les paramètres Azure CLI correspondants dans la commande `az blueprint assignment create` ou fournissez-les dans le fichier JSON **parameters**.

1. Exécutez le déploiement du blueprint en l’affectant à un abonnement. Du fait que les paramètres **contributors** et **owners** nécessitent l’octroi de l’attribution de rôle à un tableau d’ID d’objet des principaux, utilisez [l’API Graph Azure Active Directory](../../active-directory/develop/active-directory-graph-api.md) pour collecter les ID d’objet à utiliser dans les **paramètres** pour vos propres utilisateurs, groupes ou principaux de service.

   - Fichier JSON – blueprintAssignment.json

     ```json
     {
        "storageAccountType": {
            "value": "Standard_GRS"
        },
        "tagName": {
            "value": "CostCenter"
        },
        "tagValue": {
            "value": "ContosoIT"
        },
        "contributors": {
            "value": [
                "7be2f100-3af5-4c15-bcb7-27ee43784a1f",
                "38833b56-194d-420b-90ce-cff578296714"
            ]
        },
        "owners": {
            "value": [
                "44254d2b-a0c7-405f-959c-f829ee31c2e7",
                "316deb5f-7187-4512-9dd4-21e7798b0ef9"
            ]
        }
     }
     ```

   - Commande Azure CLI

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

   - Identité managée affectée par l’utilisateur

     Une affectation de blueprint peut également utiliser une [identité managée affectée par l’utilisateur](../../active-directory/managed-identities-azure-resources/overview.md).
     Dans ce cas, le paramètre **identity-type** est défini sur _UserAssigned_ et le paramètre **user-assigned-identities** spécifie l’identité. Remplacez `{userIdentity}` par le nom de l’identité managée affectée par l’utilisateur.

     ```azurecli-interactive
     az blueprint assignment create \
        --name 'assignMyBlueprint' \
        --location 'westus' \
        --identity-type UserAssigned \
        --user-assigned-identities {userIdentity} \
        --resource-group-value artifact_name=storageRG name=StorageAccount location=eastus \
        --parameters blueprintAssignment.json
     ```

     L’**identité managée affectée par l’utilisateur** peut être dans n’importe quels abonnement et groupe de ressources pour lesquels l’utilisateur qui affecte le blueprint a les autorisations nécessaires.

     > [!IMPORTANT]
     > Azure Blueprints ne gère pas l’identité managée affectée par l’utilisateur. Les utilisateurs doivent attribuer des rôles et autorisations suffisants. À défaut, l’affectation du blueprint échouera.

## <a name="clean-up-resources"></a>Nettoyer les ressources

### <a name="unassign-a-blueprint"></a>Annuler l’affectation d’un blueprint

Vous pouvez supprimer un blueprint d’un abonnement. La suppression est souvent effectuée lorsque les ressources d’artefact ne sont plus nécessaires. Quand un blueprint est supprimé, les artefacts affectés dans le cadre de ce blueprint sont abandonnés. Pour supprimer une affectation de blueprint, utilisez la commande `az blueprint assignment delete` :

```azurecli-interactive
az blueprint assignment delete --name 'assignMyBlueprint'
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé, affecté et supprimé un blueprint à l’aide d’Azure CLI. Pour plus d’informations sur Azure Blueprints, consultez l’article concernant le cycle de vie des blueprints.

> [!div class="nextstepaction"]
> [En savoir plus sur le cycle de vie des blueprints](./concepts/lifecycle.md)
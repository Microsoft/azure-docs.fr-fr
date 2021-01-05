---
title: Déploiement de ressources avec le modèle et l’API REST
description: Utilisez Azure Resource Manager et l’API REST Resource Manager pour déployer des ressources sur Azure. Les ressources sont définies dans un modèle Resource Manager.
ms.topic: conceptual
ms.date: 10/22/2020
ms.openlocfilehash: d1c8a365153007d3337d922bc163ba3767eeddc9
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675414"
---
# <a name="deploy-resources-with-arm-templates-and-azure-resource-manager-rest-api"></a>Déployer des ressources avec des modèles Resource Manager et l’API REST Azure Resource Manager

Cet article explique comment utiliser l’API REST Azure Resource Manager avec des modèles Azure Resource Manager (modèles ARM) pour déployer vos ressources sur Azure.

Vous pouvez inclure votre modèle dans le corps de la requête ou un lien vers un fichier. Si vous utilisez un fichier, il peut s’agir d’un fichier local ou d’un fichier externe disponible par le biais d’un URI. Lorsque votre modèle se trouve dans un compte de stockage, vous pouvez restreindre l’accès au modèle et fournir un jeton de signature d’accès partagé (SAP) au cours du déploiement.

## <a name="deployment-scope"></a>Étendue du déploiement

Vous pouvez cibler votre déploiement au niveau d’un groupe de ressources, d’un abonnement Azure, d’un groupe d’administration ou d’un locataire. Les commandes à utiliser diffèrent en fonction de l’étendue du déploiement.

* Pour un déploiement dans un **groupe de ressources** , utilisez [Déploiements - Créer](/rest/api/resources/deployments/createorupdate). La requête est envoyée à :

  ```HTTP
  PUT https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-06-01
  ```

* Pour un déploiement dans un **abonnement** , utilisez [Déploiements - Créer au niveau de l’abonnement](/rest/api/resources/deployments/createorupdateatsubscriptionscope). La requête est envoyée à :

  ```HTTP
  PUT https://management.azure.com/subscriptions/{subscriptionId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-06-01
  ```

  Pour plus d’informations sur les déploiements au niveau de l’abonnement, consultez [Créer des groupes de ressources et des ressources au niveau de l’abonnement](deploy-to-subscription.md).

* Pour un déploiement dans un **groupe d’administration** , utilisez [Déploiements - Créer au niveau du groupe d’administration](/rest/api/resources/deployments/createorupdateatmanagementgroupscope). La requête est envoyée à :

  ```HTTP
  PUT https://management.azure.com/providers/Microsoft.Management/managementGroups/{groupId}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-06-01
  ```

  Pour plus d’informations sur les déploiements au niveau du groupe d’administration, consultez [Créer des ressources au niveau du groupe d’administration](deploy-to-management-group.md).

* Pour effectuer un déploiement sur un **locataire** , utilisez [Déploiements – Créer ou mettre à jour au niveau du locataire](/rest/api/resources/deployments/createorupdateattenantscope). La requête est envoyée à :

  ```HTTP
  PUT https://management.azure.com/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2020-06-01
  ```

  Pour plus d’informations sur les déploiements au niveau d’un locataire, consultez [Créer des ressources au niveau du locataire](deploy-to-tenant.md).

Les exemples de cet article illustrent des déploiements dans des groupes de ressources.

## <a name="deploy-with-the-rest-api"></a>Déployer avec l’API REST

1. Définissez [des en-têtes et des paramètres communs](/rest/api/azure/), y compris des jetons d’authentification.

1. Si vous effectuez un déploiement vers un groupe de ressources qui n’existe pas, vous devez commencer par créer ce dernier. Fournissez votre ID abonnement, le nom du groupe de ressources et l’emplacement dont vous avez besoin pour votre solution. Pour plus d’informations, consultez [Créer un groupe de ressources](/rest/api/resources/resourcegroups/createorupdate).

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>?api-version=2020-06-01
   ```

   Avec un corps de requête comme suit :

   ```json
   {
    "location": "West US",
    "tags": {
      "tagname1": "tagvalue1"
    }
   }
   ```

1. Avant de déployer votre modèle, vous pouvez obtenir un aperçu des changements que le modèle apportera à votre environnement. Utilisez l’[opération de simulation](template-deploy-what-if.md) pour vérifier que le modèle apporte les changements prévus. Cette opération vérifie aussi que le modèle ne comporte pas d’erreurs.

1. Pour déployer un modèle, indiquez votre ID d’abonnement, le nom du groupe de ressources et le nom du déploiement dans l’URI de requête.

   ```HTTP
   PUT https://management.azure.com/subscriptions/<YourSubscriptionId>/resourcegroups/<YourResourceGroupName>/providers/Microsoft.Resources/deployments/<YourDeploymentName>?api-version=2019-10-01
   ```

   Dans le corps de la requête, fournissez un lien vers votre modèle et le fichier de paramètres. Pour plus d’informations sur le fichier de paramètres, consultez [Créer un fichier de paramètres Resource Manager](parameter-files.md).

   Notez que le **Mode** est défini sur **Incremental (Incrémentiel)** . Pour exécuter un déploiement complet, définissez le paramètre **Mode** sur la valeur **Complete (Terminé)** . Soyez prudent lorsque vous utilisez le mode complet, car vous pouvez supprimer par inadvertance des ressources qui ne sont pas dans votre modèle.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental"
    }
   }
   ```

    Si vous voulez journaliser le contenu de la réponse et/ou le contenu de la demande, incluez `debugSetting` dans la demande.

   ```json
   {
    "properties": {
      "templateLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/template.json",
        "contentVersion": "1.0.0.0"
      },
      "parametersLink": {
        "uri": "http://mystorageaccount.blob.core.windows.net/templates/parameters.json",
        "contentVersion": "1.0.0.0"
      },
      "mode": "Incremental",
      "debugSetting": {
        "detailLevel": "requestContent, responseContent"
      }
    }
   }
   ```

    Vous pouvez configurer votre compte de stockage pour qu’il utilise un jeton de signature d’accès partagé (SAP). Pour plus d’informations, consultez [Délégation de l’accès avec une signature d’accès partagé](/rest/api/storageservices/delegating-access-with-a-shared-access-signature).

    Pour fournir une valeur sensible pour un paramètre (par exemple, un mot de passe), ajoutez cette valeur à un coffre de clés. Récupérez le coffre de clés pendant le déploiement comme indiqué dans l’exemple précédent. Pour plus d’informations, consultez [Passage de valeurs sécurisés pendant le déploiement](key-vault-parameter.md).

1. Au lieu d’effectuer la liaison vers des fichiers pour le modèle et les paramètres, vous pouvez les inclure dans le corps de la requête. L’exemple suivant montre le corps de la requête où sont spécifiés le modèle et ses paramètres :

   ```json
   {
      "properties": {
      "mode": "Incremental",
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
            "defaultValue": "[resourceGroup().location]",
            "metadata": {
              "description": "Location for all resources."
            }
          }
        },
        "variables": {
          "storageAccountName": "[concat(uniquestring(resourceGroup().id), 'standardsa')]"
        },
        "resources": [
          {
            "type": "Microsoft.Storage/storageAccounts",
            "apiVersion": "2018-02-01",
            "name": "[variables('storageAccountName')]",
            "location": "[parameters('location')]",
            "sku": {
              "name": "[parameters('storageAccountType')]"
            },
            "kind": "StorageV2",
            "properties": {}
          }
        ],
        "outputs": {
          "storageAccountName": {
            "type": "string",
            "value": "[variables('storageAccountName')]"
          }
        }
      },
      "parameters": {
        "location": {
          "value": "eastus2"
        }
      }
    }
   }
   ```

1. Pour obtenir l’état du déploiement du modèle, utilisez [Déploiements - Obtenir](/rest/api/resources/deployments/get).

   ```HTTP
   GET https://management.azure.com/subscriptions/{subscriptionId}/resourcegroups/{resourceGroupName}/providers/Microsoft.Resources/deployments/{deploymentName}?api-version=2019-10-01
   ```

## <a name="deployment-name"></a>Nom du déploiement

Vous pouvez attribuer un nom à votre déploiement tel que `ExampleDeployment`.

Chaque fois que vous exécutez un déploiement, une entrée est ajoutée à l’historique de déploiement du groupe de ressources avec le nom du déploiement. Si vous exécutez un autre déploiement et que vous lui attribuez le même nom, l’entrée précédente est remplacée par le déploiement actuel. Si vous souhaitez conserver des entrées uniques dans l’historique de déploiement, attribuez un nom unique à chaque déploiement.

Pour créer un nom unique, vous pouvez assigner un numéro aléatoire. Vous pouvez aussi ajouter une valeur de date.

Si vous exécutez des déploiements simultanés dans le même groupe de ressources avec le même nom de déploiement, seul le dernier déploiement aboutit. Les déploiements de même nom qui n’arrivent pas à terme sont remplacés par le dernier déploiement. Par exemple, si vous exécutez un déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage1` et que, dans le même temps, vous exécutez un autre déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage2`, vous ne déployez qu’un seul compte de stockage. Le compte de stockage qui en résulte est nommé `storage2`.

En revanche, si vous exécutez un déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage1` et que, aussitôt terminé, vous exécutez un autre déploiement nommé `newStorage` qui déploie un compte de stockage nommé `storage2`, vous disposez de deux comptes de stockage : un nommé `storage1` et l’autre nommé `storage2`. Cependant, l’historique de déploiement ne présente qu’une seule entrée.

Quand vous spécifiez un nom unique pour chaque déploiement, vous pouvez les exécuter simultanément sans conflit. Si vous exécutez un déploiement nommé `newStorage1` qui déploie un compte de stockage nommé `storage1` et que, dans le même temps, vous exécutez un autre déploiement nommé `newStorage2` qui déploie un compte de stockage nommé `storage2`, vous disposez de deux comptes de stockage et l’historique de déploiement présente deux entrées.

Pour éviter les conflits lors de déploiements simultanés et faire en sorte que l’historique de déploiement présente des entrées uniques, attribuez un nom unique à chaque déploiement.

## <a name="next-steps"></a>Étapes suivantes

- Pour restaurer un déploiement réussi lorsque vous obtenez une erreur, consultez [Restaurer en cas d’erreur vers un déploiement réussi](rollback-on-error.md).
- Pour spécifier comment gérer les ressources présentes dans le groupe de ressources, mais non définies dans le modèle, consultez [Modes de déploiement Azure Resource Manager](deployment-modes.md).
- Pour plus d’informations sur la gestion des opérations REST asynchrones, consultez [Track asynchronous Azure operations (Suivi des opérations asynchrones Azure)](../management/async-operations.md).
- Pour en savoir plus sur les modèles, voir [Comprendre la structure et la syntaxe des modèles ARM](template-syntax.md).


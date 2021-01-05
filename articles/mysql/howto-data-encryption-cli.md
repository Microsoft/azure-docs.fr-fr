---
title: Chiffrement des données - Azure CLI - Azure Database pour MySQL
description: Découvrez comment configurer et gérer le chiffrement des données pour votre instance Azure Database pour MySQL avec Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 03/30/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 6d9abc67035b4581a028d8e59ef080b4f1ffa5b9
ms.sourcegitcommit: 84e3db454ad2bccf529dabba518558bd28e2a4e6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96519040"
---
# <a name="data-encryption-for-azure-database-for-mysql-by-using-the-azure-cli"></a>Chiffrement des données pour Azure Database pour MySQL avec Azure CLI

Découvrez comment utiliser Azure CLI pour configurer et gérer le chiffrement des données pour votre Azure Database pour MySQL.

## <a name="prerequisites-for-azure-cli"></a>Prérequis pour Azure CLI

* Vous devez avoir un abonnement Azure et être un administrateur de cet abonnement.
* Créez un coffre de clés et une clé à utiliser pour une clé gérée par le client. Activez également la protection de purge et la suppression réversible sur le coffre de clés.

  ```azurecli-interactive
  az keyvault create -g <resource_group> -n <vault_name> --enable-soft-delete true --enable-purge-protection true
  ```

* Dans le coffre de clés Azure créé, créez la clé qui sera utilisée pour le chiffrement des données du serveur Azure Database pour MySQL.

  ```azurecli-interactive
  az keyvault key create --name <key_name> -p software --vault-name <vault_name>
  ```

* Pour que vous puissiez utiliser un coffre de clés existant, celui-ci doit avoir les propriétés suivantes à utiliser en tant que clé gérée par le client :

  * [Suppression réversible](../key-vault/general/soft-delete-overview.md)

    ```azurecli-interactive
    az resource update --id $(az keyvault show --name \ <key_vault_name> -o tsv | awk '{print $1}') --set \ properties.enableSoftDelete=true
    ```

  * [Protégé contre le vidage](../key-vault/general/soft-delete-overview.md#purge-protection)

    ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --enable-purge-protection true
    ```
  * Durée de conservation définie sur 90 jours
  ```azurecli-interactive
    az keyvault update --name <key_vault_name> --resource-group <resource_group_name>  --retention-days 90
    ```

* La clé doit avoir les attributs suivants à utiliser en tant que clé gérée par le client :
  * Aucune date d’expiration
  * Non activée
  * Effectuer les opérations **get**, **wrap** et **unwrap**
  * Attribut recoverylevel défini sur **Récupérable** (cela nécessite l’activation de la suppression réversible avec la période de conservation définie sur 90 jours)
  * Protection contre le vidage activée

Vous pouvez vérifier les attributs de la clé ci-dessus avec la commande suivante :

```azurecli-interactive
az keyvault key show --vault-name <key_vault_name> -n <key_name>
```

## <a name="set-the-right-permissions-for-key-operations"></a>Définir les permissions appropriées pour les opérations sur les clés

1. Il existe deux façons d’obtenir l’identité managée pour votre serveur Azure Database pour MySQL.

   ### <a name="create-an-new-azure-database-for-mysql-server-with-a-managed-identity"></a>Créez un serveur Azure Database pour MySQL avec une identité managée.

   ```azurecli-interactive
   az mysql server create --name -g <resource_group> --location <locations> --storage-size size>  -u <user>-p <pwd> --backup-retention <7> --sku-name <sku name> -geo-redundant-backup <Enabled/Disabled>  --assign-identity
   ```

   ### <a name="update-an-existing-the-azure-database-for-mysql-server-to-get-a-managed-identity"></a>Mettez à jour un serveur Azure Database pour MySQL existant pour obtenir une identité managée.

   ```azurecli-interactive
   az mysql server update --name  <server name>  -g <resource_group> --assign-identity
   ```

2. Définissez les **Autorisations de clé** (**Get**, **Wrap**, **Unwrap**) pour le **Principal** (nom du serveur MySQL).

    ```azurecli-interactive
    az keyvault set-policy --name -g <resource_group> --key-permissions get unwrapKey wrapKey --object-id <principal id of the server>
    ```

## <a name="set-data-encryption-for-azure-database-for-mysql"></a>Configurer le chiffrement des données pour Azure Database pour MySQL

1. Activez le chiffrement des données pour le serveur Azure Database pour MySQL à l’aide de la clé créée dans le coffre de clés Azure.

    ```azurecli-interactive
    az mysql server key create –name  <server name>  -g <resource_group> --kid <key url>
    ```

    URL de la clé :  `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Utilisation du chiffrement des données pour les serveurs de restauration ou réplicas

Une fois Azure Database pour MySQL chiffré à l'aide d'une clé gérée par le client stockée dans Key Vault, toute copie nouvellement créée du serveur est également chiffrée. Vous pouvez effectuer cette nouvelle copie par l’intermédiaire d’une opération de restauration locale ou géographique, ou par le biais d’une opération de réplica (locale ou interrégion). Ainsi, pour un serveur MySQL chiffré, vous pouvez utiliser les étapes suivantes afin de créer un serveur restauré chiffré.

### <a name="creating-a-restoredreplica-server"></a>Création d’un serveur de restauration/réplica

* [Créer un serveur de restauration](howto-restore-server-cli.md) 
* [Créer un serveur réplica en lecture](howto-read-replicas-cli.md) 

### <a name="once-the-server-is-restored-revalidate-data-encryption-the-restored-server"></a>Une fois le serveur restauré, revalidez le chiffrement des données sur le serveur restauré

*   Attribuer une identité pour le serveur réplica
```azurecli-interactive
az mysql server update --name  <server name>  -g <resoure_group> --assign-identity
```

*   Obtenir la clé existante à utiliser pour le serveur de restauration/réplica

```azurecli-interactive
az mysql server key list --name  '<server_name>'  -g '<resource_group_name>'
```

*   Définir la stratégie pour la nouvelle identité du serveur de restauration/réplica
  
```azurecli-interactive
az keyvault set-policy --name <keyvault> -g <resoure_group> --key-permissions get unwrapKey wrapKey --object-id <principl id of the server returned by the step 1>
```

* Revalider le serveur de restauration/réplica avec la clé de chiffrement

```azurecli-interactive
az mysql server key create –name  <server name> -g <resource_group> --kid <key url>
```

## <a name="additional-capability-for-the-key-being-used-for-the-azure-database-for-mysql"></a>Capacité supplémentaire pour la clé utilisée pour Azure Database pour MySQL

### <a name="get-the-key-used"></a>Obtenir la clé utilisée

```azurecli-interactive
az mysql server key show --name  <server name>  -g <resource_group> --kid <key url>
```

URL de la clé : `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

### <a name="list-the-key-used"></a>Répertorier la clé utilisée

```azurecli-interactive
az mysql server key list --name  <server name>  -g <resource_group>
```

### <a name="drop-the-key-being-used"></a>Supprimer la clé utilisée

```azurecli-interactive
az mysql server key delete -g <resource_group> --kid <key url>
```

## <a name="using-an-azure-resource-manager-template-to-enable-data-encryption"></a>Utilisation d’un modèle Azure Resource Manager pour activer le chiffrement des données

Outre le portail Azure, vous pouvez également activer le chiffrement des données sur votre serveur Azure Database pour MySQL à l’aide de modèles Azure Resource Manager pour les serveurs nouveaux et existants.

### <a name="for-a-new-server"></a>Pour un nouveau serveur

Utilisez l’un des modèles Azure Resource Manager précréés pour provisionner le serveur avec le chiffrement des données activé : [Exemple avec chiffrement des données](https://github.com/Azure/azure-mysql/tree/master/arm-templates/ExampleWithDataEncryption)

Ce modèle Azure Resource Manager crée un serveur Azure Database pour MySQL et utilise les valeurs **KeyVault** et **Key** transmises comme paramètres pour activer le chiffrement des données sur le serveur.

### <a name="for-an-existing-server"></a>Pour un serveur existant

Vous pouvez également utiliser des modèles Azure Resource Manager pour activer le chiffrement des données sur vos serveurs Azure Database pour MySQL existants.

* Transmettez l’ID de ressource de la clé Azure Key Vault que vous avez copié sous la propriété `Uri` dans l’objet properties.

* Utilisez *2020-01-01-preview* comme version de l’API.

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string"
    },
    "serverName": {
      "type": "string"
    },
    "keyVaultName": {
      "type": "string",
      "metadata": {
        "description": "Key vault name where the key to use is stored"
      }
    },
    "keyVaultResourceGroupName": {
      "type": "string",
      "metadata": {
        "description": "Key vault resource group name where it is stored"
      }
    },
    "keyName": {
      "type": "string",
      "metadata": {
        "description": "Key name in the key vault to use as encryption protector"
      }
    },
    "keyVersion": {
      "type": "string",
      "metadata": {
        "description": "Version of the key in the key vault to use as encryption protector"
      }
    }
  },
  "variables": {
    "serverKeyName": "[concat(parameters('keyVaultName'), '_', parameters('keyName'), '_', parameters('keyVersion'))]"
  },
  "resources": [
    {
      "type": "Microsoft.DBforMySQL/servers",
      "apiVersion": "2017-12-01",
      "kind": "",
      "location": "[parameters('location')]",
      "identity": {
        "type": "SystemAssigned"
      },
      "name": "[parameters('serverName')]",
      "properties": {
      }
    },
    {
      "type": "Microsoft.Resources/deployments",
      "apiVersion": "2019-05-01",
      "name": "addAccessPolicy",
      "resourceGroup": "[parameters('keyVaultResourceGroupName')]",
      "dependsOn": [
        "[resourceId('Microsoft.DBforMySQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "mode": "Incremental",
        "template": {
          "$schema": "http://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
          "contentVersion": "1.0.0.0",
          "resources": [
            {
              "type": "Microsoft.KeyVault/vaults/accessPolicies",
              "name": "[concat(parameters('keyVaultName'), '/add')]",
              "apiVersion": "2018-02-14-preview",
              "properties": {
                "accessPolicies": [
                  {
                    "tenantId": "[subscription().tenantId]",
                    "objectId": "[reference(resourceId('Microsoft.DBforMySQL/servers/', parameters('serverName')), '2017-12-01', 'Full').identity.principalId]",
                    "permissions": {
                      "keys": [
                        "get",
                        "wrapKey",
                        "unwrapKey"
                      ]
                    }
                  }
                ]
              }
            }
          ]
        }
      }
    },
    {
      "name": "[concat(parameters('serverName'), '/', variables('serverKeyName'))]",
      "type": "Microsoft.DBforMySQL/servers/keys",
      "apiVersion": "2020-01-01-preview",
      "dependsOn": [
        "addAccessPolicy",
        "[resourceId('Microsoft.DBforMySQL/servers', parameters('serverName'))]"
      ],
      "properties": {
        "serverKeyType": "AzureKeyVault",
        "uri": "[concat(reference(resourceId(parameters('keyVaultResourceGroupName'), 'Microsoft.KeyVault/vaults/', parameters('keyVaultName')), '2018-02-14-preview', 'Full').properties.vaultUri, 'keys/', parameters('keyName'), '/', parameters('keyVersion'))]"
      }
    }
  ]
}

```

## <a name="next-steps"></a>Étapes suivantes

 Pour en savoir plus sur le chiffrement des données, consultez [Chiffrement des données Azure Database pour MySQL avec une clé gérée par le client](concepts-data-encryption-mysql.md).

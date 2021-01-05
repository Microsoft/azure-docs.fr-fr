---
title: Chiffrement – Interface de ligne de commande Azure – pour Azure Database pour PostgreSQL – Serveur unique
description: Découvrez comment configurer et gérer le chiffrement des données pour votre serveur unique Azure Database pour PostgreSQL à l’aide de l’interface de ligne de commande Azure.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 03/30/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 757782e8842fbcaca9c8d95ec8086dd5791a817b
ms.sourcegitcommit: 80034a1819072f45c1772940953fef06d92fefc8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/03/2020
ms.locfileid: "93240611"
---
# <a name="data-encryption-for-azure-database-for-postgresql-single-server-by-using-the-azure-cli"></a>Chiffrement des données pour le serveur unique Azure Database pour PostgreSQL avec l’interface de ligne de commande Azure

Découvrez comment utiliser l’interface de ligne de commande Azure pour configurer et gérer le chiffrement des données pour votre serveur unique Azure Database pour PostgreSQL.

## <a name="prerequisites-for-azure-cli"></a>Prérequis pour Azure CLI

* Vous devez avoir un abonnement Azure et être un administrateur de cet abonnement.
* Créez un coffre de clés et une clé à utiliser pour une clé gérée par le client. Activez également la protection de purge et la suppression réversible sur le coffre de clés.

   ```azurecli-interactive
   az keyvault create -g <resource_group> -n <vault_name> --enable-soft-delete true --enable-purge-protection true
   ```

* Dans le coffre de clés Azure créé, créez la clé qui sera utilisée pour le chiffrement des données du serveur Azure Database pour le serveur unique PostgreSQL.

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

* La clé doit avoir les attributs suivants à utiliser en tant que clé gérée par le client :
  * Aucune date d’expiration
  * Non activée
  * Effectuer les opérations **get** , **wrap** et **unwrap**

## <a name="set-the-right-permissions-for-key-operations"></a>Définir les permissions appropriées pour les opérations sur les clés

1. Il existe deux façons d’obtenir l’identité managée pour votre serveur unique Azure Database pour PostgreSQL.

    ### <a name="create-an-new-azure-database-for-postgresql-server-with-a-managed-identity"></a>Créez un serveur Azure Database pour PostgreSQL avec une identité managée.

    ```azurecli-interactive
    az postgres server create --name <server_name> -g <resource_group> --location <location> --storage-size <size>  -u <user> -p <pwd> --backup-retention <7> --sku-name <sku name> --geo-redundant-backup <Enabled/Disabled> --assign-identity
    ```

    ### <a name="update-an-existing-the-azure-database-for-postgresql-server-to-get-a-managed-identity"></a>Mettez à jour un serveur Azure Database pour PostgreSQL existant pour obtenir une identité managée.

    ```azurecli-interactive
    az postgres server update --resource-group <resource_group> --name <server_name> --assign-identity
    ```

2. Définissez les **Autorisations de clé** ( **Get** , **Wrap** , **Unwrap** ) pour le **Principal** , qui représente le nom du serveur unique PostgreSQL.

    ```azurecli-interactive
    az keyvault set-policy --name -g <resource_group> --key-permissions get unwrapKey wrapKey --object-id <principal id of the server>
    ```

## <a name="set-data-encryption-for-azure-database-for-postgresql-single-server"></a>Configurer le chiffrement des données pour le serveur unique Azure Database pour PostgreSQL

1. Activez le chiffrement des données pour le serveur unique Azure Database pour PostgreSQL à l’aide de la clé créée dans le coffre de clés Azure.

    ```azurecli-interactive
    az postgres server key create --name <server_name> -g <resource_group> --kid <key_url>
    ```

    URL de la clé :  `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

## <a name="using-data-encryption-for-restore-or-replica-servers"></a>Utilisation du chiffrement des données pour les serveurs de restauration ou réplicas

Une fois Azure Database pour PostgreSQL Serveur unique chiffré à l'aide d'une clé gérée par le client stockée dans Key Vault, toute copie nouvellement créée du serveur est également chiffrée. Vous pouvez effectuer cette nouvelle copie par l’intermédiaire d’une opération de restauration locale ou géographique, ou par le biais d’une opération de réplica (locale ou interrégion). Ainsi, pour un serveur unique PostgreSQL chiffré, vous pouvez utiliser les étapes suivantes afin de créer un serveur restauré chiffré.

### <a name="creating-a-restoredreplica-server"></a>Création d’un serveur de restauration/réplica

* [Créer un serveur de restauration](howto-restore-server-cli.md)
* [Créer un serveur réplica en lecture](howto-read-replicas-cli.md)

### <a name="once-the-server-is-restored-revalidate-data-encryption-the-restored-server"></a>Une fois le serveur restauré, revalidez le chiffrement des données sur le serveur restauré

*   Attribuer une identité pour le serveur réplica
```azurecli-interactive
az postgres server update --name  <server name>  -g <resoure_group> --assign-identity
```

*   Obtenir la clé existante à utiliser pour le serveur de restauration/réplica

```azurecli-interactive
az postgres server key list --name  '<server_name>'  -g '<resource_group_name>'
```

*   Définir la stratégie pour la nouvelle identité du serveur de restauration/réplica

```azurecli-interactive
az keyvault set-policy --name <keyvault> -g <resoure_group> --key-permissions get unwrapKey wrapKey --object-id <principl id of the server returned by the step 1>
```

* Revalider le serveur de restauration/réplica avec la clé de chiffrement

```azurecli-interactive
az postgres server key create –name  <server name> -g <resource_group> --kid <key url>
```

## <a name="additional-capability-for-the-key-being-used-for-the-azure-database-for-postgresql-single-server"></a>Capacité supplémentaire pour la clé utilisée pour le serveur unique Azure Database pour PostgreSQL

### <a name="get-the-key-used"></a>Obtenir la clé utilisée

```azurecli-interactive
az postgres server key show --name <server name>  -g <resource_group> --kid <key url>
```

URL de la clé : `https://YourVaultName.vault.azure.net/keys/YourKeyName/01234567890123456789012345678901>`

### <a name="list-the-key-used"></a>Répertorier la clé utilisée

```azurecli-interactive
az postgres server key list --name  <server name>  -g <resource_group>
```

### <a name="drop-the-key-being-used"></a>Supprimer la clé utilisée

```azurecli-interactive
az postgres server key delete -g <resource_group> --kid <key url> 
```

## <a name="using-an-azure-resource-manager-template-to-enable-data-encryption"></a>Utilisation d’un modèle Azure Resource Manager pour activer le chiffrement des données

Outre le portail Azure, vous pouvez également activer le chiffrement des données sur votre serveur unique Azure Database pour PostgreSQL à l’aide de modèles Azure Resource Manager pour les serveurs nouveaux et existants.

### <a name="for-a-new-server"></a>Pour un nouveau serveur

Utilisez l’un des modèles Azure Resource Manager précréés pour provisionner le serveur avec le chiffrement des données activé : [Exemple avec chiffrement des données](https://github.com/Azure/azure-postgresql/tree/master/arm-templates/ExampleWithDataEncryption)

Ce modèle Azure Resource Manager crée un serveur unique Azure Database pour PostgreSQL et utilise les valeurs **KeyVault** et **Key** transmises comme paramètres pour activer le chiffrement des données sur le serveur.

### <a name="for-an-existing-server"></a>Pour un serveur existant
Vous pouvez également utiliser des modèles Azure Resource Manager pour activer le chiffrement des données sur vos serveurs uniques Azure Database pour PostgreSQL existants.

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
      "type": "Microsoft.DBforPostgreSQL/servers",
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
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
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
                    "objectId": "[reference(resourceId('Microsoft.DBforPostgreSQL/servers/', parameters('serverName')), '2017-12-01', 'Full').identity.principalId]",
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
      "type": "Microsoft.DBforPostgreSQL/servers/keys",
      "apiVersion": "2020-01-01-preview",
      "dependsOn": [
        "addAccessPolicy",
        "[resourceId('Microsoft.DBforPostgreSQL/servers', parameters('serverName'))]"
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

 Pour en savoir plus sur le chiffrement des données, consultez [Chiffrement des données pour un serveur unique Azure Database pour PostgreSQL avec une clé gérée par le client](concepts-data-encryption-postgresql.md).

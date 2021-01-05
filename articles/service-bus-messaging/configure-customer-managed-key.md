---
title: Configurer votre propre clé pour chiffrer les données Azure Service Bus au repos
description: Cet article vous explique comment configurer votre propre clé pour chiffrer les données Azure Service Bus au repos.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 35680819350582062dd4227c65f9e72ae8b3ee5c
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96489698"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-service-bus-data-at-rest-by-using-the-azure-portal"></a>Configurer des clés gérées par le client pour chiffrer les données Azure Service Bus au repos à l’aide du portail Azure
Azure Service Bus Premium fournit une fonctionnalité de chiffrement des données au repos à l’aide d’Azure Storage Service Encryption (Azure SSE). Service Bus Premium utilise le service Stockage Azure pour stocker les données. Par défaut, toutes les données stockées avec ce service sont chiffrées à l’aide de clés gérées par Microsoft. 

## <a name="overview"></a>Vue d’ensemble
Azure Service Bus prend désormais en charge le chiffrement des données au repos à l’aide de clés gérées par Microsoft ou de clés gérées par le client (Bring Your Own Key ; BYOK). Cette fonctionnalité vous permet de créer, de faire pivoter, de désactiver et de révoquer l’accès aux clés gérées par le client qui sont utilisées pour chiffrer les données Azure Service Bus au repos.

L'activation de la fonctionnalité BYOK sur votre espace de noms ne s'effectue qu'une seule fois.

> [!NOTE]
> Il existe quelques inconvénients à la clé gérée par le client pour le chiffrement côté service. 
>   * Cette fonctionnalité est prise en charge par le niveau de service [Azure Service Bus Premium](service-bus-premium-messaging.md). Elle ne peut pas être activée pour les espaces de noms Service Bus de niveau standard.
>   * Le chiffrement ne peut être activé que pour les espaces de noms nouveaux ou vides. Si l’espace de noms contient des files d’attente ou des rubriques, l’opération de chiffrement échoue.

Vous pouvez utiliser Azure Key Vault pour gérer vos clés et effectuer un audit sur leur utilisation. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. Pour plus d’informations sur le coffre de clés Azure, consultez la page [Présentation du coffre de clés Azure](../key-vault/general/overview.md)

Cet article explique comment configurer un coffre de clés à l'aide de clés gérées par le client via le portail Azure. Pour savoir comment créer un coffre de clés à l’aide du portail Azure, consultez [Démarrage rapide : Créer un coffre de clés Azure Key Vault à l'aide du portail Azure](../key-vault/general/quick-create-portal.md).

> [!IMPORTANT]
> Pour utiliser des clés gérées par le client avec Azure Service Bus, le coffre de clés doit contenir deux propriétés requises configurées. Il s'agit de :  **Suppression réversible** et **Ne pas vider**. Ces propriétés sont activées par défaut lorsque vous créez un coffre de clés dans le portail Azure. Toutefois, si vous devez activer ces propriétés sur un coffre de clés existant, vous devez utiliser PowerShell ou Azure CLI.

## <a name="enable-customer-managed-keys"></a>Activer des clés gérées par le client
Pour activer des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez à votre espace de noms Service Bus Premium.
2. Dans la page **Paramètres** de votre espace de noms Service Bus, sélectionnez **Chiffrement**.
3. Sélectionnez **Clé gérée par le client**, comme indiqué sur l'illustration suivante.

    ![Activer une clé gérée par le client](./media/configure-customer-managed-key/enable-customer-managed-key.png)


## <a name="set-up-a-key-vault-with-keys"></a>Configurer un coffre de clés avec des clés

Après avoir activé une clé gérée par le client, vous devez l’associer à votre espace de noms Azure Service Bus. Service Bus prend uniquement en charge Azure Key Vault. Si vous activez l'option **Chiffrement à l'aide de la clé gérée par le client** dans la section précédente, vous devez importer la clé dans Azure Key Vault. En outre, les fonctionnalités **Suppression réversible** et **Ne pas vider** doivent être configurées pour la clé. Ces paramètres peuvent être configurés à l'aide de [PowerShell](../key-vault/general/key-vault-recovery.md) ou de l'[interface CLI](../key-vault/general/key-vault-recovery.md).

1. Pour créer un coffre de clés, suivez le [guide de démarrage rapide](../key-vault/general/overview.md) d'Azure Key Vault. Pour plus d'informations sur l'importation de clés existantes, consultez [Présentation des clés, des secrets et des certificats](../key-vault/general/about-keys-secrets-certificates.md).
1. Pour activer à la fois la suppression réversible et la protection contre le vidage lors de la création d'un coffre, utilisez la commande [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create).

    ```azurecli-interactive
    az keyvault create --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Pour ajouter la protection contre le vidage à un coffre existant (où la suppression réversible est déjà activée), utilisez la commande [az keyvault update](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-update).

    ```azurecli-interactive
    az keyvault update --name contoso-SB-BYOK-keyvault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Pour créer des clés, procédez comme suit :
    1. Pour créer une clé, sélectionnez **Générer/Importer** dans le menu **Clés** sous **Paramètres**.
        
        ![Sélectionner le bouton Générer/Importer](./media/configure-customer-managed-key/select-generate-import.png)

    1. Définissez **Options** sur **Générer** et donnez un nom à la clé.

        ![Créer une clé](./media/configure-customer-managed-key/create-key.png) 

    1. Vous pouvez maintenant sélectionner cette clé dans la liste déroulante pour l’associer à l’espace de noms Service Bus à des fins de chiffrement. 

        ![Sélectionner une clé dans le coffre de clés](./media/configure-customer-managed-key/select-key-from-key-vault.png)
        > [!NOTE]
        > À des fins de redondance, vous pouvez ajouter jusqu’à 3 clés. Dans le cas où l’une des clés a expiré ou n’est pas accessible, les autres clés seront utilisées pour le chiffrement.
        
    1. Renseignez les détails de la clé, puis cliquez sur **Sélectionner**. Cela permettra de chiffrer les données au repos à l'aide d'une clé gérée par le client sur l'espace de noms. 


    > [!IMPORTANT]
    > Si vous envisagez d’utiliser une clé gérée par le client avec la géorécupération d’urgence, veuillez consulter ce qui suit : 
    >
    > Pour activer le chiffrement au repos avec la clé gérée par le client, une [stratégie d’accès](../key-vault/general/secure-your-key-vault.md) est configurée pour l’identité managée de Service Bus sur le coffre Azure Key Vault spécifié. Cela garantit un accès contrôlé à Azure Key Vault à partir de l’espace de noms Azure Service Bus.
    >
    > Pour cette raison :
    > 
    >   * Si [la géorécupération d’urgence](service-bus-geo-dr.md) est déjà activée pour l’espace de noms Service Bus et que vous cherchez à activer la clé gérée par le client, alors : 
    >     * Brisez l’appairage.
    >     * [Configurez la stratégie d’accès](../key-vault/general/assign-access-policy-portal.md) de l’identité managée pour les espaces de noms principal et secondaire sur le coffre de clés.
    >     * Configurez le chiffrement sur l’espace de noms principal.
    >     * Appairez à nouveau les espaces de noms principal et secondaire.
    > 
    >   * Si vous envisagez d’activer la géorécupération d’urgence sur un espace de noms Service Bus où la clé gérée par le client est déjà configurée, alors :
    >     * [Configurez la stratégie d’accès](../key-vault/general/assign-access-policy-portal.md) de l’identité managée pour l’espace de noms secondaire sur le coffre de clés.
    >     * Appairez les espaces de noms principal et secondaire.


## <a name="rotate-your-encryption-keys"></a>Faire pivoter vos clés de chiffrement

Vous pouvez faire pivoter votre clé dans le coffre de clés à l'aide du mécanisme de rotation d'Azure Key Vault. Des dates d'activation et d'expiration peuvent également être définies pour automatiser la rotation des clés. Le service Service Bus détectera les nouvelles versions des clés et commencera à les utiliser automatiquement.

## <a name="revoke-access-to-keys"></a>Révoquer l'accès aux clés

La révocation de l’accès aux clés de chiffrement ne videra pas les données de Service Bus. Cependant, il sera impossible d’accéder aux données à partir de l’espace de noms Service Bus. Vous pouvez révoquer la clé de chiffrement en appliquant une stratégie d'accès ou en supprimant la clé. Pour en savoir plus sur les stratégies d'accès et sur la sécurisation de votre coffre de clés, consultez [Sécuriser l'accès à un coffre de clés](../key-vault/general/secure-your-key-vault.md).

Une fois la clé de chiffrement révoquée, le service Service Bus devient inutilisable sur l’espace de noms chiffré. Si l’accès à la clé est activé ou si la clé supprimée est restaurée, le service Service Bus choisira la clé afin de vous permettre d’accéder aux données à partir de l’espace de noms Service Bus chiffré.

## <a name="use-resource-manager-template-to-enable-encryption"></a>Utiliser un modèle Resource Manager pour activer le chiffrement
Cette section montre comment effectuer les tâches suivantes à l’aide de **modèles Azure Resource Manager**. 

1. Créez un espace de noms Service Bus **Premium** avec une **identité managée du service**.
2. Créez un **coffre de clés** et accorder l’accès au coffre de clés à l’identité du service. 
3. Mettez à jour l’espace de noms Service Bus avec les informations du coffre de clés (clé/valeur). 


### <a name="create-a-premium-service-bus-namespace-with-managed-service-identity"></a>Créer un espace de noms Service Bus Premium avec une identité managée du service
Cette section vous montre comment créer un espace de noms Azure Service Bus avec une identité managée du service à l’aide d’un modèle Azure Resource Manager et de PowerShell. 

1. Créez un modèle Azure Resource Manager pour créer un espace de noms Service Bus de niveau de service Premium avec une identité managée du service. Nommez le fichier : **CreateServiceBusPremiumNamespace.json** : 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
    
             }
          }
       ],
       "outputs":{
          "ServiceBusNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.ServiceBus/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Créez un fichier de paramètres de modèle nommé : **CreateServiceBusPremiumNamespaceParams.json**. 

    > [!NOTE]
    > Remplacez les valeurs suivantes : 
    > - `<ServiceBusNamespaceName>` : Nom de votre espace de noms Service Bus
    > - `<Location>` : Emplacement de votre espace de noms Service Bus

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    ```
3. Exécutez la commande PowerShell suivante pour déployer le modèle afin de créer un espace de noms Service Bus Premium. Ensuite, récupérez l’ID de l’espace de noms Service Bus pour l’utiliser ultérieurement. Remplacez `{MyRG}` par le nom du groupe de ressources avant d’exécuter la commande.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateServiceBusPremiumNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateServiceBusPremiumNamespace.json -TemplateParameterFile ./CreateServiceBusPremiumNamespaceParams.json
    
    $ServiceBusNamespaceId = $outputs.Outputs["serviceBusNamespaceId"].value
    ```
 
### <a name="grant-service-bus-namespace-identity-access-to-key-vault"></a>Autorisez l’identité d’espace de noms Service Bus à accéder au coffre de clés.

1. Exécutez la commande suivante pour créer un coffre de clés avec la **protection contre le vidage** et la **suppression réversible** activées. 

    ```powershell
    New-AzureRmKeyVault -Name "{keyVaultName}" -ResourceGroupName {RGName}  -Location "{location}" -EnableSoftDelete -EnablePurgeProtection    
    ```
    
    (OU)
    
    Exécutez la commande suivante pour mettre à jour un **coffre de clés existant**. Spécifiez des valeurs pour les noms de groupe de ressources et de coffre de clés avant d’exécuter la commande. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Définissez la stratégie d’accès au coffre de clés afin que l’identité managée de l’espace de noms Service Bus puisse accéder à la valeur de clé dans le coffre de clés. Utilisez l’ID de l’espace de noms Service Bus de la section précédente. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $ServiceBusNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-service-bus-namespace-with-customer-managed-key-from-key-vault"></a>Chiffrer les données dans l’espace de noms Service Bus avec une clé gérée par le client à partir du coffre de clés
Jusqu’à présent, vous avez effectué les étapes suivantes : 

1. Création d’un espace de noms Premium avec une identité managée.
2. Création d’un coffre de clés et octroi de l’accès au coffre de clés à l’identité managée. 

Dans cette étape, vous allez mettre à jour l’espace de noms Service Bus avec des informations du coffre de clés. 

1. Créez un fichier JSON nommé **UpdateServiceBusNamespaceWithEncryption.json** avec le contenu suivant : 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Namespace to be created in cluster."
             }
          },
          "location":{
             "type":"string",
             "defaultValue":"[resourceGroup().location]",
             "metadata":{
                "description":"Specifies the Azure location for all resources."
             }
          },
          "keyVaultUri":{
             "type":"string",
             "metadata":{
                "description":"URI of the KeyVault."
             }
          },
          "keyName":{
             "type":"string",
             "metadata":{
                "description":"KeyName."
             }
          }
       },
       "resources":[
          {
             "type":"Microsoft.ServiceBus/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Premium",
                "tier":"Premium",
                "capacity":1
             },
             "properties":{
                "encryption":{
                   "keySource":"Microsoft.KeyVault",
                   "keyVaultProperties":[
                      {
                         "keyName":"[parameters('keyName')]",
                         "keyVaultUri":"[parameters('keyVaultUri')]"
                      }
                   ]
                }
             }
          }
       ]
    }
    ``` 

2. Créez un fichier de paramètres de modèle : **UpdateServiceBusNamespaceWithEncryptionParams.json**.

    > [!NOTE]
    > Remplacez les valeurs suivantes : 
    > - `<ServiceBusNamespaceName>` : Nom de votre espace de noms Service Bus
    > - `<Location>` : Emplacement de votre espace de noms Service Bus
    > - `<KeyVaultName>` : Nom de votre coffre de clés
    > - `<KeyName>` : Nom de la clé dans le coffre de clés  

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "namespaceName":{
             "value":"<ServiceBusNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          },
          "keyName":{
             "value":"<KeyName>"
          },
          "keyVaultUri":{
             "value":"https://<KeyVaultName>.vault.azure.net"
          }
       }
    }
    ```             
3. Exécutez la commande PowerShell suivante pour déployer le modèle Resource Manager. Remplacez `{MyRG}` par le nom de votre groupe de ressources avant d’exécuter la commande. 

    ```powershell
    New-AzResourceGroupDeployment -Name UpdateServiceBusNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateServiceBusNamespaceWithEncryption.json -TemplateParameterFile ./UpdateServiceBusNamespaceWithEncryptionParams.json
    ```
    

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :
- [Vue d’ensemble de Service Bus](service-bus-messaging-overview.md)
- [Vue d'ensemble de Key Vault](../key-vault/general/overview.md)
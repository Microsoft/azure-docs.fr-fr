---
title: Configurer votre propre clé pour chiffrer les données Azure Event Hubs au repos
description: Cet article vous explique comment configurer votre propre clé pour chiffrer les données Azure Event Hubs au repos.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: 1b0469a2f25b7f2bec2668b6ab33ff99eb1df809
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348209"
---
# <a name="configure-customer-managed-keys-for-encrypting-azure-event-hubs-data-at-rest-by-using-the-azure-portal"></a>Configurer des clés gérées par le client pour chiffrer les données Azure Event Hubs au repos via le portail Azure
Azure Event Hubs fournit une fonctionnalité de chiffrement des données au repos avec Azure Storage Service Encryption (Azure SSE). Event Hubs utilise le service Stockage Azure pour stocker les données. Par défaut, toutes les données stockées avec ce service sont chiffrées à l'aide de clés gérées par Microsoft. 

## <a name="overview"></a>Vue d’ensemble
Azure Event Hubs prend désormais en charge le chiffrement des données au repos à l'aide de clés gérées par Microsoft ou de clés gérées par le client (Bring Your Own Key - BYOK). Cette fonctionnalité vous permet de créer, de faire pivoter, de désactiver et de révoquer l'accès aux clés gérées par le client qui sont utilisées pour chiffrer les données Azure Event Hubs au repos.

L'activation de la fonctionnalité BYOK sur votre espace de noms ne s'effectue qu'une seule fois.

> [!NOTE]
> La fonctionnalité BYOK est prise en charge par les clusters [Event Hubs Dedicated à un seul locataire](event-hubs-dedicated-overview.md). Elle ne peut pas être activée pour les espaces de noms Event Hubs standard.

Vous pouvez utiliser Azure Key Vault pour gérer vos clés et effectuer un audit sur leur utilisation. Vous pouvez créer vos propres clés et les stocker dans un coffre de clés, ou utiliser les API d’Azure Key Vault pour générer des clés. Pour plus d’informations sur le coffre de clés Azure, consultez la page [Présentation du coffre de clés Azure](../key-vault/general/overview.md)

Cet article explique comment configurer un coffre de clés à l'aide de clés gérées par le client via le portail Azure. Pour savoir comment créer un coffre de clés à l’aide du portail Azure, consultez [Démarrage rapide : Créer un coffre de clés Azure Key Vault à l'aide du portail Azure](../key-vault/general/quick-create-portal.md).

> [!IMPORTANT]
> Pour utiliser des clés gérées par le client avec Azure Event Hubs, le coffre de clés doit contenir deux propriétés requises configurées. Il s'agit de :  **Suppression réversible** et **Ne pas vider**. Ces propriétés sont activées par défaut lorsque vous créez un coffre de clés dans le portail Azure. Toutefois, si vous devez activer ces propriétés sur un coffre de clés existant, vous devez utiliser PowerShell ou Azure CLI.

## <a name="enable-customer-managed-keys"></a>Activer des clés gérées par le client
Pour activer des clés gérées par le client dans le portail Azure, procédez comme suit :

1. Accédez à votre cluster Event Hubs Dedicated.
1. Sélectionnez l'espace de noms sur lequel vous souhaitez activer la fonctionnalité BYOK.
1. Sur la page **Paramètres** de votre espace de noms Event Hubs, sélectionnez **Chiffrement**. 
1. Sélectionnez **Clé gérée par le client**, comme indiqué sur l'illustration suivante. 

    ![Activer une clé gérée par le client](./media/configure-customer-managed-key/enable-customer-managed-key.png)

## <a name="set-up-a-key-vault-with-keys"></a>Configurer un coffre de clés avec des clés
Après avoir activé une clé gérée par le client, vous devez l'associer à votre espace de noms Azure Event Hubs. Event Hubs prend uniquement en charge Azure Key Vault. Si vous activez l'option **Chiffrement à l'aide de la clé gérée par le client** dans la section précédente, vous devez importer la clé dans Azure Key Vault. En outre, les fonctionnalités **Suppression réversible** et **Ne pas vider** doivent être configurées pour la clé. Ces paramètres peuvent être configurés à l'aide de [PowerShell](../key-vault/general/key-vault-recovery.md) ou de l'[interface CLI](../key-vault/general/key-vault-recovery.md).

1. Pour créer un coffre de clés, suivez le [guide de démarrage rapide](../key-vault/general/overview.md) d'Azure Key Vault. Pour plus d'informations sur l'importation de clés existantes, consultez [Présentation des clés, des secrets et des certificats](../key-vault/general/about-keys-secrets-certificates.md).
1. Pour activer à la fois la suppression réversible et la protection contre le vidage lors de la création d'un coffre, utilisez la commande [az keyvault create](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-create).

    ```azurecli-interactive
    az keyvault create --name ContosoVault --resource-group ContosoRG --location westus --enable-soft-delete true --enable-purge-protection true
    ```    
1. Pour ajouter la protection contre le vidage à un coffre existant (où la suppression réversible est déjà activée), utilisez la commande [az keyvault update](/cli/azure/keyvault?view=azure-cli-latest#az-keyvault-update).

    ```azurecli-interactive
    az keyvault update --name ContosoVault --resource-group ContosoRG --enable-purge-protection true
    ```
1. Pour créer des clés, procédez comme suit :
    1. Pour créer une clé, sélectionnez **Générer/Importer** dans le menu **Clés** sous **Paramètres**.
        
        ![Sélectionner le bouton Générer/Importer](./media/configure-customer-managed-key/select-generate-import.png)
    1. Définissez **Options** sur **Générer** et donnez un nom à la clé.

        ![Créer une clé](./media/configure-customer-managed-key/create-key.png) 
    1. Vous pouvez maintenant sélectionner cette clé dans la liste déroulante pour l'associer à l'espace de noms Event Hubs à des fins de chiffrement. 

        ![Sélectionner une clé dans le coffre de clés](./media/configure-customer-managed-key/select-key-from-key-vault.png)
    1. Renseignez les détails de la clé, puis cliquez sur **Sélectionner**. Cela permettra de chiffrer les données au repos à l'aide d'une clé gérée par le client sur l'espace de noms. 


## <a name="rotate-your-encryption-keys"></a>Faire pivoter vos clés de chiffrement
Vous pouvez faire pivoter votre clé dans le coffre de clés à l'aide du mécanisme de rotation d'Azure Key Vault. Des dates d'activation et d'expiration peuvent également être définies pour automatiser la rotation des clés. Le service Event Hubs détectera les nouvelles versions des clés et commencera à les utiliser automatiquement.

## <a name="revoke-access-to-keys"></a>Révoquer l'accès aux clés
La révocation de l'accès aux clés de chiffrement ne videra pas les données d'Event Hubs. Mais il sera impossible d'accéder aux données à partir de l'espace de noms Event Hubs. Vous pouvez révoquer la clé de chiffrement en appliquant une stratégie d'accès ou en supprimant la clé. Pour en savoir plus sur les stratégies d'accès et sur la sécurisation de votre coffre de clés, consultez [Sécuriser l'accès à un coffre de clés](../key-vault/general/secure-your-key-vault.md).

Une fois la clé de chiffrement révoquée, le service Event Hubs devient inutilisable sur l'espace de noms chiffré. Si l'accès à la clé est activé ou si la clé de suppression est restaurée, le service Event Hubs choisira la clé afin de vous permettre d'accéder aux données à partir de l'espace de noms Event Hubs chiffré.

## <a name="set-up-diagnostic-logs"></a>Configurer les journaux de diagnostic 
En configurant les journaux de diagnostic pour les espaces de noms BYOK, vous disposez des informations requises sur les opérations lorsqu'un espace de noms est chiffré à l'aide de clés gérées par le client. Ces journaux peuvent être activés et ultérieurement transmis en continu à un hub d'événements, analysés via l'analytique des journaux d'activité ou transmis en continu au stockage pour effectuer des analyses personnalisées. Pour en savoir plus sur les journaux de diagnostic, consultez [Présentation des journaux de diagnostic Azure](../azure-monitor/platform/platform-logs-overview.md).

## <a name="enable-user-logs"></a>Activer les journaux utilisateur
Pour activer les journaux relatifs aux clés gérées par le client, procédez comme suit.

1. Sur le portail Azure, accédez à l'espace de noms pour lequel la fonctionnalité BYOK est activée.
1. Sous **Supervision**, sélectionnez **Paramètres de diagnostic**.

    ![Sélectionner Paramètres de diagnostic](./media/configure-customer-managed-key/select-diagnostic-settings.png)
1. Sélectionnez **+Ajouter le paramètre de diagnostic**. 

    ![Sélectionner Ajouter le paramètre de diagnostic](./media/configure-customer-managed-key/select-add-diagnostic-setting.png)
1. Entrez un **nom** et sélectionnez l'emplacement où vous souhaitez diffuser les journaux.
1. Sélectionnez **CustomerManagedKeyUserLogs** et **Enregistrer**. Cette action active les journaux de la fonctionnalité BYOK sur l'espace de noms.

    ![Sélectionner l'option des journaux utilisateur des clés gérées par le client](./media/configure-customer-managed-key/select-customer-managed-key-user-logs.png)

## <a name="log-schema"></a>Schéma du journal 
Tous les journaux d’activité sont stockés au format JSON (JavaScript Object Notation). Chaque entrée comporte des champs de type chaîne au format décrit dans le tableau suivant. 

| Nom | Description |
| ---- | ----------- | 
| TaskName | Description de la tâche en échec. |
| ActivityId | ID interne utilisé à des fins de suivi. |
| catégorie | Définit la classification de la tâche. Par exemple, si la clé de votre coffre de clés est désactivée, il s'agit d'une catégorie d'informations ou, s'il est impossible d'annuler l'inclusion d'une clé dans un wrapper, il peut s'agir d'une erreur. |
| resourceId | ID de ressource Azure Resource Manager |
| keyVault | Nom complet du coffre de clés. |
| key | Nom de clé utilisé pour chiffrer l'espace de noms Event Hubs. |
| version | Version de la clé utilisée. |
| opération | Opération exécutée sur la clé dans votre coffre de clés. Par exemple, activation/désactivation de la clé ou inclusion/annulation de l'inclusion dans un wrapper. |
| code | Code associé à l'opération. Exemple : Code d'erreur ; 404 signifie que la clé est introuvable. |
| message | Tout message d'erreur associé à l'opération. |

Voici un exemple de journal pour une clé gérée par le client :

```json
{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111-1111-1111-1111-111111111111",
   "category": "error"
   "resourceId": "/SUBSCRIPTIONS/11111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "1111111111111111111111111111111",
   "operation": "wrapKey",
   "code": "404",
   "message": "Key not found: ehbyok0/111111111111111111111111111111",
}



{
   "TaskName": "CustomerManagedKeyUserLog",
   "ActivityId": "11111111111111-1111-1111-1111111111111",
   "category": "info"
   "resourceId": "/SUBSCRIPTIONS/111111111-1111-1111-1111-11111111111/RESOURCEGROUPS/DEFAULT-EVENTHUB-CENTRALUS/PROVIDERS/MICROSOFT.EVENTHUB/NAMESPACES/FBETTATI-OPERA-EVENTHUB",
   "keyVault": "https://mykeyvault.vault-int.azure-int.net",
   "key": "mykey",
   "version": "111111111111111111111111111111",
   "operation": "disable" | "restore",
   "code": "",
   "message": "",
}
```

## <a name="use-resource-manager-template-to-enable-encryption"></a>Utiliser un modèle Resource Manager pour activer le chiffrement
Cette section montre comment effectuer les tâches suivantes à l’aide de **modèles Azure Resource Manager**. 

1. Créez un **espace de noms Event Hubs** avec une identité du service managé.
2. Créez un **coffre de clés** et autorisez l'identité du service à accéder au coffre de clés. 
3. Mettez à jour l’espace de noms Event Hubs avec les informations du coffre de clés (clé/valeur). 


### <a name="create-an-event-hubs-cluster-and-namespace-with-managed-service-identity"></a>Créer un cluster et un espace de noms Event Hubs avec une identité du service managé
Cette section vous montre comment créer un espace de noms Azure Event Hubs avec identité managée du service à l’aide d’un modèle Azure Resource Manager et de PowerShell. 

1. Créez un modèle Resource Manager pour créer un espace de noms Event Hubs avec une identité du service managé. Nommez le fichier : **CreateEventHubClusterAndNamespace.json** : 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
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
          }
       },
       "resources":[
          {
             "type":"Microsoft.EventHub/clusters",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('clusterName')]",
             "location":"[parameters('location')]",
             "sku":{
                "name":"Dedicated",
                "capacity":1
             }
          },
          {
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             },
             "dependsOn":[
                "[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]"
             ]
          }
       ],
       "outputs":{
          "EventHubNamespaceId":{
             "type":"string",
             "value":"[resourceId('Microsoft.EventHub/namespaces',parameters('namespaceName'))]"
          }
       }
    }
    ```
2. Créez un fichier de paramètres de modèle nommé : **CreateEventHubClusterAndNamespaceParams.json**. 

    > [!NOTE]
    > Remplacez les valeurs suivantes : 
    > - `<EventHubsClusterName>` - Nom de votre cluster Event Hubs    
    > - `<EventHubsNamespaceName>` - Nom de votre espace de noms Event Hubs
    > - `<Location>` - Emplacement de votre espace de noms Event Hubs

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
          },
          "location":{
             "value":"<Location>"
          }
       }
    }
    
    ```
3. Exécutez la commande PowerShell suivante pour déployer le modèle afin de créer un espace de noms Event Hubs. Ensuite, récupérez l’ID de l’espace de noms Event Hubs pour l’utiliser ultérieurement. Remplacez `{MyRG}` par le nom du groupe de ressources avant d'exécuter la commande.  

    ```powershell
    $outputs = New-AzResourceGroupDeployment -Name CreateEventHubClusterAndNamespace -ResourceGroupName {MyRG} -TemplateFile ./CreateEventHubClusterAndNamespace.json -TemplateParameterFile ./CreateEventHubClusterAndNamespaceParams.json

    $EventHubNamespaceId = $outputs.Outputs["eventHubNamespaceId"].value
    ```
 
### <a name="grant-event-hubs-namespace-identity-access-to-key-vault"></a>Autorisez l'identité d'espace de noms Event Hubs à accéder au coffre de clés.

1. Exécutez la commande suivante pour créer un coffre de clés avec la **protection contre le vidage** et la **suppression réversible** activées. 

    ```powershell
    New-AzureRmKeyVault -Name {keyVaultName} -ResourceGroupName {RGName}  -Location {location} -EnableSoftDelete -EnablePurgeProtection    
    ```     
    
    (OU)    
    
    Exécutez la commande suivante pour mettre à jour un **coffre de clés existant**. Spécifiez des valeurs pour les noms de groupe de ressources et de coffre de clés avant d’exécuter la commande. 
    
    ```powershell
    ($updatedKeyVault = Get-AzureRmResource -ResourceId (Get-AzureRmKeyVault -ResourceGroupName {RGName} -VaultName {keyVaultName}).ResourceId).Properties| Add-Member -MemberType "NoteProperty" -Name "enableSoftDelete" -Value "true"-Force | Add-Member -MemberType "NoteProperty" -Name "enablePurgeProtection" -Value "true" -Force
    ``` 
2. Définissez la stratégie d’accès au coffre de clés afin que l’identité managée de l’espace de noms Event Hubss puisse accéder à la valeur de clé dans le coffre de clés. Utilisez l’ID de l’espace de noms Event Hubs de la section précédente. 

    ```powershell
    $identity = (Get-AzureRmResource -ResourceId $EventHubNamespaceId -ExpandProperties).Identity
    
    Set-AzureRmKeyVaultAccessPolicy -VaultName {keyVaultName} -ResourceGroupName {RGName} -ObjectId $identity.PrincipalId -PermissionsToKeys get,wrapKey,unwrapKey,list
    ```

### <a name="encrypt-data-in-event-hubs-namespace-with-customer-managed-key-from-key-vault"></a>Chiffrer les données dans l’espace de noms Event Hubs avec une clé gérée par le client à partir du coffre de clés
Jusqu’à présent, vous avez effectué les étapes suivantes : 

1. Création d’un espace de noms Premium avec une identité managée.
2. Création d’un coffre de clés et octroi de l’accès au coffre de clés à l’identité managée. 

Dans cette étape, vous allez mettre à jour l’espace de noms Event Hubs avec des informations du coffre de clés. 

1. Créez un fichier JSON nommé **CreateEventHubClusterAndNamespace.json** avec le contenu suivant : 

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "type":"string",
             "metadata":{
                "description":"Name for the Event Hub cluster."
             }
          },
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
             "type":"Microsoft.EventHub/namespaces",
             "apiVersion":"2018-01-01-preview",
             "name":"[parameters('namespaceName')]",
             "location":"[parameters('location')]",
             "identity":{
                "type":"SystemAssigned"
             },
             "sku":{
                "name":"Standard",
                "tier":"Standard",
                "capacity":1
             },
             "properties":{
                "isAutoInflateEnabled":false,
                "maximumThroughputUnits":0,
                "clusterArmId":"[resourceId('Microsoft.EventHub/clusters', parameters('clusterName'))]",
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

2. Créez un fichier de paramètres de modèle : **UpdateEventHubClusterAndNamespaceParams.json**. 

    > [!NOTE]
    > Remplacez les valeurs suivantes : 
    > - `<EventHubsClusterName>` - Nom de votre cluster Event Hubs        
    > - `<EventHubsNamespaceName>` - Nom de votre espace de noms Event Hubs
    > - `<Location>` - Emplacement de votre espace de noms Event Hubs
    > - `<KeyVaultName>` - Nom de votre coffre de clés
    > - `<KeyName>` : Nom de la clé dans le coffre de clés

    ```json
    {
       "$schema":"https://schema.management.azure.com/schemas/2015-01-01/deploymentParameters.json#",
       "contentVersion":"1.0.0.0",
       "parameters":{
          "clusterName":{
             "value":"<EventHubsClusterName>"
          },
          "namespaceName":{
             "value":"<EventHubsNamespaceName>"
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
3. Exécutez la commande PowerShell suivante pour déployer le modèle Resource Manager. Remplacez `{MyRG}` par le nom de votre groupe de ressources avant d'exécuter la commande. 

    ```powershell
    New-AzResourceGroupDeployment -Name UpdateEventHubNamespaceWithEncryption -ResourceGroupName {MyRG} -TemplateFile ./UpdateEventHubClusterAndNamespace.json -TemplateParameterFile ./UpdateEventHubClusterAndNamespaceParams.json 
    ```

## <a name="troubleshoot"></a>Dépanner
Il est recommandé de toujours activer les journaux, comme indiqué dans la section précédente. Cela permet de suivre les activités lorsque le chiffrement BYOK est activé. Cela permet également d'identifier les problèmes.

Vous trouverez ci-dessous les codes d'erreur courants à rechercher lorsque le chiffrement BYOK est activé.

| Action | Code d'erreur | État consécutif des données |
| ------ | ---------- | ----------------------- | 
| Inclure une autorisation dans un wrapper/Annuler l'inclusion d'une autorisation dans un wrapper à partir d'un coffre de clés | 403 |    Inaccessible |
| Supprimer l'appartenance à un rôle AAD d'un principal AAD qui a accordé l'autorisation d'inclusion/d'annulation de l'inclusion dans un wrapper. | 403 |  Inaccessible |
| Supprimer une clé de chiffrement du coffre de clés | 404 | Inaccessible |
| Supprimer le coffre de clés | 404 | Inaccessible (si la suppression réversible - qui est un paramètre obligatoire - est activée.) |
| Modification de la période d'expiration de la clé de chiffrement pour qu'elle soit déjà obsolète | 403 |   Inaccessible  |
| Modification de la configuration NBF (pas avant) pour que la clé de chiffrement principale ne soit pas active | 403 | Inaccessible  |
| Sélection de l'option **Autoriser les services MSFT** pour le pare-feu du coffre de clés, ou autre type de blocage de l'accès réseau au coffre de clés contenant la clé de chiffrement | 403 | Inaccessible |
| Déplacement du coffre de clés vers un autre locataire | 404 | Inaccessible |  
| Problème réseau intermittent ou interruption de DNS/AAD/MSI |  | Accessible à l'aide d'une clé de chiffrement de données en cache |

> [!IMPORTANT]
> Pour activer la géorécupération d'urgence sur un espace de noms utilisant le chiffrement BYOK, l'espace de noms secondaire destiné à l'appariement doit se trouver dans un cluster dédié, et une identité managée affectée par le système doit y être activée. Pour en savoir plus, consultez [Identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).

## <a name="next-steps"></a>Étapes suivantes
Voir les articles suivants :
- [Vue d’ensemble d’Event Hubs](event-hubs-about.md)
- [Vue d'ensemble de Key Vault](../key-vault/general/overview.md)
---
title: Utiliser des références Key Vault
description: Découvrez comment configurer Azure App Service et Azure Functions pour leur faire utiliser des références Azure Key Vault. Mettez les secrets Key Vault à la disposition de votre code d’application.
author: mattchenderson
ms.topic: article
ms.date: 10/09/2019
ms.author: mahender
ms.custom: seodec18
ms.openlocfilehash: bb220da0b906c9d7a5f45dcc841129e14c7c6c51
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92205844"
---
# <a name="use-key-vault-references-for-app-service-and-azure-functions"></a>Utiliser des références Key Vault pour App Service et Azure Functions

Cette rubrique vous montre comment utiliser des secrets d’Azure Key Vault dans votre application App Service ou Azure Functions, sans avoir à modifier le code. [Azure Key Vault](../key-vault/general/overview.md) est un service qui fournit une gestion centralisée des secrets, avec un contrôle total sur les stratégies d’accès et l’historique d’audit.

## <a name="granting-your-app-access-to-key-vault"></a>Autoriser votre application à accéder à Key Vault

Pour pouvoir lire les secrets dans Key Vault, vous devez créer un coffre et donner à votre application l’autorisation d’y accéder.

1. Créez un coffre de clés en suivant le [Guide de démarrage rapide de Key Vault](../key-vault/secrets/quick-create-cli.md).

1. Créez une [identité managée affectée par le système](overview-managed-identity.md) pour votre application.

   > [!NOTE] 
   > Actuellement, les références Key Vault prennent uniquement en charge les identités managées affectées par le système. Vous ne pouvez pas utiliser d’identités affectées par l’utilisateur.

1. Créez une [stratégie d’accès dans Key Vault](../key-vault/general/secure-your-key-vault.md#key-vault-access-policies) pour l’identité d’application que vous avez créée précédemment. Activez l’autorisation de secret « Get » sur cette stratégie. Ne configurez pas les paramètres « application autorisée » ou `applicationId` car ils sont incompatibles avec une identité managée.

   > [!IMPORTANT]
   > Les références Key Vault ne sont actuellement pas en mesure de résoudre les secrets stockés dans un coffre de clés avec des [restrictions de réseau](../key-vault/general/overview-vnet-service-endpoints.md)à moins que l’application ne soit hébergée au sein [de App Service Environment](./environment/intro.md).

## <a name="reference-syntax"></a>Syntaxe de référence

Une référence Key Vault est de la forme `@Microsoft.KeyVault({referenceString})`, où `{referenceString}` est remplacé par une des options suivantes :

> [!div class="mx-tdBreakAll"]
> | Chaîne de référence                                                            | Description                                                                                                                                                                                 |
> |-----------------------------------------------------------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
> | SecretUri= _URI_secret_                                                       | L’ **URI_secret** doit être l’URI complet du plan de données d’un secret dans Key Vault, y compris une version, par exemple, https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931  |
> | VaultName= _Nom_coffre_ ;SecretName= _Nom_secret_ ;SecretVersion= _Version_secret_ | Le **Nom_coffre** doit être le nom de votre ressource Key Vault. Le **Nom_secret** doit être le nom du secret cible. La **Version_secret** doit être la version du secret à utiliser. |

> [!NOTE] 
> Les versions sont actuellement requises. Pendant la rotation des secrets, vous devez mettre à jour la version dans la configuration de votre application.
Par exemple, une référence complète ressemble à ce qui suit :


```
@Microsoft.KeyVault(SecretUri=https://myvault.vault.azure.net/secrets/mysecret/ec96f02080254f109c51a1f14cdb1931)
```

Sinon :

```
@Microsoft.KeyVault(VaultName=myvault;SecretName=mysecret;SecretVersion=ec96f02080254f109c51a1f14cdb1931)
```


## <a name="source-application-settings-from-key-vault"></a>Paramètres d’application sources dans Key Vault

Les références Key Vault peuvent servir de valeurs pour les [Paramètres d’application](configure-common.md#configure-app-settings), ce qui vous permet de garder des secrets dans Key Vault au lieu de la configuration de site. Les paramètres d’application sont chiffrés au repos de manière sécurisée, mais si vous avez besoin de fonctionnalités de gestion des secrets, ils doivent être gardés dans Key Vault.

Afin d’utiliser une référence Key Vault pour un paramètre d’application, définissez la référence comme valeur du paramètre. Votre application peut référencer le secret par le biais de sa clé comme d’habitude. Le code n’a pas besoin d’être modifié.

> [!TIP]
> La plupart des paramètres d’application qui utilisent des références Key Vault doivent être marqués comme des paramètres d’emplacement, car vous devez avoir des coffres distincts pour chaque environnement.

### <a name="azure-resource-manager-deployment"></a>Déploiement Azure Resource Manager

Quand vous automatisez des déploiements de ressources par le biais de modèles Azure Resource Manager, vous pouvez avoir besoin de séquencer vos dépendances dans un ordre particulier pour que cette fonctionnalité fonctionne. Sinon, vous devez définir vos paramètres d’application comme leur propre ressource, au lieu d’utiliser une propriété `siteConfig` dans la définition de site. C’est parce que le site doit être défini en premier pour que le système puisse affecter l’identité et que cette identité puisse être utilisée dans la stratégie d’accès.

Voici à quoi peut ressembler un exemple de pseudo-modèle d’application de fonction :

```json
{
    //...
    "resources": [
        {
            "type": "Microsoft.Storage/storageAccounts",
            "name": "[variables('storageAccountName')]",
            //...
        },
        {
            "type": "Microsoft.Insights/components",
            "name": "[variables('appInsightsName')]",
            //...
        },
        {
            "type": "Microsoft.Web/sites",
            "name": "[variables('functionAppName')]",
            "identity": {
                "type": "SystemAssigned"
            },
            //...
            "resources": [
                {
                    "type": "config",
                    "name": "appsettings",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), variables('storageConnectionStringName'))]",
                        "[resourceId('Microsoft.KeyVault/vaults/secrets', variables('keyVaultName'), variables('appInsightsKeyName'))]"
                    ],
                    "properties": {
                        "AzureWebJobsStorage": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('storageConnectionStringResourceId')).secretUriWithVersion, ')')]",
                        "WEBSITE_CONTENTAZUREFILECONNECTIONSTRING": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('storageConnectionStringResourceId')).secretUriWithVersion, ')')]",
                        "APPINSIGHTS_INSTRUMENTATIONKEY": "[concat('@Microsoft.KeyVault(SecretUri=', reference(variables('appInsightsKeyResourceId')).secretUriWithVersion, ')')]",
                        "WEBSITE_ENABLE_SYNC_UPDATE_SITE": "true"
                        //...
                    }
                },
                {
                    "type": "sourcecontrols",
                    "name": "web",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]",
                        "[resourceId('Microsoft.Web/sites/config', variables('functionAppName'), 'appsettings')]"
                    ],
                }
            ]
        },
        {
            "type": "Microsoft.KeyVault/vaults",
            "name": "[variables('keyVaultName')]",
            //...
            "dependsOn": [
                "[resourceId('Microsoft.Web/sites', variables('functionAppName'))]"
            ],
            "properties": {
                //...
                "accessPolicies": [
                    {
                        "tenantId": "[reference(concat('Microsoft.Web/sites/',  variables('functionAppName'), '/providers/Microsoft.ManagedIdentity/Identities/default'), '2015-08-31-PREVIEW').tenantId]",
                        "objectId": "[reference(concat('Microsoft.Web/sites/',  variables('functionAppName'), '/providers/Microsoft.ManagedIdentity/Identities/default'), '2015-08-31-PREVIEW').principalId]",
                        "permissions": {
                            "secrets": [ "get" ]
                        }
                    }
                ]
            },
            "resources": [
                {
                    "type": "secrets",
                    "name": "[variables('storageConnectionStringName')]",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.Storage/storageAccounts', variables('storageAccountName'))]"
                    ],
                    "properties": {
                        "value": "[concat('DefaultEndpointsProtocol=https;AccountName=', variables('storageAccountName'), ';AccountKey=', listKeys(variables('storageAccountResourceId'),'2015-05-01-preview').key1)]"
                    }
                },
                {
                    "type": "secrets",
                    "name": "[variables('appInsightsKeyName')]",
                    //...
                    "dependsOn": [
                        "[resourceId('Microsoft.KeyVault/vaults/', variables('keyVaultName'))]",
                        "[resourceId('Microsoft.Insights/components', variables('appInsightsName'))]"
                    ],
                    "properties": {
                        "value": "[reference(resourceId('microsoft.insights/components/', variables('appInsightsName')), '2015-05-01').InstrumentationKey]"
                    }
                }
            ]
        }
    ]
}
```

> [!NOTE] 
> Dans cet exemple, le déploiement du contrôle de code source varie selon les paramètres d’application. Il s’agit normalement d’un comportement non sécurisé, car la mise à jour du paramètre d’application se comporte de façon asynchrone. Toutefois, comme nous avons inclus le paramètre d'application `WEBSITE_ENABLE_SYNC_UPDATE_SITE`, la mise à jour est synchrone. Cela signifie que le déploiement de contrôle source commence uniquement une fois que les paramètres d’application ont été entièrement mis à jour.

## <a name="troubleshooting-key-vault-references"></a>Résolution des problèmes liés aux références Key Vault

Si une référence ne se résout pas correctement, la valeur de référence est utilisée. Ainsi, pour les paramètres d’application, une variable d’environnement dont la valeur répond à la syntaxe `@Microsoft.KeyVault(...)` est créée. L'application peut alors générer des erreurs car elle s'attendait à un secret répondant à une certaine structure.

En règle générale, il s'agit d'une configuration incorrecte de la [stratégie d'accès à Key Vault](#granting-your-app-access-to-key-vault). Mais cela peut également être dû à un secret qui n'existe plus ou à une erreur de syntaxe dans la référence elle-même.

Si la syntaxe est correcte, vous pouvez afficher d’autres causes d’erreur en vérifiant l’état de résolution actuel dans le portail. Accédez à Paramètres d’application et sélectionnez « Modifier » pour la référence en question. Sous la configuration du paramètre, vous devez voir des informations d’état, y compris les erreurs éventuelles. L’absence de ces informations implique que la syntaxe de la référence n’est pas valide.

Vous pouvez également utiliser l’un des détecteurs intégrés pour obtenir des informations supplémentaires.

### <a name="using-the-detector-for-app-service"></a>Utilisation du détecteur pour App Service

1. Dans le portail, accédez à votre application.
2. Sélectionnez **Diagnostiquer et résoudre les problèmes** .
3. Sélectionnez **Disponibilité et performances** , puis **Application web inactive** .
4. Recherchez **Diagnostics des paramètres de l'application Key Vault** et cliquez sur **En savoir plus** .


### <a name="using-the-detector-for-azure-functions"></a>Utilisation du détecteur pour Azure Functions

1. Dans le portail, accédez à votre application.
2. Accédez à **Fonctionnalités de la plateforme** .
3. Sélectionnez **Diagnostiquer et résoudre les problèmes** .
4. Sélectionnez **Disponibilité et performances** , puis **Function App cesse de fonctionner ou signale des erreurs** .
5. Cliquez sur **Diagnostics des paramètres de l'application Key Vault** .

---
title: Utilisation d’identités managées dans le service Gestion des API Azure | Microsoft Docs
description: Découvrez comment créer des identités affectées par le système et affectées par l’utilisateur dans Gestion des API à l’aide du Portail Azure, de PowerShell et d’un modèle Resource Manager.
services: api-management
documentationcenter: ''
author: miaojiang
manager: anneta
editor: ''
ms.service: api-management
ms.workload: integration
ms.topic: article
ms.date: 11/14/2020
ms.author: apimpm
ms.openlocfilehash: 8ec0f8cf090b3ae85a8602fb39cb07f03a417133
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97605596"
---
# <a name="use-managed-identities-in-azure-api-management"></a>Utilisation d’identités managées dans le service Gestion des API Azure

Cet article vous montre comment créer une identité managée pour une instance Gestion des API Azure et comment accéder à d’autres ressources. Une identité managée générée par Azure Active Directory (Azure AD) permet à votre instance Gestion des API d’accéder facilement et en toute sécurité aux autres ressources protégées par Azure AD, telles qu’Azure Key Vault. Azure gère cette identité et vous n’avez pas besoin d’approvisionner ou de faire tourner les secrets. Pour en savoir plus sur les identités managées, consultez la section [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)

Vous pouvez accorder deux types d’identités à une instance Gestion des API :

- Une *identité affectée par le système* est liée à votre service et est supprimée si votre service est supprimé. Le service ne peut avoir qu’une seule identité attribuée par le système.
- Une *identité affectée par l’utilisateur* est une ressource Azure autonome qui peut être affectée à votre service. Le service peut avoir plusieurs identités affectées par l’utilisateur.

## <a name="create-a-system-assigned-managed-identity"></a>Créer une identité managée affectée par le système

### <a name="azure-portal"></a>Portail Azure

Pour configurer une identité managée dans le portail Azure, vous devez d’abord créer une instance du service Gestion des API, puis activer la fonctionnalité.

1. Créez une instance Gestion des API dans le portail comme vous le faites en temps normal. Accédez-y dans le portail.
2. Sélectionnez **Identités managées**.
3. Dans l’onglet **Attribuée par le système**, définissez **État** sur **Activé**. Sélectionnez **Enregistrer**.

    :::image type="content" source="./media/api-management-msi/enable-system-msi.png" alt-text="Sélections pour activer une identité managée affectée par le système" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les étapes suivantes vous guideront dans la création d’une instance Gestion des API application à laquelle vous attribuez une identité avec Azure PowerShell. 

1. Si nécessaire, installez Azure PowerShell à l’aide des instructions figurant dans le [Guide Azure PowerShell](/powershell/azure/install-az-ps). Exécutez ensuite `Connect-AzAccount` pour créer une connexion avec Azure.

2. Utilisez le code suivant pour créer l’instance. Pour plus d’exemples sur l’utilisation d’Azure PowerShell avec Gestion des API, consultez [Exemples Gestion des API PowerShell](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create an API Management Consumption Sku service.
    New-AzApiManagement -ResourceGroupName $resourceGroupName -Name consumptionskuservice -Location $location -Sku Consumption -Organization contoso -AdminEmail contoso@contoso.com -SystemAssignedIdentity
    ```

3. Mettez à jour une instance existante pour créer l’identité :

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Update an API Management instance
    Set-AzApiManagement -InputObject $apimService -SystemAssignedIdentity
    ```

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Vous pouvez créer une instance Gestion des API avec une identité en incluant la propriété suivante dans la définition de la ressource :

```json
"identity" : {
    "type" : "SystemAssigned"
}
```

Cette propriété indique à Azure de créer et de manager l’identité de votre instance Gestion des API.

Par exemple, un modèle complet Azure Resource Manager peut se présenter comme suit :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "systemAssigned"
        }
    }]
}
```

Quand l’instance est créée, elle a les propriétés supplémentaires suivantes :

```json
"identity": {
    "type": "SystemAssigned",
    "tenantId": "<TENANTID>",
    "principalId": "<PRINCIPALID>"
}
```

La propriété `tenantId` identifie le locataire Azure AD auquel appartient l’identité. La propriété `principalId` est un identificateur unique pour la nouvelle identité de l’instance. Dans Azure AD, le principal de service a le même nom que celui que vous avez donné à votre instance Gestion des API.

> [!NOTE]
> Une instance Gestion des API peut avoir simultanément une identité attribuée par le système et une identité attribuée par l’utilisateur. Dans ce cas, la propriété `type` est `SystemAssigned,UserAssigned`.

## <a name="supported-scenarios-using-system-assigned-identity"></a>Scénarios pris en charge utilisant l’identité affectée par le système

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault"></a>Obtenir un certificat TLS/SSL personnalisé pour l’instance Gestion des API à partir de Azure Key Vault
Vous pouvez utiliser l’identité affectée par le système d’une instance Gestion des API pour récupérer les certificats TLS/SSL personnalisés stockés dans Azure Key Vault. Vous pouvez ensuite affecter ces certificats à des domaines personnalisés dans l’instance Gestion des API. Gardez à l’esprit les éléments suivants :

- Le type de contenu du secret doit être *application/x-pkcs12*.
- Utilisez le point de terminaison du secret de certificat Key Vault, qui contient le secret.

> [!Important]
> Si vous ne fournissez pas la version de l’objet du certificat n’est pas fournie, le service Gestion des API obtiendra automatiquement la version la plus récente du certificat dans les quatre heures qui suivent sa mise à jour dans Key Vault.

L’exemple suivant illustre un modèle Azure Resource Manager qui contient les étapes suivantes :

1. Créer une instance du service Gestion des API avec une identité managée.
2. Mettre à jour les stratégies d’accès d’une instance Azure Key Vault et permettre à l’instance Gestion des API d’obtenir des secrets à partir de cette dernière.
3. Mettre à jour l’instance Gestion des API en définissant un nom de domaine personnalisé à l’aide d’un certificat obtenu à partir de l’instance Key Vault.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "publisherEmail": {
            "type": "string",
            "minLength": 1,
            "metadata": {
                "description": "The email address of the owner of the service"
            }
        },
        "publisherName": {
            "type": "string",
            "defaultValue": "Contoso",
            "minLength": 1,
            "metadata": {
                "description": "The name of the owner of the service"
            }
        },
        "sku": {
            "type": "string",
            "allowedValues": ["Developer",
            "Standard",
            "Premium"],
            "defaultValue": "Developer",
            "metadata": {
                "description": "The pricing tier of this API Management instance"
            }
        },
        "skuCount": {
            "type": "int",
            "defaultValue": 1,
            "metadata": {
                "description": "The instance size of this API Management instance."
            }
        },
        "keyVaultName": {
            "type": "string",
            "metadata": {
                "description": "Name of the vault"
            }
        },
        "proxyCustomHostname1": {
            "type": "string",
            "metadata": {
                "description": "Proxy Custom hostname."
            }
        },
        "keyVaultIdToCertificate": {
            "type": "string",
            "metadata": {
                "description": "Reference to the Key Vault certificate. https://contoso.vault.azure.net/secrets/contosogatewaycertificate."
            }
        }
    },
    "variables": {
        "apiManagementServiceName": "[concat('apiservice', uniqueString(resourceGroup().id))]",
        "apimServiceIdentityResourceId": "[concat(resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName')),'/providers/Microsoft.ManagedIdentity/Identities/default')]"
    },
    "resources": [{
        "apiVersion": "2019-01-01",
        "name": "[variables('apiManagementServiceName')]",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {
        },
        "sku": {
            "name": "[parameters('sku')]",
            "capacity": "[parameters('skuCount')]"
        },
        "properties": {
            "publisherEmail": "[parameters('publisherEmail')]",
            "publisherName": "[parameters('publisherName')]"
        },
        "identity": {
            "type": "systemAssigned"
        }
    },
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2015-06-01",
        "dependsOn": [
            "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "accessPolicies": [{
                "tenantId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').tenantId]",
                "objectId": "[reference(variables('apimServiceIdentityResourceId'), '2015-08-31-PREVIEW').principalId]",
                "permissions": {
                    "secrets": ["get"]
                }
            }]
        }
    },
    {
        "apiVersion": "2017-05-10",
        "name": "apimWithKeyVault",
        "type": "Microsoft.Resources/deployments",
        "dependsOn": [
        "[resourceId('Microsoft.ApiManagement/service', variables('apiManagementServiceName'))]"
        ],
        "properties": {
            "mode": "incremental",
            "templateLink": {
                "uri": "https://raw.githubusercontent.com/solankisamir/arm-templates/master/basicapim.keyvault.json",
                "contentVersion": "1.0.0.0"
            },
            "parameters": {
                "publisherEmail": { "value": "[parameters('publisherEmail')]"},
                "publisherName": { "value": "[parameters('publisherName')]"},
                "sku": { "value": "[parameters('sku')]"},
                "skuCount": { "value": "[parameters('skuCount')]"},
                "proxyCustomHostname1": {"value" : "[parameters('proxyCustomHostname1')]"},
                "keyVaultIdToCertificate": {"value" : "[parameters('keyVaultIdToCertificate')]"}
            }
        }
    }]
}
```

### <a name="authenticate-to-the-back-end-by-using-an-api-management-identity"></a>S’authentifier auprès du back end à l’aide d’une identité Gestion des API

Vous pouvez utiliser l’identité affectée par le système pour vous authentifier auprès du back end par le biais de la stratégie [authentication-managed-identity](api-management-authentication-policies.md#ManagedIdentity).

## <a name="create-a-user-assigned-managed-identity"></a>Créer une identité managée attribuée par l’utilisateur

> [!NOTE]
> Vous pouvez associer une instance Gestion des API à un maximum de 10 identités managées assignées par l’utilisateur.

### <a name="azure-portal"></a>Portail Azure

Pour configurer une identité managée dans le portail, vous devez d’abord créer une instance du service Gestion des API, puis activer la fonctionnalité.

1. Créez une instance Gestion des API dans le portail comme vous le faites en temps normal. Accédez-y dans le portail.
2. Sélectionnez **Identités managées**.
3. Dans l’onglet **Attribuée par l’utilisateur**, sélectionnez **Ajouter**.
4. Recherchez l’identité que vous avez créée précédemment et sélectionnez-la. Sélectionnez **Ajouter**.

   :::image type="content" source="./media/api-management-msi/enable-user-assigned-msi.png" alt-text="Sélections pour activer une identité managée affectée par l’utilisateur" border="true":::

### <a name="azure-powershell"></a>Azure PowerShell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

Les étapes suivantes vous guideront dans la création d’une instance Gestion des API application à laquelle vous attribuez une identité avec Azure PowerShell. 

1. Si nécessaire, installez Azure PowerShell à l’aide des instructions figurant dans le [Guide Azure PowerShell](/powershell/azure/install-az-ps). Exécutez ensuite `Connect-AzAccount` pour créer une connexion avec Azure.

2. Utilisez le code suivant pour créer l’instance. Pour plus d’exemples sur l’utilisation d’Azure PowerShell avec Gestion des API, consultez [Exemples Gestion des API PowerShell](powershell-samples.md).

    ```azurepowershell-interactive
    # Create a resource group.
    New-AzResourceGroup -Name $resourceGroupName -Location $location

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Create an API Management Consumption Sku service.
    $userIdentities = @($userAssignedIdentity.Id)

    New-AzApiManagement -ResourceGroupName $resourceGroupName -Location $location -Name $apiManagementName -Organization contoso -AdminEmail admin@contoso.com -Sku Consumption -UserAssignedIdentity $userIdentities
    ```

3. Mettre à jour un service existant pour affecter une identité au service :

    ```azurepowershell-interactive
    # Get an API Management instance
    $apimService = Get-AzApiManagement -ResourceGroupName $resourceGroupName -Name $apiManagementName

    # Create a user-assigned identity. This requires installation of the "Az.ManagedServiceIdentity" module.
    $userAssignedIdentity = New-AzUserAssignedIdentity -Name $userAssignedIdentityName -ResourceGroupName $resourceGroupName

    # Update an API Management instance
    $userIdentities = @($userAssignedIdentity.Id)
    Set-AzApiManagement -InputObject $apimService -UserAssignedIdentity $userIdentities
    ```

### <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager

Vous pouvez créer une instance Gestion des API avec une identité en incluant la propriété suivante dans la définition de la ressource :

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {}
    }
}
```

L’ajout du type attribué par l’utilisateur indique à Azure d’utiliser l’identité affectée par l’utilisateur qui est spécifiée pour votre instance.

Par exemple, un modèle complet Azure Resource Manager peut se présenter comme suit :

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "0.9.0.0",
    "resources": [{
        "apiVersion": "2019-12-01",
        "name": "contoso",
        "type": "Microsoft.ApiManagement/service",
        "location": "[resourceGroup().location]",
        "tags": {},
        "sku": {
            "name": "Developer",
            "capacity": "1"
        },
        "properties": {
            "publisherEmail": "admin@contoso.com",
            "publisherName": "Contoso"
        },
        "identity": {
            "type": "UserAssigned",
             "userAssignedIdentities": {
                "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]": {}
             }
        },
         "dependsOn": [       
          "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities', variables('identityName'))]"
        ]
    }]
}
```

Quand le service est créé, il a les propriétés supplémentaires suivantes :

```json
"identity": {
    "type": "UserAssigned",
    "userAssignedIdentities": {
        "<RESOURCEID>": {
            "principalId": "<PRINCIPALID>",
            "clientId": "<CLIENTID>"
        }
    }
}
```

La propriété `principalId` est un identificateur unique pour l’identité qui est utilisée pour l’administration d’Azure AD. La propriété `clientId` est un identificateur unique pour la nouvelle identité de l’application qui est utilisée pour spécifier l’identité à utiliser lors des appels de runtime.

> [!NOTE]
> Une instance Gestion des API peut avoir simultanément une identité attribuée par le système et une identité attribuée par l’utilisateur. Dans ce cas, la propriété `type` est `SystemAssigned,UserAssigned`.

## <a name="supported-scenarios-using-user-assigned-managed-identity"></a>Scénarios pris en charge utilisant l’identité managée affectée par l’utilisateur

### <a name="obtain-a-custom-tlsssl-certificate-for-the-api-management-instance-from-azure-key-vault"></a><a name="use-ssl-tls-certificate-from-azure-key-vault-ua"></a>Obtenir un certificat TLS/SSL personnalisé pour l’instance Gestion des API à partir de Azure Key Vault
Vous pouvez utiliser n’importe quelle identité affectée par l’utilisateur pour établir une relation de confiance entre une instance Gestion des API et Azure Key Vault. Cette confiance peut ensuite être utilisée pour récupérer des certificats TLS/SSL personnalisés stockés dans Azure Key Vault. Vous pouvez ensuite affecter ces certificats à des domaines personnalisés dans l’instance Gestion des API. 

Gardez à l’esprit les éléments suivants :

- Le type de contenu du secret doit être *application/x-pkcs12*.
- Utilisez le point de terminaison du secret de certificat Key Vault, qui contient le secret.

> [!Important]
> Si vous ne fournissez pas la version de l’objet du certificat n’est pas fournie, le service Gestion des API obtiendra automatiquement la version la plus récente du certificat dans les quatre heures qui suivent sa mise à jour dans Key Vault.

Pour obtenir le modèle complet, consultez [Gestion des API avec SSL basé sur Key Vault à l’aide de l’identité affectée par l’utilisateur](https://github.com/Azure/azure-quickstart-templates/blob/master/101-api-management-key-vault-create/azuredeploy.json).

Dans ce modèle, vous allez déployer :

* Gestion des API Azure
* Identité affectée par l’utilisateur gérée par Azure
* Azure Key Vault pour le stockage du certificat SSL/TLS

Pour exécuter automatiquement le déploiement, cliquez sur le bouton ci-dessous :

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-api-management-key-vault-create%2Fazuredeploy.json)

### <a name="authenticate-to-the-back-end-by-using-a-user-assigned-identity"></a>S’authentifier auprès du back end à l’aide d’une identité affectée par l’utilisateur

Vous pouvez utiliser l’identité affectée par l’utilisateur pour vous authentifier auprès du back end par le biais de la stratégie [authentication-managed-identity](api-management-authentication-policies.md#ManagedIdentity).

## <a name="remove-an-identity"></a><a name="remove"></a>Supprimer une identité

Vous pouvez supprimer une identité affectée par le système en désactivant la fonctionnalité via le portail ou le modèle Azure Resource Manager de la même manière qu’elle a été créée. Les identités attribuées par l’utilisateur peuvent être supprimées individuellement. Pour supprimer toutes les identités, définissez le type d’identité sur `"None"`.

Si vous supprimez une identité affectée par le système de cette façon, vous la supprimez également d’Azure AD. Les identités affectées par le système sont aussi automatiquement supprimées d’Azure AD quand l’instance Gestion des API est supprimée.

Pour supprimer toutes les identités à l’aide du modèle Azure Resource Manager, mettez à jour cette section :

```json
"identity": {
    "type": "None"
}
```

> [!Important]
> Si une instance Gestion des API est configurée avec un certificat SSL personnalisé de Key Vault et que vous essayez de désactiver une identité managée, la requête échoue.
>
> Vous pouvez vous débloquer en passant du certificat Azure Key Vault à un certificat encodé en ligne, puis en désactivant l’identité managée. Pour plus d’informations, consultez [Configurer un nom de domaine personnalisé](configure-custom-domain.md).

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les identités managées pour les ressources Azure :

* [Que sont les identités managées pour les ressources Azure ?](../active-directory/managed-identities-azure-resources/overview.md)
* [Modèles Microsoft Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates)
* [Authentifier avec l’identité managée](./api-management-authentication-policies.md#ManagedIdentity)

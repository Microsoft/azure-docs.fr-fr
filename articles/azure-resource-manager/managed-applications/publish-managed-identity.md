---
title: Application gérée avec identité managée
description: Configurez l’application gérée avec une identité gérée pour lier des ressources existantes, gérer des ressources Azure et fournir une identité opérationnelle pour le journal d’activité.
ms.topic: conceptual
ms.author: jobreen
author: jjbfour
ms.date: 05/13/2019
ms.openlocfilehash: 277faa2d47df9fddd1762d90d9aa2fb5bf00d4df
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "82508125"
---
# <a name="azure-managed-application-with-managed-identity"></a>Application managée Azure avec identité managée

> [!NOTE]
> La prise en charge de l’identité managée pour les applications managées est actuellement en préversion. Utilisez la version d’api 2018-09-01-preview pour employer l’identité managée.

Découvrez comment configurer une application managée pour contenir une identité managée. L’identité managée peut être utilisée pour permettre au client d’accorder l’accès à d’autres ressources existantes à l’application managée. Managée par la plateforme Azure, l’identité ne nécessite pas que vous approvisionniez ou permutiez de secrets. Pour plus d’informations sur les identités managées dans AAD (Azure Active Directory), consultez [Identités managées pour les ressources Azure](../../active-directory/managed-identities-azure-resources/overview.md).

Deux types d’identité peuvent être accordés à votre application :

- Une **identité attribuée par le système** est liée à votre application et est supprimée si votre application est supprimée. Une application ne peut avoir qu’une seule identité attribuée par le système.
- Une **identité attribuée par l’utilisateur** est une ressource Azure autonome qui peut être assignée à votre application. Une application peut avoir plusieurs identités attribuées par l’utilisateur.

## <a name="how-to-use-managed-identity"></a>Utilisation de l’identité managée

L’identité managée sert dans de nombreux scénarios pour les applications managées. Quelques scénarios courants qui peuvent être résolus sont :

- Déploiement d’une application managée liée à des ressources Azure existantes. Il peut s’agir par exemple du déploiement d’une machine virtuelle Azure dans l’application managée qui est attachée à une [interface réseau existante](../../virtual-network/virtual-network-network-interface-vm.md).
- Octroi de l’accès aux ressources Azure en dehors du **groupe de ressources managé** à l’application managée et à l’éditeur.
- Fourniture d’une identité opérationnelle des applications managées pour le journal d’activité et d’autres services dans Azure.

## <a name="adding-managed-identity"></a>Ajout d’une identité managée

La création d’une application managée avec une identité managée nécessite la définition d’une propriété supplémentaire sur la ressource Azure. L’exemple suivant illustre un exemple de propriété **identity** :

```json
{
"identity": {
    "type": "SystemAssigned, UserAssigned",
    "userAssignedIdentities": {
        "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity": {}
    }
}
```

Il existe deux manières courantes de créer une application managée avec **identity** : [CreateUIDefinition.json](./create-uidefinition-overview.md) et les [modèles Azure Resource Manager](../templates/template-syntax.md). Pour des scénarios de création uniques et simples, CreateUIDefinition doit être utilisé pour activer l’identité managée, car il propose une expérience plus riche. Toutefois, quand vous utilisez des systèmes élaborés ou complexes qui demandent des déploiements automatisés ou multiples d’applications managées, les modèles peuvent être utilisés.

### <a name="using-createuidefinition"></a>Utilisation de CreateUIDefinition

Une application managée peut être configurée avec une identité managée via [CreateUIDefinition.json](./create-uidefinition-overview.md). Dans la [section des sorties](./create-uidefinition-overview.md#outputs), la clé `managedIdentity` peut être utilisée pour remplacer la propriété d’identité du modèle d’application managée. L’exemple ci-dessous active l’identité **attribuée par le système** sur l’application managée. Des objets d’identité plus complexes peuvent être formés à l’aide des éléments CreateUIDefinition pour demander des entrées au consommateur. Ces entrées peuvent être utilisées pour construire des applications managées avec une **identité attribuée par l’utilisateur**.

```json
"outputs": {
    "managedIdentity": { "Type": "SystemAssigned" }
}
```

#### <a name="when-to-use-createuidefinition-for-managed-identity"></a>Quand utiliser CreateUIDefinition pour l’identité managée

Voici quelques recommandations sur le moment opportun d’utiliser CreateUIDefinition pour l’activation de l’identité managée sur les applications managées.

- La création de l’application managée passe par le portail Azure ou la Place de marché Azure.
- L’identité managée nécessite une entrée de consommateur complexe.
- L’identité managée est nécessaire lors de la création de l’application managée.

#### <a name="managed-identity-createuidefinition-control"></a>Contrôle CreateUIDefinition d’identité managée

CreateUIDefinition prend en charge un [contrôle d’identité managée](./microsoft-managedidentity-identityselector.md) intégré.

```json
{
  "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
  "handler": "Microsoft.Azure.CreateUIDef",
  "version": "0.0.1-preview",
  "parameters": {
    "basics": [],
    "steps": [
      {
        "name": "applicationSettings",
        "label": "Application Settings",
        "subLabel": {
          "preValidation": "Configure your application settings",
          "postValidation": "Done"
        },
        "bladeTitle": "Application Settings",
        "elements": [
          {
            "name": "appName",
            "type": "Microsoft.Common.TextBox",
            "label": "Managed application Name",
            "toolTip": "Managed application instance name",
            "visible": true
          },
          {
            "name": "appIdentity",
            "type": "Microsoft.ManagedIdentity.IdentitySelector",
            "label": "Managed Identity Configuration",
            "toolTip": {
              "systemAssignedIdentity": "Enable system assigned identity to grant the managed application access to additional existing resources.",
              "userAssignedIdentity": "Add user assigned identities to grant the managed application access to additional existing resources."
            },
            "defaultValue": {
              "systemAssignedIdentity": "Off"
            },
            "options": {
              "hideSystemAssignedIdentity": false,
              "hideUserAssignedIdentity": false,
              "readOnlySystemAssignedIdentity": false
            },
            "visible": true
          }
        ]
      }
    ],
    "outputs": {
      "applicationResourceName": "[steps('applicationSettings').appName]",
      "location": "[location()]",
      "managedIdentity": "[steps('applicationSettings').appIdentity]"
    }
  }
}
```

![CreateUIDefinition d’identité managée](./media/publish-managed-identity/msi-cuid.png)

### <a name="using-azure-resource-manager-templates"></a>Utilisation de modèles Azure Resource Manager

> [!NOTE]
> Les modèles d’application managée de la Place de marché sont automatiquement générés pour les clients qui passent par l’expérience de création du portail Azure.
> Pour ces scénarios, la clé de sortie `managedIdentity` sur CreateUIDefinition doit être utilisée pour activer l’identité.

L’identité managée peut également être activée par le biais de modèles Azure Resource Manager. L’exemple ci-dessous active l’identité **attribuée par le système** sur l’application managée. Des objets d’identité plus complexes peuvent être formés à l’aide des paramètres de modèle Azure Resource Manager pour fournir des entrées. Ces entrées peuvent être utilisées pour construire des applications managées avec une **identité attribuée par l’utilisateur**.

#### <a name="when-to-use-azure-resource-manager-templates-for-managed-identity"></a>Quand utiliser des modèles Azure Resource Manager pour l’identité managée

Voici quelques recommandations sur le moment opportun d’utiliser des modèles Azure Resource Manager pour l’activation de l’identité managée sur les applications managées.

- Les applications managées peuvent être déployées par programmation selon un modèle.
- Les attributions de rôles personnalisées pour l’identité managée sont nécessaires pour provisionner l’application managée.
- L’application managée n’a pas besoin du flux de création du portail ni de la Place de marché Azure.

#### <a name="systemassigned-template"></a>Modèle SystemAssigned

Modèle Azure Resource Manager de base qui déploie une application managée avec une identité **attribuée par le système**.

```json
"resources": [
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "SystemAssigned"
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

### <a name="userassigned-template"></a>Modèle UserAssigned

Modèle Azure Resource Manager de base qui déploie une application managée avec une **identité attribuée par l’utilisateur**.

```json
"resources": [
    {
      "type": "Microsoft.ManagedIdentity/userAssignedIdentities",
      "name": "[parameters('managedIdentityName')]",
      "apiVersion": "2018-11-30",
      "location": "[parameters('location')]"
    },
    {
        "type": "Microsoft.Solutions/applications",
        "name": "[parameters('applicationName')]",
        "apiVersion": "2018-09-01-preview",
        "location": "[parameters('location')]",
        "identity": {
            "type": "UserAssigned",
            "userAssignedIdentities": {
                "[resourceID('Microsoft.ManagedIdentity/userAssignedIdentities/',parameters('managedIdentityName'))]": {}
            }
        },
        "properties": {
            "ManagedResourceGroupId": "[parameters('managedByResourceGroupId')]",
            "parameters": { }
        }
    }
]
```

## <a name="granting-access-to-azure-resources"></a>Octroi de l’accès aux ressources Azure

Une fois qu’une application managée a reçu une identité, l’accès aux ressources Azure existantes peut lui être accordé. Ce processus peut être effectué via l’interface de contrôle d’accès (IAM) dans le portail Azure. Le nom de l’application managée ou **identité attribuée par l’utilisateur** peut faire l’objet d’une recherche pour ajouter une attribution de rôle.

![Ajouter une attribution de rôle pour l’application managée](./media/publish-managed-identity/identity-role-assignment.png)

## <a name="linking-existing-azure-resources"></a>Liaison des ressources Azure existantes

> [!NOTE]
> Une **identité attribuée par l’utilisateur** doit être [configurée](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) avant de déployer l’application managée. De plus, le déploiement de ressources liées des applications managées est uniquement pris en charge pour le type **place de marché**.

L’identité managée peut également servir à déployer une application managée qui requiert l’accès à des ressources existantes lors de son déploiement. Quand l’application managée est provisionnée par le client, des **identités attribuées par l’utilisateur** peuvent être ajoutées afin de fournir des autorisations supplémentaires pour le déploiement de **mainTemplate**.

### <a name="authoring-the-createuidefinition-with-a-linked-resource"></a>Création de CreateUIDefinition avec une ressource liée

Lors de la liaison du déploiement de l’application managée à des ressources existantes, la ressource Azure existante et une **identité attribuée par l’utilisateur** avec l’attribution de rôle applicable à cette ressource doivent toutes deux être fournies.

 Exemple de CreateUIDefinition qui requiert deux entrées : un ID de ressource d’interface réseau et un ID de ressource d’identité attribuée par l’utilisateur.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/0.1.2-preview/CreateUIDefinition.MultiVm.json#",
    "handler": "Microsoft.Compute.MultiVm",
    "version": "0.1.2-preview",
    "parameters": {
        "basics": [
            {}
        ],
        "steps": [
            {
                "name": "managedApplicationSetting",
                "label": "Managed Application Settings",
                "subLabel": {
                    "preValidation": "Managed Application Settings",
                    "postValidation": "Done"
                },
                "bladeTitle": "Managed Application Settings",
                "elements": [
                    {
                        "name": "networkInterfaceId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "network interface resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.Network/networkInterfaces/existingnetworkinterface",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.Network/networkInterfaces/networkinterface1",
                        "visible": true
                    },
                    {
                        "name": "userAssignedId",
                        "type": "Microsoft.Common.TextBox",
                        "label": "user assigned identity resource id",
                        "defaultValue": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/testRG/providers/Microsoft.ManagedIdentity/userassignedidentites/myuserassignedidentity",
                        "toolTip": "Must represent the identity as an Azure Resource Manager resource identifer format ex. /subscriptions/sub1/resourcegroups/myGroup/providers/Microsoft.ManagedIdentity/userAssignedIdentities/identity1",
                        "visible": true
                    }
                ]
            }
        ],
        "outputs": {
            "existingNetworkInterfaceId": "[steps('managedApplicationSetting').networkInterfaceId]",
            "managedIdentity": "[parse(concat('{\"Type\":\"UserAssigned\",\"UserAssignedIdentities\":{',string(steps('managedApplicationSetting').userAssignedId),':{}}}'))]"
        }
    }
}
```

Ce fichier CreateUIDefinition.json génère une expérience de création d’utilisateur qui comporte deux champs. Le premier champ permet à l’utilisateur d’entrer l’ID de ressource Azure pour la ressource liée au déploiement de l’application managée. Le deuxième champ permet à un consommateur d’entrer l’ID de ressource Azure de l’**identité attribuée par l’utilisateur**, qui a accès à la ressource Azure liée. L’expérience générée peut se présenter comme suit :

![Exemple de CreateUIDefinition avec deux entrées : un ID de ressource d’interface réseau et un ID de ressource d’identité attribuée par l’utilisateur](./media/publish-managed-identity/network-interface-cuid.png)

### <a name="authoring-the-maintemplate-with-a-linked-resource"></a>Création de mainTemplate avec une ressource liée

En plus de la mise à jour de CreateUIDefinition, le modèle principal doit également être mis à jour pour accepter l’ID de ressource liée transmis. Le modèle principal peut être mis à jour pour accepter la nouvelle sortie en ajoutant un nouveau paramètre. Dans la mesure où la sortie `managedIdentity` remplace la valeur sur le modèle d’application managée généré, elle n’est pas transmise au modèle principal et ne doit pas être incluse dans la section des paramètres.

Exemple de modèle principal qui définit le profil réseau sur une interface réseau existante fournie par CreateUIDefinition.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
      "existingNetworkInterfaceId": { "type": "string" }
    },
    "variables": {
    },
    "resources": [
        {
            "apiVersion": "2016-04-30-preview",
            "type": "Microsoft.Compute/virtualMachines",
            "name": "myLinkedResourceVM",
            "location": "[resourceGroup().location]",
            "properties": {
                …,
                "networkProfile": {
                    "networkInterfaces": [
                        {
                            "id": "[parameters('existingNetworkInterfaceId')]"
                        }
                    ]
                }
            }
        }
    ]
}
```

### <a name="consuming-the-managed-application-with-a-linked-resource"></a>Utilisation de l’application managée avec une ressource liée

Une fois le package d’application managée créé, l’application managée peut être consommée via le portail Azure. Avant de pouvoir être utilisée, plusieurs étapes préalables doivent être suivies.

- Une instance de la ressource Azure liée requise doit être créée.
- L’**identité attribuée par l’utilisateur** doit [être créée et recevoir des attributions de rôles](../../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-portal.md) sur la ressource liée.
- L’ID de la ressource liée existante et l’ID de l’**identité attribuée par l’utilisateur** sont fournis à CreateUIDefinition.

## <a name="accessing-the-managed-identity-token"></a>Accès au jeton de l’identité managée

Le jeton de l’application managée est désormais accessible via l’api `listTokens` du locataire de l’éditeur. Voici un exemple de demande :

``` HTTP
POST https://management.azure.com/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}/listTokens?api-version=2018-09-01-preview HTTP/1.1

{
    "authorizationAudience": "https://management.azure.com/",
    "userAssignedIdentities": [
        "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.ManagedIdentity/userAssignedIdentities/{userAssignedIdentityName}"
    ]
}
```

Paramètres du corps de la demande :

Paramètre | Obligatoire | Description
---|---|---
authorizationAudience | *non* | URI ID d’application de la ressource cible. Il s’agit également de la revendication `aud` (audience) du jeton émis. La valeur par défaut est « https://management.azure.com/  »
userAssignedIdentities | *non* | Liste des identités managées attribuées par l’utilisateur pour lesquelles récupérer un jeton. Si ce paramètre n’est pas spécifié, `listTokens` retourne le jeton pour l’identité managée attribuée par le système.


Voici un exemple de réponse :

``` HTTP
HTTP/1.1 200 OK
Content-Type: application/json

{
    "value": [
        {
            "access_token": "eyJ0eXAi…",
            "expires_in": "2…",
            "expires_on": "1557…",
            "not_before": "1557…",
            "authorizationAudience": "https://management.azure.com/",
            "resourceId": "/subscriptions/{subscriptionId}/resourceGroups/{resourceGroup}/providers/Microsoft.Solutions/applications/{applicationName}",
            "token_type": "Bearer"
        }
    ]
}
```

La réponse contient un tableau de jetons sous la propriété `value` :

Paramètre | Description
---|---
access_token | Le jeton d’accès demandé.
expires_in | Nombre de secondes de validité du jeton d’accès.
expires_on | L’intervalle de temps lorsque le jeton d’accès expire. Il est exprimé en nombre de secondes depuis l’époque.
not_before | Intervalle de temps pendant lequel le jeton d’accès prend effet. Il est exprimé en nombre de secondes depuis l’époque.
authorizationAudience | Valeur `aud` (audience) pour laquelle le jeton d’accès a été demandé. Il s’agit de la même valeur que celle fournie dans la demande `listTokens`.
resourceId | ID de ressource Azure pour le jeton émis. Il s’agit de l’ID de l’application managée ou de l’ID de l’identité attribuée par l’utilisateur.
token_type | Type du jeton.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Guide pratique pour configurer une application managée avec un fournisseur personnalisé](../custom-providers/overview.md)

---
title: Accorder à une application l’accès à d’autres ressources Azure
description: Cet article explique comment accorder à votre application Service Fabric avec identité managée l’accès à d’autres ressources Azure prenant en charge l’authentification basée sur Azure Active Directory.
ms.topic: article
ms.date: 12/09/2019
ms.openlocfilehash: c7560294fbf6d122396b6a5a8ffd3ee93bc89048
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507453"
---
# <a name="granting-a-service-fabric-applications-managed-identity-access-to-azure-resources"></a>Accorder à l’identité managée d’une application Service Fabric l’accès à des ressources Azure

Pour que l’application puisse utiliser son identité managée et accéder à d’autres ressources, les autorisations doivent être accordées à cette identité sur la ressource Azure protégée faisant l’objet d’un accès. L’octroi d’autorisations est généralement une action de gestion sur le « plan de contrôle » du service Azure propriétaire de la ressource protégée routée via Azure Resource Manager, ce qui permet d’appliquer toute vérification d’accès en fonction du rôle applicable.

La séquence exacte d’étapes dépend ensuite du type de ressource Azure qui fait l’objet de l’accès, ainsi que de la langue/du client utilisé pour accorder des autorisations. Le reste de cet article suppose qu’une identité affectée par l’utilisateur est attribuée à l’application et comprend plusieurs exemples typiques pour votre commodité, mais il ne s’agit en aucun cas d’une référence exhaustive pour cette rubrique ; consultez la documentation des services Azure respectifs pour obtenir des instructions à jour sur l’octroi d’autorisations.  

## <a name="granting-access-to-azure-storage"></a>Octroi de l’accès au Stockage Azure
Vous pouvez utiliser l’identité managée de l’application Service Fabric (affectée par l’utilisateur dans ce cas) pour récupérer les données à partir d’un objet blob de stockage Azure. Accordez à l’identité les autorisations requises dans le Portail Azure en procédant comme suit :

1. Accédez au compte de stockage.
2. Cliquez sur le lien (IAM) de contrôle d’accès dans le panneau de gauche.
3. (facultatif) Vérifier l’accès existant : sélectionnez l’identité managée affectée par le système ou par l’utilisateur dans le contrôle « Rechercher » ; sélectionnez l’identité appropriée dans la liste de résultats
4. Cliquez sur + Ajouter une attribution de rôle en haut de la page pour ajouter une nouvelle attribution de rôle à l’identité de l'application.
Sous Rôle, dans la liste déroulante, sélectionnez Lecteur des données Blob du stockage.
5. Dans la liste déroulante suivante, sous Attribuer l’accès à, choisissez `User assigned managed identity`.
6. Ensuite, assurez-vous que l’abonnement approprié est répertorié dans la liste déroulante Abonnement, puis définissez Groupe de ressources sur Tous les groupes de ressources.
7. Sous Sélectionner, choisissez l’UAI correspondant à l’application Service Fabric et cliquez sur Enregistrer.

La prise en charge des identités managées Service Fabric affectées par le système n’inclut pas l’intégration dans le Portail Azure ; si votre application utilise une identité affectée par le système, vous devez rechercher d’abord l’ID client de l’identité de l’application, puis répéter les étapes ci-dessus, mais en sélectionnant l’option `Azure AD user, group, or service principal` dans le contrôle Rechercher.

## <a name="granting-access-to-azure-key-vault"></a>Octroi de l’accès à Azure Key Vault
De même que pour l’accès au stockage, vous pouvez tirer parti de l’identité managée d’une application Service Fabric pour accéder à un coffre de clés Azure. Les étapes permettant d’accorder l’accès dans le Portail Azure sont similaires à celles répertoriées ci-dessus et ne sont pas répétées ici. Reportez-vous à l’image ci-dessous pour connaître les différences.

![Stratégie d’accès au coffre de clés](../key-vault/media/vs-secure-secret-appsettings/add-keyvault-access-policy.png)

L’exemple suivant illustre l’octroi d’accès à un coffre par le biais d’un déploiement de modèle. Ajoutez le ou les extraits de code ci-dessous comme une autre entrée sous l’élément `resources` du modèle. L’exemple montre l’octroi d’accès pour les types d’identité attribués par l’utilisateur et attribués par le système, respectivement. Choisissez celui qui convient.

```json
    # under 'variables':
  "variables": {
        "userAssignedIdentityResourceId" : "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]",
    }
    # under 'resources':
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2018-02-14",
        "properties": {
            "accessPolicies": [
                {
                    "tenantId": "[reference(variables('userAssignedIdentityResourceId'), '2018-11-30').tenantId]",
                    "objectId": "[reference(variables('userAssignedIdentityResourceId'), '2018-11-30').principalId]",
                    "dependsOn": [
                        "[variables('userAssignedIdentityResourceId')]"
                    ],
                    "permissions": {
                        "keys":         ["get", "list"],
                        "secrets":      ["get", "list"],
                        "certificates": ["get", "list"]
                    }
                }
            ]
        }
    },
```
Et pour les identités managées attribuées par le système :
```json
    # under 'variables':
  "variables": {
        "sfAppSystemAssignedIdentityResourceId": "[concat(resourceId('Microsoft.ServiceFabric/clusters/applications/', parameters('clusterName'), parameters('applicationName')), '/providers/Microsoft.ManagedIdentity/Identities/default')]"
    }
    # under 'resources':
    {
        "type": "Microsoft.KeyVault/vaults/accessPolicies",
        "name": "[concat(parameters('keyVaultName'), '/add')]",
        "apiVersion": "2018-02-14",
        "properties": {
            "accessPolicies": [
            {
                    "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
                    "tenantId": "[reference(variables('sfAppSystemAssignedIdentityResourceId'), '2018-11-30').tenantId]",
                    "objectId": "[reference(variables('sfAppSystemAssignedIdentityResourceId'), '2018-11-30').principalId]",
                    "dependsOn": [
                        "[variables('sfAppSystemAssignedIdentityResourceId')]"
                    ],
                    "permissions": {
                        "secrets": [
                            "get",
                            "list"
                        ],
                        "certificates": 
                        [
                            "get", 
                            "list"
                        ]
                    }
            },
        ]
        }
    }
```

Pour plus d’informations, consultez [Coffres - Mettre à jour la stratégie d’accès](/rest/api/keyvault/vaults/updateaccesspolicy).

## <a name="next-steps"></a>Étapes suivantes
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par le système](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)

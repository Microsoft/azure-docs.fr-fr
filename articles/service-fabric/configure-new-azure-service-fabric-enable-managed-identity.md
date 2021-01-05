---
title: Configurer la prise en charge des identités managées pour un nouveau cluster Service Fabric
description: Voici comment activer la prise en charge des identités managées dans un nouveau cluster Azure Service Fabric
ms.topic: article
ms.date: 12/09/2019
ms.openlocfilehash: ae4fec452a2342a68843d874ba955b594014c46d
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574664"
---
# <a name="configure-managed-identity-support-for-a-new-service-fabric-cluster"></a>Configurer la prise en charge des identités managées pour un nouveau cluster Service Fabric

Pour utiliser des [identités managées pour des ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) dans vos applications Service Fabric, commencez par activer le *service de jeton d’identité managée* sur le cluster. Ce service est responsable de l’authentification des applications Service Fabric à l’aide de leurs identités managées et de l’obtention de jetons d’accès en leur nom. Une fois le service activé, vous pouvez le voir dans Service Fabric Explorer sous la section **Système** dans le volet gauche, exécuté sous le nom **fabric:/System/ManagedIdentityTokenService** à côté d’autre services système.

> [!NOTE]
> Le runtime Service Fabric version 6.5.658.9590 ou une version ultérieure est requis pour activer le **Service de jetons des identités managées**.  

## <a name="enable-the-managed-identity-token-service"></a>Activer le service de jeton l’identité managée

Pour activer le service de jeton d’identité managée au moment de la création du cluster, ajoutez l’extrait de code suivant à votre modèle Azure Resource Manager de cluster :

```json
"fabricSettings": [
    {
        "name": "ManagedIdentityTokenService",
        "parameters": [
            {
                "name": "IsEnabled",
                "value": "true"
            }
        ]
    }
]
```

## <a name="errors"></a>Erreurs

Si le déploiement échoue avec ce message, cela signifie que le cluster n’est pas à la version de Service Fabric requise (le runtime minimal pris en charge est 6.5 CU2) :


```json
{
    "code": "ParameterNotAllowed",
    "message": "Section 'ManagedIdentityTokenService' and Parameter 'IsEnabled' is not allowed."
}
```

## <a name="related-articles"></a>Articles connexes

* Vérifier [la prise en charge des identités managées](./concepts-managed-identity.md) dans Azure Service Fabric

* [Activer le support d’identités managées dans un cluster Azure Service Fabric existant](./configure-existing-cluster-enable-managed-identity-token-service.md)

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application Azure Service Fabric avec une identité managée attribuée par le système](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Tirer parti de l’identité managée d’une application Service Fabric à partir du code de service](./how-to-managed-identity-service-fabric-app-code.md)
* [Accorder à une application Azure Service Fabric l’accès à d’autres ressources Azure](./how-to-grant-access-other-resources.md)
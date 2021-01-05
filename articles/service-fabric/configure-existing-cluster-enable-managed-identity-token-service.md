---
title: Configurer la prise en charge des identités managées dans un cluster Service Fabric existant
description: Voici comment activer la prise en charge des identités managées dans un cluster Azure Service Fabric existant
ms.topic: article
ms.date: 03/11/2019
ms.openlocfilehash: 7bcba34e941f6fee038aa475d7c0869eb65aca2e
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574698"
---
# <a name="configure-managed-identity-support-in-an-existing-service-fabric-cluster"></a>Configurer la prise en charge des identités managées dans un cluster Service Fabric existant

Pour utiliser des [identités managées pour des ressources Azure](../active-directory/managed-identities-azure-resources/overview.md) dans vos applications Service Fabric, commencez par activer le *service de jeton d’identité managée* sur le cluster. Ce service est responsable de l’authentification des applications Service Fabric à l’aide de leurs identités managées et de l’obtention de jetons d’accès en leur nom. Une fois le service activé, vous pouvez le voir dans Service Fabric Explorer sous la section **Système** dans le volet de gauche, exécuté sous le nom **fabric:/System/ManagedIdentityTokenService**.

> [!NOTE]
> Le runtime Service Fabric version 6.5.658.9590 ou une version ultérieure est requis pour activer le **Service de jetons des identités managées**.  
>
> Vous pouvez rechercher la version Service Fabric d’un cluster à partir du Portail Azure en ouvrant la ressource de cluster et en vérifiant la propriété **version Fabric version** dans la section **Essentials**.
>
> Si le cluster est en mode de mise à niveau **Manuelle**, vous devez d’abord le mettre à niveau vers 6.5.658.9590 ou une version ultérieure.

## <a name="enable-managed-identity-token-service-in-an-existing-cluster"></a>Activer le *service de jeton d’identité managée* dans un cluster existant

Pour activer le service de jeton d’identité managée dans un cluster existant, vous devez lancer une mise à niveau de cluster en spécifiant deux modifications : (1) activation du service de jeton d’identité managée et (2) demande de redémarrage de chaque nœud. Tout d’abord, ajoutez l’extrait de code suivant à votre modèle Azure Resource Manager de cluster :

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

Pour que les modifications prennent effet, vous devez également modifier la stratégie de mise à niveau pour spécifier un redémarrage forcé du runtime Service Fabric sur chaque nœud à mesure que la mise à niveau progresse au sein du cluster. Ce redémarrage garantit que le service système qui vient d’être activé est démarré et en cours d’exécution sur chaque nœud. Dans l’extrait de code ci-dessous, `forceRestart` est le paramètre essentiel permettant d’activer le redémarrage. Pour les paramètres restants, utilisez les valeurs décrites ci-dessous ou utilisez les valeurs personnalisées existantes déjà spécifiées pour la ressource de cluster. Les paramètres personnalisés pour la stratégie de mise à niveau de l’infrastructure (« upgradeDescription ») peuvent être consultés à partir du portail Azure en sélectionnant l’option « Mises à niveau de la structure » sur la ressource Service Fabric ou sur resources.azure.com. Les options par défaut pour la stratégie de mise à niveau (« upgradeDescription ») ne peuvent pas être consultées à partir de PowerShell ou de resources.azure.com. Pour plus d’informations, consultez [ClusterUpgradePolicy](/dotnet/api/microsoft.azure.management.servicefabric.models.clusterupgradepolicy?view=azure-dotnet).  

```json
"upgradeDescription": {
    "forceRestart": true,
    "healthCheckRetryTimeout": "00:45:00",
    "healthCheckStableDuration": "00:05:00",
    "healthCheckWaitDuration": "00:05:00",
    "upgradeDomainTimeout": "02:00:00",
    "upgradeReplicaSetCheckTimeout": "1.00:00:00",
    "upgradeTimeout": "12:00:00"
}
```

> [!NOTE]
> À la fin de la mise à niveau, n’oubliez pas de restaurer le paramètre `forceRestart` pour réduire l’impact des mises à niveau suivantes. 

## <a name="errors-and-troubleshooting"></a>Erreurs et résolution des problèmes

Si le déploiement échoue avec le message suivant, cela signifie que le cluster ne s’exécute pas sur une version de Service Fabric suffisamment élevée :

```json
{
    "code": "ParameterNotAllowed",
    "message": "Section 'ManagedIdentityTokenService' and Parameter 'IsEnabled' is not allowed."
}
```

## <a name="next-steps"></a>Étapes suivantes
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par le système](./how-to-deploy-service-fabric-application-system-assigned-managed-identity.md)
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Tirer parti de l’identité managée d’une application Service Fabric à partir du code de service](./how-to-managed-identity-service-fabric-app-code.md)
* [Accorder à une application Azure Service Fabric l’accès à d’autres ressources Azure](./how-to-grant-access-other-resources.md)

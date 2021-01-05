---
title: Déployer une application Service Fabric avec une identité managée attribuée par le système
description: Cet article explique comment attribuer une identité managée attribuée par le système à une application Azure Service Fabric
ms.topic: article
ms.date: 07/25/2019
ms.openlocfilehash: c5c7a17c51eee18d9b7276f2c57289a5de5c8181
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81415656"
---
# <a name="deploy-service-fabric-application-with-system-assigned-managed-identity"></a>Déployer une application Azure Service Fabric avec une identité managée attribuée par le système

Pour accéder à la fonctionnalité d’identité managée pour les applications Azure Service Fabric, vous devez d’abord activer le service de jeton d’identité managée sur le cluster. Ce service est responsable de l’authentification des applications Service Fabric à l’aide de leurs identités managées et de l’obtention de jetons d’accès en leur nom. Une fois le service activé, vous pouvez le voir dans Service Fabric Explorer sous la section **Système** dans le volet gauche, exécuté sous le nom **fabric:/System/ManagedIdentityTokenService** à côté d’autre services système.

> [!NOTE] 
> Le déploiement d’applications Service Fabric avec des identités managées est pris en charge à partir de la version d’API `"2019-06-01-preview"`. Vous pouvez également utiliser la même version d’API pour le type d’application, la version de type d’application et les ressources de service. La version minimale du runtime Service Fabric pris en charge est 6.5 CU2. De plus, l’environnement de build/package doit aussi disposer du SDK SF .Net au niveau de la CU2 ou d’une version supérieure

## <a name="system-assigned-managed-identity"></a>Identité managée affectée par le système

### <a name="application-template"></a>Modèle d’application

Pour activer l’application avec une identité managée attribuée par le système, ajoutez la propriété **identity** à la ressource d’application, avec le type **systemAssigned** comme indiqué dans l’exemple ci-dessous :

```json
    {
      "apiVersion": "2019-06-01-preview",
      "type": "Microsoft.ServiceFabric/clusters/applications",
      "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
      "location": "[resourceGroup().location]",
      "dependsOn": [
        "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]"
      ],
      "identity": {
        "type" : "systemAssigned"
      },
      "properties": {
        "typeName": "[parameters('applicationTypeName')]",
        "typeVersion": "[parameters('applicationTypeVersion')]",
        "parameters": {
        }
      }
    }
```
Cette propriété déclare, pour Azure Resource Manager, et l’identité managée et les fournisseurs de ressources Service Fabric, respectivement, que cette ressource doit avoir une identité managée (`system assigned`) implicite.

### <a name="application-and-service-package"></a>Package d’application et de service

1. Mettez à jour le manifeste de l’application pour ajouter un élément **ManagedIdentity** dans la section **Principaux**, contenant une seule entrée comme indiqué ci-dessous :

    **ApplicationManifest.xml**

    ```xml
    <Principals>
      <ManagedIdentities>
        <ManagedIdentity Name="SystemAssigned" />
      </ManagedIdentities>
    </Principals>
    ```
    Cela mappe l’identité assignée à l’application en tant que ressource avec un nom convivial, pour une attribution ultérieure aux services qui composent l’application. 

2. Dans la section **ServiceManifestImport** correspondant au service auquel est affectée l’identité managée, ajoutez un élément **IdentityBindingPolicy**, comme indiqué ci-dessous :

    **ApplicationManifest.xml**

      ```xml
        <ServiceManifestImport>
          <Policies>
            <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="SystemAssigned" />
          </Policies>
        </ServiceManifestImport>
      ```

    Cet élément assigne l’identité de l’application au service; sans cette attribution, le service ne sera pas en mesure d’accéder à l’identité de l’application. Dans l’extrait de code ci-dessus, l’identité `SystemAssigned` (qui est un mot clé réservé) est mappée à la définition du service sous le nom convivial `WebAdmin`.

3. Mettez à jour le manifeste de service pour ajouter un élément **ManagedIdentity** à l’intérieur de la section **Ressources** avec le nom correspondant à la valeur du paramètre `ServiceIdentityRef` de la définition `IdentityBindingPolicy` dans le manifeste de l’application :

    **ServiceManifest.xml**

    ```xml
      <Resources>
        ...
        <ManagedIdentities DefaultIdentity="WebAdmin">
          <ManagedIdentity Name="WebAdmin" />
        </ManagedIdentities>
      </Resources>
    ```
    Il s’agit du mappage équivalent d’une identité à un service, comme décrit ci-dessus, mais du point de vue de la définition de service. L’identité est référencée ici par son nom convivial (`WebAdmin`), comme déclaré dans le manifeste de l’application.

## <a name="next-steps"></a>Étapes suivantes
* Vérifier [la prise en charge des identités managées](./concepts-managed-identity.md) dans Azure Service Fabric
* [Déployer un nouveau](./configure-new-azure-service-fabric-enable-managed-identity.md) cluster Azure Service Fabric avec une prise en charge d’identité managée 
* [Activer une identité managée](./configure-existing-cluster-enable-managed-identity-token-service.md) dans un cluster Azure Service Fabric existant
* Tirer parti de l’identité managée d’une [application Service fabric à partir du code source](./how-to-managed-identity-service-fabric-app-code.md)
* [Déployer une application Azure Service Fabric avec une identité managée attribuée par l’utilisateur](./how-to-deploy-service-fabric-application-user-assigned-managed-identity.md)
* [Accorder à une application Azure Service Fabric l’accès à d’autres ressources Azure](./how-to-grant-access-other-resources.md)

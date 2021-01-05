---
title: Déployer une application avec une identité managée affectée par l’utilisateur
description: Cet article explique comment déployer une application Service Fabric avec une identité managée attribuée par l’utilisateur
ms.topic: article
ms.date: 12/09/2019
ms.openlocfilehash: 79d8654733b580be96d59e78f31105077929ac78
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86260083"
---
# <a name="deploy-service-fabric-application-with-a-user-assigned-managed-identity"></a>Déployer une application Service Fabric avec une identité managée attribuée par l’utilisateur

Pour déployer une application Service Fabric avec une identité managée, l’application doit être déployée via Azure Resource Manager, généralement avec un modèle Azure Resource Manager. Pour plus d’informations sur le déploiement d’une application Service Fabric via Azure Resource Manager, consultez [Gérer des applications et services en tant que ressources Azure Resource Manager](service-fabric-application-arm-resource.md).

> [!NOTE] 
> 
> Les applications qui ne sont pas déployées en tant que ressource Azure **ne peuvent pas avoir** d’identités managées. 
>
> Le déploiement d’application Service Fabric avec une identité managée est pris en charge avec la version d’API `"2019-06-01-preview"`. Vous pouvez également utiliser la même version d’API pour le type d’application, la version de type d’application et les ressources de service.
>

## <a name="user-assigned-identity"></a>Identité attribuée par l’utilisateur

Pour activer l’application avec une identité attribuée par l’utilisateur, ajoutez tout d’abord la propriété **identity** à la ressource d’application avec le type **userAssigned** et les identités attribuées par l’utilisateur référencées. Ajoutez ensuite une section **managedIdentities** dans la section **properties** de la ressource **application** qui contient une liste de noms conviviaux pour le mappage de principalId de chacune des identités attribuées par l’utilisateur. Pour plus d’informations sur les identités attribuées par l’utilisateur, consultez [Créer, lister ou supprimer une identité managée attribuée par l’utilisateur](../active-directory/managed-identities-azure-resources/how-to-manage-ua-identity-powershell.md).

### <a name="application-template"></a>Modèle d’application

Pour activer l’application avec l’identité attribuée par l’utilisateur, ajoutez tout d’abord la propriété **identity** à la ressource d’application avec le type **userAssigned** et les identités attribuées par l’utilisateur référencées, puis ajoutez un objet **managedIdentities** dans la section **properties** qui contient une liste de noms conviviaux pour le mappage de principalId de chacune des identités attribuées par l’utilisateur.

```json
{
  "apiVersion": "2019-06-01-preview",
  "type": "Microsoft.ServiceFabric/clusters/applications",
  "name": "[concat(parameters('clusterName'), '/', parameters('applicationName'))]",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.ServiceFabric/clusters/', parameters('clusterName'), '/applicationTypes/', parameters('applicationTypeName'), '/versions/', parameters('applicationTypeVersion'))]",
    "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]"
  ],
  "identity": {
    "type" : "userAssigned",
    "userAssignedIdentities": {
        "[resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName'))]": {}
    }
  },
  "properties": {
    "typeName": "[parameters('applicationTypeName')]",
    "typeVersion": "[parameters('applicationTypeVersion')]",
    "parameters": {
    },
    "managedIdentities": [
      {
        "name" : "[parameters('userAssignedIdentityName')]",
        "principalId" : "[reference(resourceId('Microsoft.ManagedIdentity/userAssignedIdentities/', parameters('userAssignedIdentityName')), '2018-11-30').principalId]"
      }
    ]
  }
}
```

Dans l’exemple ci-dessus, le nom de ressource de l’identité attribuée par l’utilisateur est utilisé comme nom convivial de l’identité managée pour l’application. Les exemples suivants supposent que le nom convivial réel est « AdminUser ».

### <a name="application-package"></a>Package d’application

1. Pour chaque identité définie dans la section `managedIdentities` du modèle Azure Resource Manager, ajoutez une balise `<ManagedIdentity>` dans le manifeste de l’application sous la section **Principals**. L’attribut `Name` doit correspondre à la propriété `name` définie dans la section `managedIdentities`.

    **ApplicationManifest.xml**

    ```xml
      <Principals>
        <ManagedIdentities>
          <ManagedIdentity Name="AdminUser" />
        </ManagedIdentities>
      </Principals>
    ```

2. Dans la section **ServiceManifestImport**, ajoutez une **IdentityBindingPolicy** pour le service qui utilise l’identité managée. Cette stratégie mappe l’identité `AdminUser` à un nom d’identité spécifique au service qui doit être ajouté ultérieurement dans le manifeste de service.

    **ApplicationManifest.xml**

    ```xml
      <ServiceManifestImport>
        <Policies>
          <IdentityBindingPolicy ServiceIdentityRef="WebAdmin" ApplicationIdentityRef="AdminUser" />
        </Policies>
      </ServiceManifestImport>
    ```

3. Mettez à jour le manifeste de service pour ajouter une **ManagedIdentity** à l’intérieur de la section **Ressources** avec le nom correspondant à `ServiceIdentityRef` dans `IdentityBindingPolicy` du manifeste de l’application :

    **ServiceManifest.xml**

    ```xml
      <Resources>
        ...
        <ManagedIdentities DefaultIdentity="WebAdmin">
          <ManagedIdentity Name="WebAdmin" />
        </ManagedIdentities>
      </Resources>
    ```

## <a name="next-steps"></a>Étapes suivantes

* [Comment utiliser des identités managées dans du code d’application de Service Fabric](how-to-managed-identity-service-fabric-app-code.md)
* [Comment accorder à une application Service Fabric l’accès à d’autres ressources Azure](how-to-grant-access-other-resources.md)

---
title: Gérer les secrets d’application Azure Service Fabric Mesh
description: Gérez les secrets d’application afin de pouvoir créer et déployer une application Service Fabric Mesh de manière sécurisée.
ms.date: 4/2/2019
ms.topic: conceptual
ms.custom: devx-track-azurecli
ms.openlocfilehash: 379560b73c38558fe0b712ed5e036c7a3736b600
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87500706"
---
# <a name="manage-service-fabric-mesh-application-secrets"></a>Gérer les secrets d’application Service Fabric Mesh
Service Fabric Mesh prend en charge les secrets comme des ressources Azure. Un secret Service Fabric Mesh peut correspondre à n’importe quel type de texte sensible, comme des chaînes de connexion de stockage, des mots de passe ou d’autres valeurs devant être stockées et transmises de manière sécurisée. Cet article explique comment utiliser le service Banque d’informations sécurisé de Service Fabric pour déployer et gérer des secrets.

Un secret d’application Mesh comprend :
* Une ressource **Secrets**, qui est un conteneur où sont stockés les secrets texte. Les secrets contenus dans la ressource **Secrets** sont stockés et transmis de manière sécurisée.
* Une ou plusieurs ressources **Secrets/Valeurs** qui sont stockées dans le conteneur de ressources **Secrets**. Chaque ressource **Secrets/Valeurs** se distingue par son numéro de version. Vous ne pouvez pas modifier la version d’une ressource **Secrets/Valeurs**. Vous pouvez seulement ajouter une nouvelle version.

La gestion des secrets comprend les étapes suivantes :
1. Déclarer une ressource **Secrets** Mesh dans un fichier YAML ou JSON de modèle de ressource Azure à l’aide des définitions inlinedValue kind et SecretsStoreRef contentType.
2. Déclarer les ressources **Secrets/Valeurs** Mesh dans un fichier YAML ou JSON de modèle de ressource Azure, qui seront stockées dans la ressource **Secrets** (à l’étape 1).
3. Modifier l’application Mesh pour référencer les valeurs de secret Mesh
4. Déployer ou effectuer une mise à niveau propagée de l’application Mesh pour consommer les valeurs de secret
5. Utiliser les commandes « az » de l’interface CLI Azure pour la gestion du cycle de vie du service Banque d’informations sécurisé

## <a name="declare-a-mesh-secrets-resource"></a>Déclarer une ressource Secrets Mesh
Une ressource Secrets Mesh est déclarée dans un fichier YAML ou JSON de modèle de ressource Azure à l’aide de la définition inlinedValue kind. La ressource Secrets Mesh prend en charge les secrets provenant du service Banque d’informations sécurisé. 
>
Voici un exemple montrant comment déclarer des ressources Secrets Mesh dans un fichier JSON :

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    }
  ]
}
```
Voici un exemple montrant comment déclarer des ressources Secrets Mesh dans un fichier YAML :
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="declare-mesh-secretsvalues-resources"></a>Déclarer des ressources Secrets/valeurs Mesh
Les ressources Secrets/Valeurs Mesh dépendent des ressources Secrets Mesh définies à l’étape précédente.

Concernant la relation qui existe entre les champs « value: » et « name: » de la section « resources » : la deuxième partie de la chaîne « name: » délimitée par un signe deux-points correspond au numéro de version utilisé pour un secret, et le nom qui précède le signe deux-points doit correspondre à la valeur de secret Mesh de laquelle il dépend. Par exemple, pour l’élément ```name: mysecret:1.0```, le numéro de version est 1.0 et le nom ```mysecret``` doit correspondre au ```"value": "mysecret"``` défini précédemment.

>
Voici un exemple montrant comment déclarer des ressources Secrets/Valeurs Mesh dans un fichier JSON :

```json
{
  "$schema": "http://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json",
  "contentVersion": "1.0.0.0",
  "parameters": {
    "location": {
      "type": "string",
      "defaultValue": "WestUS",
      "metadata": {
        "description": "Location of the resources (e.g. westus, eastus, westeurope)."
      }
    }
  },
  "sfbpHttpsCertificate": {
      "type": "string",
      "metadata": {
        "description": "Plain Text Secret Value that your container ingest"
      }
  },
  "resources": [
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx",
      "type": "Microsoft.ServiceFabricMesh/secrets",
      "location": "[parameters('location')]", 
      "dependsOn": [],
      "properties": {
        "kind": "inlinedValue",
        "description": "SFBP Application Secret",
        "contentType": "text/plain",
      }
    },
    {
      "apiVersion": "2018-07-01-preview",
      "name": "sfbpHttpsCertificate.pfx/2019.02.28",
      "type": "Microsoft.ServiceFabricMesh/secrets/values",
      "location": "[parameters('location')]",
      "dependsOn": [
        "Microsoft.ServiceFabricMesh/secrets/sfbpHttpsCertificate.pfx"
      ],
      "properties": {
        "value": "[parameters('sfbpHttpsCertificate')]"
      }
    }
  ],
}
```
Voici un exemple montrant comment déclarer des ressources Secrets/Valeurs Mesh dans un fichier YAML :
```yaml
    services:
      - name: helloWorldService
        properties:
          description: Hello world service.
          osType: linux
          codePackages:
            - name: helloworld
              image: myapp:1.0-alpine
              resources:
                requests:
                  cpu: 2
                  memoryInGB: 2
              endpoints:
                - name: helloWorldEndpoint
                  port: 8080
          Secrets:
            - name: MySecret.txt
            description: My Mesh Application Secret
            secret_type: inlinedValue
            content_type: SecretStoreRef
            value: mysecret
            - name: mysecret:1.0
            description: My Mesh Application Secret Value
            secret_type: value
            content_type: text/plain
            value: "P@ssw0rd#1234"
    replicaCount: 3
    networkRefs:
      - name: mynetwork
```

## <a name="modify-mesh-application-to-reference-mesh-secret-values"></a>Modifier l’application Mesh pour référencer les valeurs de secret Mesh
Les applications Service Fabric Mesh doivent connaître les deux chaînes suivantes afin de consommer les valeurs de secret du service Banque d’informations sécurisé :
1. Microsoft.ServiceFabricMesh/Secrets.name contient le nom du fichier ainsi que la valeur de secret en texte clair.
2. La variable d’environnement « Fabric_SettingPath » Windows ou Linux contient le chemin du répertoire où les fichiers contenant les valeurs de secret du service Banque d’informations sécurisé seront accessibles. Il s’agit de « C:\Settings » pour les applications Mesh hébergées dans Windows, et de « /var/settings » pour les applications Mesh hébergées dans Linux.

## <a name="deploy-or-use-a-rolling-upgrade-for-mesh-application-to-consume-secret-values"></a>Déployer ou effectuer une mise à niveau propagée de l’application Mesh pour consommer les valeurs de secret
La création de ressources Secrets, et/ou de ressources Secrets/Valeurs avec gestion de version, est limitée aux déploiements déclarés par le modèle de ressource. La seule façon de créer ces ressources est de passer un fichier JSON ou YAML de modèle de ressource à l’aide de la commande **az mesh deployment**, en procédant ainsi :

```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```

## <a name="azure-cli-commands-for-secure-store-service-lifecycle-management"></a>Commandes de l’interface CLI Azure pour la gestion du cycle de vie du service Banque d’informations sécurisé

### <a name="create-a-new-secrets-resource"></a>Créer une ressource Secrets
```azurecli-interactive
az mesh deployment create –-<template-file> or --<template-uri>
```
Passez soit **template-file** soit **template-uri** (mais pas les deux).

Par exemple :
- az mesh deployment create --c:\MyMeshTemplates\SecretTemplate1.txt
- az mesh deployment create --https:\//www.fabrikam.com/MyMeshTemplates/SecretTemplate1.txt

### <a name="show-a-secret"></a>Afficher un secret
Retourne la description du secret (mais pas la valeur).
```azurecli-interactive
az mesh secret show --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="delete-a-secret"></a>Supprimer un secret

- Vous ne pouvez pas supprimer un secret tant que celui-ci est référencé par une application Mesh.
- La suppression d’une ressource Secrets supprime également toutes les versions des ressources Secrets/Valeurs.
  ```azurecli-interactive
  az mesh secret delete --Resource-group <myResourceGroup> --secret-name <mySecret>
  ```

### <a name="list-secrets-in-subscription"></a>Lister les secrets d’un abonnement
```azurecli-interactive
az mesh secret list
```
### <a name="list-secrets-in-resource-group"></a>Lister les secrets d’un groupe de ressources
```azurecli-interactive
az mesh secret list -g <myResourceGroup>
```
### <a name="list-all-versions-of-a-secret"></a>Lister toutes les versions d’un secret
```azurecli-interactive
az mesh secretvalue list --Resource-group <myResourceGroup> --secret-name <mySecret>
```

### <a name="show-secret-version-value"></a>Afficher la valeur de version d’un secret
```azurecli-interactive
az mesh secretvalue show --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

### <a name="delete-secret-version-value"></a>Supprimer la valeur de version d’un secret
```azurecli-interactive
az mesh secretvalue delete --Resource-group <myResourceGroup> --secret-name <mySecret> --version <N>
```

## <a name="next-steps"></a>Étapes suivantes 
Pour en savoir plus sur Service Fabric mesh, voir la
- [Vue d’ensemble de Service Fabric mesh](service-fabric-mesh-overview.md)

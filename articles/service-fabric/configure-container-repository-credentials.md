---
title: Azure Service Fabric - Configurer les informations d’identification au référentiel de conteneurs
description: Configurer les informations d’identification au référentiel pour télécharger des images du registre de conteneurs
ms.topic: conceptual
ms.date: 12/09/2019
ms.openlocfilehash: 0c6421fed88a3909db717c13a6b3faf51c4491cd
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96574817"
---
# <a name="configure-repository-credentials-for-your-application-to-download-container-images"></a>Configurer les informations d’identification au référentiel de votre applications pour télécharger des images du registre de conteneurs

Configurez l’authentification du registre de conteneurs en ajoutant des `RepositoryCredentials` à la section `ContainerHostPolicies` du manifeste de votre application. Ajoutez le compte et le mot de passe de votre registre de conteneurs (*myregistry.azurecr.io* dans l’exemple ci-dessous). Cela permet au service de télécharger l’image conteneur à partir du référentiel.

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code">
            <RepositoryCredentials AccountName="myregistry" Password="=P==/==/=8=/=+u4lyOB=+=nWzEeRfF=" PasswordEncrypted="false"/>
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

Il est recommandé de chiffrer le mot de passe du référentiel à l’aide d’un certificat de chiffrement qui est déployé sur tous les nœuds du cluster. Lorsque Service Fabric déploie le package de service sur le cluster, le certificat de chiffrement est utilisé pour déchiffrer le texte chiffré. Le cmdlet Invoke-ServiceFabricEncryptText est utilisé pour créer le texte chiffré du mot de passe, qui est ensuite ajouté au fichier ApplicationManifest.xml.
Pour plus d’informations sur les certificats et la sémantique du chiffrement, consultez [Gestion des secrets](service-fabric-application-secret-management.md).

## <a name="configure-cluster-wide-credentials"></a>Configurer les informations d’identification au niveau du cluster

Service Fabric vous permet de configurer des informations d’identification au niveau du cluster qui peuvent être utilisées comme informations d’identification de référentiel par défaut par les applications.

Cette fonctionnalité peut être activée ou désactivée en ajoutant l’attribut `UseDefaultRepositoryCredentials` à `ContainerHostPolicies` dans ApplicationManifest.xml avec une valeur `true` ou `false`.

```xml
<ServiceManifestImport>
    ...
    <Policies>
        <ContainerHostPolicies CodePackageRef="Code" UseDefaultRepositoryCredentials="true">
            <PortBinding ContainerPort="80" EndpointRef="Guest1TypeEndpoint"/>
        </ContainerHostPolicies>
    </Policies>
    ...
</ServiceManifestImport>
```

Service Fabric utilise ensuite les informations d’identification de référentiel par défaut pouvant être spécifiées dans le manifeste de cluster sous la section `Hosting`.  Si `UseDefaultRepositoryCredentials` est `true`, Service Fabric lit les valeurs suivantes dans le manifeste de cluster :

* DefaultContainerRepositoryAccountName (chaîne)
* DefaultContainerRepositoryPassword (chaîne)
* IsDefaultContainerRepositoryPasswordEncrypted (valeur booléenne)
* DefaultContainerRepositoryPasswordType (chaîne)

Voici un exemple de ce qui peut être ajouté dans la section `Hosting` du fichier ClusterManifestTemplate.json. La section `Hosting` peut être ajoutée au moment de la création du cluster ou ultérieurement dans une mise à niveau de la configuration. Pour plus d’informations, consultez [Personnaliser les paramètres de cluster Service Fabric](service-fabric-cluster-fabric-settings.md) et [Gérer les secrets dans les applications Service Fabric](service-fabric-application-secret-management.md).

```json
"fabricSettings": [
    ...,
    {
        "name": "Hosting",
        "parameters": [
          {
            "name": "EndpointProviderEnabled",
            "value": "true"
          },
          {
            "name": "DefaultContainerRepositoryAccountName",
            "value": "someusername"
          },
          {
            "name": "DefaultContainerRepositoryPassword",
            "value": "somepassword"
          },
          {
            "name": "IsDefaultContainerRepositoryPasswordEncrypted",
            "value": "false"
          },
          {
            "name": "DefaultContainerRepositoryPasswordType",
            "value": "PlainText"
          }
        ]
      },
]
```

## <a name="use-tokens-as-registry-credentials"></a>Utiliser des jetons en tant qu’informations d’identification du Registre

Service Fabric prend en charge l’utilisation de jetons en tant qu’informations d’identification pour télécharger des images pour vos conteneurs.  Cette fonctionnalité tire parti de l’*identité managée* du groupe de machines virtuelles identiques sous-jacent pour s’authentifier auprès du registre, ce qui évite d’avoir à gérer les informations d’identification des utilisateurs.  Pour plus d’informations, voir [Identités managées pour les ressources Azure](../active-directory/managed-identities-azure-resources/overview.md).  L’utilisation de cette fonctionnalité requiert les étapes suivantes :

1. Vérifiez que l’option *Identité managée affectée par le système* est activée pour la machine virtuelle.

    ![Portail Azure : Créer une option d’identité de groupe de machines virtuelles identiques](./media/configure-container-repository-credentials/configure-container-repository-credentials-acr-iam.png)

> [!NOTE]
> Pour une identité managée affectée par l’utilisateur, ignorez cette étape. Les étapes restantes ci-dessous fonctionnent de la même manière, à condition que le groupe identique ne soit associé qu’à une seule identité managée affectée par l’utilisateur.

2. Accordez au groupe de machines virtuelles identiques les autorisations nécessaires pour extraire/lire des images à partir du Registre. Dans le panneau Access Control (IAM) de votre Azure Container Registry sur le Portail Azure, ajoutez une *attribution de rôle* pour votre machine virtuelle :

    ![Ajouter un principal de machine virtuelle à ACR](./media/configure-container-repository-credentials/configure-container-repository-credentials-vmss-identity.png)

3. Ensuite, modifiez le manifeste de votre application. Dans la section `ContainerHostPolicies`, ajoutez l’attribut `‘UseTokenAuthenticationCredentials=”true”`.

    ```xml
      <ServiceManifestImport>
          <ServiceManifestRef ServiceManifestName="NodeServicePackage" ServiceManifestVersion="1.0"/>
      <Policies>
        <ContainerHostPolicies CodePackageRef="NodeService.Code" Isolation="process" UseTokenAuthenticationCredentials="true">
          <PortBinding ContainerPort="8905" EndpointRef="Endpoint1"/>
        </ContainerHostPolicies>
        <ResourceGovernancePolicy CodePackageRef="NodeService.Code" MemoryInMB="256"/>
      </Policies>
      </ServiceManifestImport>
    ```

    > [!NOTE]
    > L’indicateur `UseDefaultRepositoryCredentials` défini sur true alors que `UseTokenAuthenticationCredentials` a pour valeur true entraîne une erreur pendant le déploiement.

## <a name="next-steps"></a>Étapes suivantes

* Apprenez-en davantage sur l’[authentification au registre de conteneurs](../container-registry/container-registry-authentication.md).

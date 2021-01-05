---
title: Spécifier des variables d’environnement pour des services
description: Vous montre comment utiliser des variables d’environnement pour des applications dans Service Fabric
ms.topic: conceptual
ms.date: 12/06/2017
ms.openlocfilehash: 610e44ac98b8b8a2ce5a91fdbcfda145ae36a94b
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576738"
---
# <a name="how-to-specify-environment-variables-for-services-in-service-fabric"></a>Guide pratique pour spécifier des variables d’environnement pour des services dans Service Fabric

Cet article vous explique comment spécifier des variables d’environnement pour un service ou un conteneur dans Service Fabric.

## <a name="procedure-for-specifying-environment-variables-for-services"></a>Procédure permettant de spécifier des variables d’environnement pour des services

Dans cet exemple, vous définissez une variable d’environnement pour un conteneur. Cet article suppose que vous disposez déjà d’un manifeste d’application et de service.

1. Ouvrez le fichier ServiceManifest.xml.
2. Dans l’élément `CodePackage`, ajoutez un nouveau élément `EnvironmentVariables` et un élément `EnvironmentVariable` pour chaque variable d’environnement.

    ```xml
    <CodePackage Name="MyCode" Version="CodeVersion1">
            ...
            <EnvironmentVariables>
                  <EnvironmentVariable Name="MyEnvVariable" Value="DefaultValue"/>
                  <EnvironmentVariable Name="HttpGatewayPort" Value="19080"/>
            </EnvironmentVariables>
    </CodePackage>
    ```

   Les variables d’environnement peuvent être remplacées dans le manifeste de l’application.

3. Pour remplacer les variables d’environnement dans le manifeste de l’application, utilisez l’élément `EnvironmentOverrides`.

    ```xml
      <ServiceManifestImport>
        <ServiceManifestVersion="1.0.0" />
        <EnvironmentOverrides CodePackageRef="MyCode">
          <EnvironmentVariable Name="MyEnvVariable" Value="OverrideValue"/>
        </EnvironmentOverrides>
      </ServiceManifestImport>
    ```

## <a name="specifying-environment-variables-dynamically-using-docker-compose"></a>Spécification de variables d’environnement dynamiques à l’aide de Docker Compose

Service Fabric prend en charge la possibilité d’[utiliser Docker Compose pour le déploiement](service-fabric-docker-compose.md#supported-compose-directives). Les fichiers Compose peuvent approvisionner les variables d’environnement à partir de l’interpréteur de commandes. Ce comportement peut être utilisé pour remplacer dynamiquement les valeurs d’environnement souhaitées :

```yml
environment:
  - "hostname:${hostname}"
```

## <a name="next-steps"></a>Étapes suivantes
Pour en savoir plus sur certains des principaux concepts abordés dans cet article, consultez les articles [Gérer des applications pour plusieurs environnements](service-fabric-manage-multiple-environment-app-configuration.md).

Pour plus d’informations sur les autres fonctionnalités de gestion d’application disponibles dans Visual Studio, consultez la section [Gestion de vos applications de Service Fabric dans Visual Studio](service-fabric-manage-application-in-visual-studio.md).

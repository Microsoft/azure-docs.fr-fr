---
title: API du client Java Azure Service Fabric
description: Générez et utilisez des API du client Java Service Fabric à l’aide de la spécification de l’API REST du client Service Fabric
author: rapatchi
ms.topic: conceptual
ms.date: 11/27/2017
ms.custom: devx-track-java
ms.author: rapatchi
ms.openlocfilehash: 24ee4a3d8109626bf93e01fbd10c00140762c9c6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87324622"
---
# <a name="azure-service-fabric-java-client-apis"></a>API du client Java Azure Service Fabric

Les API du client Service Fabric permettent de déployer et de gérer des conteneurs et applications basées sur des microservices dans un cluster Service Fabric sur Azure, en local, sur l’ordinateur de développement local ou dans un autre cloud. Cet article décrit comment générer et utiliser des API du client Java Service Fabric sur les API REST du client Service Fabric

## <a name="generate-the-client-code-using-autorest"></a>Générer le code client avec AutoRest

[AutoRest](https://github.com/Azure/autorest) est un outil qui génère des bibliothèques clientes pour accéder aux services web RESTful. L’entrée d’AutoRest est une spécification qui décrit l’API REST en utilisant le format de spécification OpenAPI. Les [API REST du client Service Fabric](https://github.com/Azure/azure-rest-api-specs/tree/master/specification/servicefabric/data-plane) suivent cette spécification.

Suivez les étapes indiquées ci-dessous pour générer le code client Java Service Fabric à l’aide de l’outil AutoRest.

1. Installer nodejs et NPM sur votre machine

    Si vous utilisez Linux :
    ```bash
    sudo apt-get install npm
    sudo apt install nodejs
    ```
    Si vous utilisez Mac OS X :
    ```bash
    brew install node
    ```

2. Installez AutoRest à l’aide de NPM.
    ```bash
    npm install -g autorest
    ```

3. Dupliquez (fork) et clonez le dépôt [azure-rest-api-specs](https://github.com/Azure/azure-rest-api-specs) sur votre ordinateur local et accédez à l’emplacement cloné à partir du terminal de votre ordinateur.


4. Accédez à l’emplacement indiqué ci-dessous dans votre dépôt cloné.
    ```bash
    cd specification\servicefabric\data-plane\Microsoft.ServiceFabric\stable\6.0
    ```

    > [!NOTE]
    > Si votre version de cluster n’est pas 6.0.*, accédez au répertoire approprié dans le dossier stable.
    >   

5. Exécutez la commande autorest suivante pour générer le code client Java.
    
    ```bash
    autorest --input-file= servicefabric.json --java --output-folder=[output-folder-name] --namespace=[namespace-of-generated-client]
    ```
   Voici un exemple illustrant l’utilisation d’autorest.
   
    ```bash
    autorest --input-file=servicefabric.json --java --output-folder=java-rest-api-code --namespace=servicefabricrest
    ```
   
   La commande suivante prend le fichier de spécification ``servicefabric.json`` comme entrée, génère le code client Java dans le dossier ``java-rest-api-     code`` et place le code dans l’espace de noms ``servicefabricrest``. À la fin de cette étape, deux dossiers (``models`` et ``implementation``) et deux fichiers (``ServiceFabricClientAPIs.java`` et ``package-info.java``) sont générés dans le dossier ``java-rest-api-code``.


## <a name="include-and-use-the-generated-client-in-your-project"></a>Inclure et utiliser le client généré dans votre projet

1. Ajoutez correctement le code généré dans votre projet. Nous vous recommandons de créer une bibliothèque avec le code généré et d’inclure cette bibliothèque dans votre projet.
2. Si vous créez une bibliothèque, ajoutez la dépendance suivante dans le projet de votre bibliothèque. Si vous suivez une approche différente, ajoutez la dépendance de manière appropriée.

    ```
        GroupId:  com.microsoft.rest
        Artifactid: client-runtime
        Version: 1.2.1
    ```
    Par exemple, si vous utilisez le système de build Maven, ajoutez le code suivant à votre fichier ``pom.xml`` :

    ```xml
        <dependency>
          <groupId>com.microsoft.rest</groupId>
          <artifactId>client-runtime</artifactId>
          <version>1.2.1</version>
        </dependency>
    ```

3. Créez un RestClient avec le code suivant :

    ```java
        RestClient simpleClient = new RestClient.Builder()
            .withBaseUrl("http://<cluster-ip or name:port>")
            .withResponseBuilderFactory(new ServiceResponseBuilder.Factory())
            .withSerializerAdapter(new JacksonAdapter())
            .build();
        ServiceFabricClientAPIs client = new ServiceFabricClientAPIsImpl(simpleClient);
    ```
4. Utilisez l’objet client et effectuez les appels appropriés en fonction des besoins. Voici quelques exemples qui illustrent l’utilisation de l’objet client. Nous partons du principe que le package d’application est créé et chargé sur le magasin d’images avant d’utiliser les API ci-dessous.
    * Provisionner une application
    
        ```java
            ApplicationTypeImageStorePath imageStorePath = new ApplicationTypeImageStorePath();
            imageStorePath.withApplicationTypeBuildPath("<application-path-in-image-store>");
            client.provisionApplicationType(imageStorePath);
        ```
    * Créer une application

        ```java
            ApplicationDescription applicationDescription = new ApplicationDescription();
            applicationDescription.withName("<application-uri>");
            applicationDescription.withTypeName("<application-type>");
            applicationDescription.withTypeVersion("<application-version>");
            client.createApplication(applicationDescription);
        ```

## <a name="understanding-the-generated-code"></a>Présentation du code généré
Pour chaque API, vous trouverez quatre surcharges d’implémentation. S’il existe des paramètres facultatifs, vous trouverez quatre variantes supplémentaires, dont ces paramètres facultatifs. Prenons comme exemple l’API ``removeReplica``.
 1. **public void removeReplica(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout)**
    * Il s’agit de la variante synchrone de l’appel d’API removeReplica
 2. **public ServiceFuture\<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId, Boolean forceRemove, Long timeout, final ServiceCallback\<Void> serviceCallback)**
    * Cette variante de l’appel d’API peut être utilisée si vous souhaitez employer une programmation asynchrone basée sur le futur et utiliser des rappels
 3. **public Observable\<Void> removeReplicaAsync(String nodeName, UUID partitionId, String replicaId)**
    * Cette variante de l’appel d’API peut être utilisée si vous souhaitez employer une programmation asynchrone réactive
 4. **public Observable\<ServiceResponse\<Void>> removeReplicaWithServiceResponseAsync(String nodeName, UUID partitionId, String replicaId)**
    * Cette variante de l’appel d’API peut être utilisée si vous souhaitez employer une programmation asynchrone réactive et traiter la réponse REST RAW

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur les [API REST Service Fabric](/rest/api/servicefabric/)

---
title: Se connecter à l’API Azure Media Services v3 - Java
description: Cet article explique comment se connecter à l’API Azure Media Services v3 avec Java.
services: media-services
documentationcenter: ''
author: IngridAtMicrosoft
manager: femila
editor: ''
ms.service: media-services
ms.workload: media
ms.tgt_pltfrm: na
ms.devlang: na
ms.topic: quickstart
ms.date: 11/17/2020
ms.custom: devx-track-java
ms.author: inhenkel
ms.openlocfilehash: 769b4bc431040ee4d872fa60270196db96978ed2
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94748177"
---
# <a name="connect-to-media-services-v3-api---java"></a>Se connecter à l’API Media Services v3 - Java

[!INCLUDE [media services api v3 logo](./includes/v3-hr.md)]

Cet article vous montre comment vous connecter au SDK Java Azure Media Services v3 en utilisant la méthode de connexion au principal du service.

Dans cet article, Visual Studio Code est utilisé pour développer l’exemple d’application.

## <a name="prerequisites"></a>Prérequis

- Suivez [Écriture de Java avec Visual Studio Code](https://code.visualstudio.com/docs/java/java-tutorial) pour installer les éléments ci-après :

   - JDK
   - Apache Maven
   - Pack d’extension Java
- Veillez à définir les variables d’environnement `JAVA_HOME` et `PATH`.
- [Créer un compte Media Services](./create-account-howto.md). Veillez à mémoriser le nom du groupe de ressources et le nom du compte Media Services.
- Suivez les étapes de l’article [Accéder aux API](./access-api-howto.md). Enregistrez l’ID d’abonnement, l’ID d’application (ID client), la clé d’authentification (secret), et l’ID locataire dont vous aurez besoin dans une prochaine étape.

Consultez également :

- [Java dans Visual Studio Code](https://code.visualstudio.com/docs/languages/java)
- [Gestion de projets Java dans VS Code](https://code.visualstudio.com/docs/java/java-project)

> [!IMPORTANT]
> Examinez les [conventions d’appellation](media-services-apis-overview.md#naming-conventions).

## <a name="create-a-maven-project"></a>Création d’un projet Maven

Ouvrez un outil de ligne de commande et `cd` dans un répertoire où vous souhaitez créer le projet.
    
```
mvn archetype:generate -DgroupId=com.azure.ams -DartifactId=testAzureApp -DarchetypeArtifactId=maven-archetype-quickstart -DinteractiveMode=false
```

Lorsque vous exécutez la commande, les fichiers `pom.xml` et `App.java` et les autres fichiers sont créés. 

## <a name="add-dependencies"></a>Ajout de dépendances

1. Dans Visual Studio Code, ouvrez le dossier où se trouve votre projet
1. Recherchez et ouvrez le fichier `pom.xml`
1. Ajoutez les dépendances nécessaires.

   Consultez `pom.xml` dans l’exemple d’[encodage vidéo](https://github.com/Azure-Samples/media-services-v3-java/blob/master/VideoEncoding/EncodingWithMESCustomPreset/pom.xml).

## <a name="connect-to-the-java-client"></a>Connectez-vous au client Java

1. Ouvrez le fichier `App.java` situé sous `src\main\java\com\azure\ams` et vérifiez que votre package se trouve en haut :

    ```java
    package com.azure.ams;
    ```
1. Sous l’instruction package, ajoutez ces instructions import :
   
   ```java
   import com.microsoft.azure.AzureEnvironment;
   import com.microsoft.azure.credentials.ApplicationTokenCredentials;
   import com.microsoft.azure.management.mediaservices.v2018_07_01.implementation.MediaManager;
   import com.microsoft.rest.LogLevel;
   ```
1. Pour créer les informations d’identification Active Directory dont vous avez besoin pour faire des requêtes, ajoutez le code suivant à la méthode principale de la classe App et définissez les valeurs que vous avez obtenues auprès des [API Access](./access-api-howto.md) :
   
   ```java
   final String clientId = "00000000-0000-0000-0000-000000000000";
   final String tenantId = "00000000-0000-0000-0000-000000000000";
   final String clientSecret = "00000000-0000-0000-0000-000000000000";
   final String subscriptionId = "00000000-0000-0000-0000-000000000000";

   try {
      ApplicationTokenCredentials credentials = new ApplicationTokenCredentials(clientId, tenantId, clientSecret, AzureEnvironment.AZURE);
      credentials.withDefaultSubscriptionId(subscriptionId);

      MediaManager manager = MediaManager
              .configure()
              .withLogLevel(LogLevel.BODY_AND_HEADERS)
              .authenticate(credentials, credentials.defaultSubscriptionId());
      System.out.println("signed in");
   }
   catch (Exception e) {
      System.out.println("Exception encountered.");
      System.out.println(e.toString());
   }
   ```
1. Exécutez l'application.

## <a name="see-also"></a>Voir aussi

- [Concepts Media Services](concepts-overview.md)
- [Kit SDK Java](https://aka.ms/ams-v3-java-sdk)
- [Référence Java](/java/api/overview/azure/mediaservices/management)
- [com.microsoft.azure.mediaservices.v2018_07_01:azure-mgmt-media](https://search.maven.org/artifact/com.microsoft.azure.mediaservices.v2018_07_01/azure-mgmt-media/1.0.0-beta/jar)

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez maintenant inclure `import com.microsoft.azure.management.mediaservices.v2018_07_01.*;` et commencer à manipuler les entités.

Pour obtenir plus d’exemples de code, consultez le dépôt d’[exemples du Kit SDK Java](/samples/azure-samples/media-services-v3-java/azure-media-services-v3-samples-using-java/).

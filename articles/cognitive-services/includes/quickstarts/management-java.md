---
title: 'Démarrage rapide : Bibliothèque de client de gestion Azure pour Java'
description: Utilisez ce guide de démarrage rapide pour vous familiariser avec la bibliothèque de client de gestion Azure pour Java.
services: cognitive-services
author: PatrickFarley
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 09/01/2020
ms.author: pafarley
ms.openlocfilehash: ed85faf9ffe38489eb00ed5a71b3eb841c399d24
ms.sourcegitcommit: 44844a49afe8ed824a6812346f5bad8bc5455030
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/23/2020
ms.locfileid: "97745704"
---
[Documentation de référence](/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable) | [Code source de la bibliothèque](https://github.com/Azure/azure-sdk-for-java/tree/master/sdk/cognitiveservices/mgmt-v2017_04_18/src/main/java/com/microsoft/azure/management/cognitiveservices/v2017_04_18) | [Package (Maven)](https://mvnrepository.com/artifact/com.microsoft.azure/azure-mgmt-cognitiveservices)

## <a name="java-prerequisites"></a>Configuration Java requise

* Un abonnement Azure valide - [Créer un abonnement gratuitement](https://azure.microsoft.com/free/)
* La version actuelle du [JDK (Java Development Kit)](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
* L’[outil de génération Gradle](https://gradle.org/install/) ou un autre gestionnaire de dépendances.


[!INCLUDE [Create a service principal](./create-service-principal.md)]

[!INCLUDE [Create a resource group](./create-resource-group.md)]

## <a name="create-a-new-java-application"></a>Créer une application Java

Dans une fenêtre de console (telle que cmd, PowerShell ou bash), créez un répertoire pour votre application et accédez-y. 

```console
mkdir myapp && cd myapp
```

Exécutez la commande `gradle init` à partir de votre répertoire de travail. Cette commande crée des fichiers de build essentiels pour Gradle, notamment *build.gradle.kts*, qui est utilisé au moment de l’exécution pour créer et configurer votre application.

```console
gradle init --type basic
```

Quand vous êtes invité à choisir un **DSL**, sélectionnez **Kotlin**.

Entrez la commande suivante à partir de votre répertoire de travail :

```console
mkdir -p src/main/java
```

### <a name="install-the-client-library"></a>Installer la bibliothèque de client

Ce guide de démarrage rapide utilise le gestionnaire de dépendances Gradle. Vous trouverez la bibliothèque de client et des informations concernant d’autres gestionnaires de dépendances sur le [référentiel central Maven](https://mvnrepository.com/artifact/com.azure/azure-ai-formrecognizer).

Dans le fichier *build.gradle.kts* de votre projet, incluez la bibliothèque de client sous la forme d’une instruction `implementation`, avec les plug-ins et les paramètres obligatoires.

```kotlin
plugins {
    java
    application
}
application {
    mainClass.set("FormRecognizer")
}
repositories {
    mavenCentral()
}
dependencies {
    implementation(group = "com.microsoft.azure", name = "azure-mgmt-cognitiveservices", version = "1.10.0-beta")
}
```

### <a name="import-libraries"></a>Importer des bibliothèques

Accédez au nouveau dossier **src/main/java**, puis créez un fichier appelé *Management.java*. Ouvrez-le dans votre éditeur ou IDE habituel et ajoutez les instructions `import` suivantes :

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_imports)]

## <a name="authenticate-the-client"></a>Authentifier le client

Ajoutez une classe dans *Management.java*, puis ajoutez-y les champs suivants et leurs valeurs. Renseignez les valeurs appropriées à l’aide du principal de service que vous avez créé et des autres informations de votre compte Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_constants)]

Ensuite, dans votre méthode **main**, utilisez ces valeurs pour construire un objet **CognitiveServicesManager**. Cet objet est nécessaire pour toutes vos opérations de gestion Azure.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_auth)]

## <a name="call-management-methods"></a>Appeler les méthodes de gestion

Ajoutez le code suivant à votre méthode **Main** pour répertorier les ressources disponibles, créer un exemple de ressource, répertorier les ressources que vous possédez, puis supprimer l'exemple de ressource. Vous définirez ces méthodes au cours des étapes suivantes.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_calls)]

## <a name="create-a-cognitive-services-resource"></a>Créer une ressource Cognitive Services

Pour créer une ressource Cognitive Services et vous y abonner, utilisez la méthode **create**. Cette méthode ajoute une nouvelle ressource facturable au groupe de ressources que vous transmettez. Lorsque vous créez votre nouvelle ressource, vous devez connaître le « type » du service que vous souhaitez utiliser, ainsi que son niveau tarifaire (ou référence SKU) et un emplacement Azure : La méthode suivante utilise tous ces arguments et crée une ressource.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_create)]

### <a name="choose-a-service-and-pricing-tier"></a>Choisir un service et son niveau tarifaire

Lorsque vous créez une nouvelle ressource, vous devez connaître le « type » du service que vous souhaitez utiliser, ainsi que le [niveau tarifaire](https://azure.microsoft.com/pricing/details/cognitive-services/) (ou référence SKU) souhaité. Ces informations et d'autres vous serviront de paramètres lors de la création de la ressource. Vous pouvez voir la liste des « types » de ressources Cognitive Services en appelant la méthode suivante :

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list_avail)]

[!INCLUDE [cognitive-services-subscription-types](../../../../includes/cognitive-services-subscription-types.md)]

[!INCLUDE [SKUs and pricing](./sku-pricing.md)]

## <a name="view-your-resources"></a>Afficher vos ressources

Pour afficher toutes les ressources disponibles sous votre compte Azure (dans tous les groupes de ressources), utilisez la méthode suivante :

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_list)]

## <a name="delete-a-resource"></a>Supprimer une ressource

La méthode suivante supprime la ressource spécifiée du groupe de ressources donné.

[!code-java[](~/cognitive-services-quickstart-code/java/azure_management_service/quickstart.java?name=snippet_delete)]

## <a name="see-also"></a>Voir aussi

* [Documentation de référence du SDK de gestion Azure](/java/api/com.microsoft.azure.management.cognitiveservices?view=azure-java-stable)
* [Présentation d’Azure Cognitive Services](../../what-are-cognitive-services.md)
* [Authentifier des requêtes auprès d’Azure Cognitive Services](../../authentication.md)
* [Créer une ressource en utilisant le portail Azure](../../cognitive-services-apis-create-account.md)
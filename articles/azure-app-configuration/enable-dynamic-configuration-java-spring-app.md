---
title: Utiliser la configuration dynamique dans une application Spring Boot
titleSuffix: Azure App Configuration
description: Découvrez comment mettre à jour dynamiquement les données de configuration pour les applications Spring Boot.
services: azure-app-configuration
author: AlexandraKemperMS
ms.service: azure-app-configuration
ms.topic: tutorial
ms.date: 08/06/2020
ms.custom: devx-track-java
ms.author: alkemper
ms.openlocfilehash: c32e928bd4a83b4884c99e3ec3a9c647f5433e87
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96929155"
---
# <a name="tutorial-use-dynamic-configuration-in-a-java-spring-app"></a>Tutoriel : Utiliser la configuration dynamique dans une application Java Spring

La bibliothèque de client Spring Boot App Configuration permet d’effectuer la mise à jour à la demande d’un ensemble de paramètres de configuration, sans entraîner le redémarrage de l’application. La bibliothèque de client met en cache chaque paramètre pour éviter un trop grand nombre d’appels au magasin de configurations. Tant que la valeur mise en cache d’un paramètre n’a pas expiré, l’opération d’actualisation ne met pas à jour la valeur, même si celle-ci a été modifiée dans le magasin de configurations. Par défaut, le délai d’expiration de chaque requête est de 30 secondes. Au besoin, ce délai peut être remplacé.

Vous pouvez vérifier à la demande les paramètres mis à jour en appelant la méthode `refreshConfigurations()` de `AppConfigurationRefresh`.

Vous pouvez également utiliser le package `spring-cloud-azure-appconfiguration-config-web`, qui prend une dépendance sur `spring-web` pour gérer l’actualisation automatisée.

## <a name="use-automated-refresh"></a>Utiliser l’actualisation automatisée

Pour utiliser l’actualisation automatisée, démarrez par une application Spring Boot qui utilise App Configuration, telle que l’application que vous pouvez créer en suivant le [guide de démarrage rapide Spring Boot pour App Configuration](quickstart-java-spring-app.md).

Ensuite, ouvrez le fichier *pom.xml* dans un éditeur de texte, puis ajoutez un `<dependency>` pour `spring-cloud-azure-appconfiguration-config-web`.

**Spring Cloud 1.1.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.1.5</version>
</dependency>
```

**Spring Cloud 1.2.x**

```xml
<dependency>
    <groupId>com.microsoft.azure</groupId>
    <artifactId>spring-cloud-azure-appconfiguration-config-web</artifactId>
    <version>1.2.7</version>
</dependency>
```

## <a name="run-and-test-the-app-locally"></a>Exécuter et tester l’application localement

1. Générez votre application Spring Boot avec Maven, puis exécutez-la.

    ```shell
    mvn clean package
    mvn spring-boot:run
    ```

1. Ouvrez une fenêtre de navigateur, puis accédez à l’URL : `http://localhost:8080`.  Le message associé à votre clé s’affiche. 

    Vous pouvez également utiliser *curl* pour tester votre application, par exemple : 
    
    ```cmd
    curl -X GET http://localhost:8080/
    ```

1. Pour tester la configuration dynamique, ouvrez le portail Azure App Configuration associé à votre application. Sélectionnez l’**Explorateur de configurations**, et mettez à jour la valeur de votre clé affichée, par exemple :
    | Clé | Valeur |
    |---|---|
    | application/config.message | Hello – Mis à jour |

1. Actualisez la page du navigateur pour afficher le nouveau message.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez permis à votre application Spring Boot d’actualiser dynamiquement les paramètres de configuration à partir d’App Configuration. Pour savoir comment utiliser une identité managée Azure afin de simplifier l’accès à App Configuration, passez au tutoriel suivant.

> [!div class="nextstepaction"]
> [Intégration des identités managées](./howto-integrate-azure-managed-service-identity.md)

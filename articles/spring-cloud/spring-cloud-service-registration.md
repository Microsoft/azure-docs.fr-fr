---
title: Automatiser le registre et la découverte de services
description: Découvrez comment automatiser la découverte et l’inscription de services à l’aide de Spring Cloud Service Registry.
author: bmitchell287
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 09/08/2020
ms.author: brendm
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 1e60799878cc30b729344c03df36a4c5e4f4a199
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90904215"
---
# <a name="discover-and-register-your-spring-cloud-services"></a>Découvrir et inscrire vos services Spring Cloud

La découverte de services est une condition essentielle pour une architecture basée sur des microservices.  La configuration manuelle de chaque client prend du temps et introduit le risque d’erreur humaine.  Spring Cloud Service Registry résout ce problème.  Une fois configuré, un serveur Service Registry contrôle l’inscription et la découverte des services pour les microservices de votre application. Le serveur Service Registry gère un registre des microservices déployés, permet l’équilibrage de charge côté client et découple les fournisseurs de services des clients sans dépendre du système DNS.

::: zone pivot="programming-language-csharp"
Pour plus d’informations sur la configuration d’une inscription au service pour une application Steeltoe, consultez [Préparer une application Spring Java pour le déploiement dans Azure Spring Cloud](spring-cloud-tutorial-prepare-app-deployment.md).
::: zone-end
::: zone pivot="programming-language-java"

## <a name="register-your-application-using-spring-cloud-service-registry"></a>Inscrire votre application à l’aide de Spring Cloud Service Registry

Pour que votre application puisse gérer l’inscription et la découverte des services à l’aide de Spring Cloud Service Registry, plusieurs dépendances doivent être incluses dans le fichier *pom.xml* de l’application.
Incluez des dépendances pour *spring-cloud-starter-netflix-eureka-client* et *spring-cloud-starter-azure-spring-cloud-client* à votre fichier *pom.xml*.

```xml
    <dependency>
        <groupId>org.springframework.cloud</groupId>
        <artifactId>spring-cloud-starter-netflix-eureka-client</artifactId>
    </dependency>
    <dependency>
        <groupId>com.microsoft.azure</groupId>
        <artifactId>spring-cloud-starter-azure-spring-cloud-client</artifactId>
        <version>2.1.0</version>
    </dependency>
```

## <a name="update-the-top-level-class"></a>Mettre à jour la classe de niveau supérieur

Pour finir, ajoutez une annotation à la classe de niveau supérieur de votre application.

 ```java
    package foo.bar;

    import org.springframework.boot.SpringApplication;
    import org.springframework.cloud.client.SpringCloudApplication;

    @SpringCloudApplication
    public class DemoApplication {
        public static void main(String... args) {
            SpringApplication.run(DemoApplication.class, args);
        }
    }
 ```

Le point de terminaison de serveur Spring Cloud Service Registry est injecté en tant que variable d’environnement dans votre application.  Les microservices pourront désormais s’inscrire eux-mêmes auprès du serveur Service Registry et découvrir d’autres microservices dépendants.

Notez que quelques minutes peuvent être nécessaires pour que les modifications se propagent du serveur à tous les microservices.
::: zone-end

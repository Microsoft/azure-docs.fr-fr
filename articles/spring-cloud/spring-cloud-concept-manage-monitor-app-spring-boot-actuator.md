---
title: Gérer et superviser une application avec l’actionneur Azure Spring Boot
description: Découvrez comment gérer et surveiller une application avec l’actionneur Spring Boot.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: conceptual
ms.date: 05/20/2020
ms.custom: devx-track-java
ms.openlocfilehash: 93fd286aa76a0409a515abbf8c9dabd88a9a65c4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90904283"
---
# <a name="manage-and-monitor-app-with-azure-spring-boot-actuator"></a>Gérer et superviser une application avec l’actionneur Azure Spring Boot

**Cet article s’applique à :** ✔️ Java

Après avoir déployé un nouveau binaire dans votre application, vous pouvez vérifier la fonctionnalité et afficher des informations sur votre application en cours d’exécution. Cet article explique comment accéder à l’API à partir d’un point de terminaison de test fourni par Azure Spring Cloud et exposer les fonctionnalités prêtes pour la production de votre application.

## <a name="prerequisites"></a>Prérequis
Cet article suppose que vous disposez d’une application Spring Boot 2.x qui peut être déployée et démarrée sur le service Azure Spring Cloud.  Consultez [Démarrage rapide : Lancer une application Azure Spring Cloud existante à partir du portail Azure](spring-cloud-quickstart.md).

## <a name="verify-app-through-test-endpoint"></a>Vérifier l’application via le point de terminaison de test
1. Accédez au **tableau de bord de l’application** et cliquez sur votre application pour accéder à la page de présentation de l’application.

1. Dans le volet **Vue d’ensemble**, vous devez voir **Point de terminaison de test**.  Accédez à ce point de terminaison à partir de la ligne de commande ou du navigateur et observez la réponse de l’API.

1. Notez l’URI du **point de terminaison de test** qui sera utilisé dans la section à venir.

>[!TIP]
> * Si l’application retourne une page frontale et référence d’autres fichiers par le biais d’un chemin d’accès relatif, vérifiez que votre point de terminaison de test se termine par une barre oblique (/). Cela permet de s’assurer que le fichier CSS est chargé correctement.
> * Si vous visualisez votre API à partir d’un navigateur et que celui-ci vous demande d’entrer des informations d’identification de connexion pour afficher la page, utilisez [URL Decode](https://www.urldecoder.org/) pour décoder votre point de terminaison de test. URL Decode retourne une URL au format « https://\<username>:\<password>@\<cluster-name>.test.azureapps.io/\<app-name>/\<deployment-name> ».  Utilisez-le pour accéder à votre point de terminaison.

## <a name="add-actuator-dependency"></a>Ajouter une dépendance d’actionneur

Pour ajouter l’actionneur à un projet basé sur Maven, ajoutez la dépendance « Starter » :

```xml
<dependencies>
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-actuator</artifactId>
    </dependency>
</dependencies>
```

Compilez le nouveau binaire et déployez-le sur votre application.

## <a name="enable-production-ready-features"></a>Activer les fonctionnalités prêtes pour la production
Les points de terminaison d’actionneur vous permettent de surveiller votre application et d’interagir avec elle. Par défaut, l’application Spring Boot expose les points de terminaison `health` et `info` pour afficher des informations arbitraires sur l’application et son intégrité.

Pour observer la configuration et l’environnement configurable, nous devons également activer les points de terminaison `env` et `configgrops`.

1. Accédez au volet **Vue d’ensemble** de l’application, cliquez sur **Configuration** dans le menu des paramètres, puis accédez à la page de configuration **Variables d’environnement**.
1. Ajoutez les propriétés suivantes comme dans le formulaire « key:value ». Cet environnement ouvre le point de terminaison « env », « health » et « info » de l’actionneur Spring Boot.

   ```
   management.endpoints.web.exposure.include: env,health,info
   ```
1. Cliquez sur le bouton **Enregistrer**, votre application redémarrera automatiquement et chargera les nouvelles variables d’environnement.

Vous pouvez maintenant revenir au volet Vue d’ensemble de l’application et patienter jusqu’à ce que l’état d’approvisionnement passe à « Réussi ».  Il y aura plusieurs instances en cours d’exécution.

> [!Note] 
> Une fois que vous exposez l’application au public, ces points de terminaison d’actionneur sont également exposés au public. Vous pouvez masquer tous les points de terminaison en supprimant les variables d’environnement `management.endpoints.web.exposure.include` et en définissant `management.endpoints.web.exposure.exclude=*`.

## <a name="view-the-actuator-endpoint-to-view-application-information"></a>Afficher le point de terminaison de l’actionneur pour afficher les informations de l’application
1. Vous pouvez maintenant accéder à l’URL `"<test-endpoint>/actuator/"` pour voir tous les points de terminaison exposés par l’actionneur Spring Boot.
1. Accédez à l’URL `"<test-endpoint>/actuator/env"`, vous pouvez voir les profils actifs utilisés par l’application et toutes les variables d’environnement chargées.
1. Si vous souhaitez effectuer une recherche dans un environnement spécifique, vous pouvez accéder à l’URL `"<test-endpoint>/actuator/env/{toMatch}"` pour l’afficher.

Pour afficher tous les points de terminaison intégrés, consultez [Exposition des points de terminaison](https://docs.spring.io/spring-boot/docs/current/reference/html/production-ready-features.html#production-ready-endpoints-exposing-endpoints).

## <a name="next-steps"></a>Étapes suivantes

* [Comprendre les métriques pour Azure Spring Cloud](spring-cloud-concept-metrics.md)
* [Présentation de l’état des applications dans Azure Spring Cloud](spring-cloud-concept-app-status.md)


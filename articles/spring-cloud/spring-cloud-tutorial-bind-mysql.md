---
title: Lier une instance Azure Database pour MySQL à votre application Azure Spring Cloud
description: Découvrez comment lier une instance Azure Database pour MySQL à votre application Azure Spring Cloud
author: bmitchell287
ms.service: spring-cloud
ms.topic: how-to
ms.date: 11/04/2019
ms.author: brendm
ms.custom: devx-track-java
ms.openlocfilehash: 5ca8f2b003b7f2142da329a07f929ecf31fee627
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90908269"
---
# <a name="bind-an-azure-database-for-mysql-instance-to-your-azure-spring-cloud-application"></a>Lier une instance Azure Database pour MySQL à votre application Azure Spring Cloud 

**Cet article s’applique à :** ✔️ Java

Azure Spring Cloud vous permet de lier automatiquement certains services Azure à vos applications au lieu de configurer manuellement votre application Spring Boot. Cet article vous montre comment lier votre application à votre instance Azure Database pour MySQL.

## <a name="prerequisites"></a>Prérequis

* Une instance Azure Spring Cloud déployée
* Un compte Azure Database pour MySQL
* Azure CLI

Si vous n’avez pas déployé d’instance Azure Spring Cloud, suivez les instructions du [Démarrage rapide : Lancer une application Azure Spring Cloud à l’aide du portail Azure](spring-cloud-quickstart.md) pour déployer votre première application Cloud Spring.

## <a name="bind-your-app-to-your-azure-database-for-mysql-instance"></a>Lier votre application à votre instance Azure Database pour MySQL

1. Notez le nom d’utilisateur et le mot de passe de l’administrateur de votre compte Azure Database pour MySQL. 

1. Connectez-vous au serveur, créez une base de données nommée **testdb** à partir d’un client MySQL, puis créez un nouveau compte non-administrateur.

1. Dans le fichier *pom.xml* de votre projet, ajoutez la dépendance suivante :

    ```xml
    <dependency>
        <groupId>org.springframework.boot</groupId>
        <artifactId>spring-boot-starter-data-jpa</artifactId>
    </dependency>
    ```
1. Dans le fichier *application.properties*, supprimez toutes les propriétés `spring.datasource.*`.

1. Mettez à jour le déploiement actuel en exécutant `az spring-cloud app update` ou créez un déploiement pour ce changement en exécutant `az spring-cloud app deployment create`.  Ces commandes mettent à jour ou créent l’application avec la nouvelle dépendance.

1. Dans le portail Azure, dans la page de votre service **Azure Spring Cloud**, recherchez le **Tableau de bord d’application**, puis sélectionnez l’application à lier à votre instance Azure Database pour MySQL.  Il s’agit de la même application que celle que vous avez mise à jour ou déployée à l’étape précédente. 

1. Sélectionnez **Liaison de service**, puis sélectionnez le bouton **Créer une liaison de service**. 

1. Remplissez le formulaire en sélectionnant **Azure MySQL** comme **Type de liaison** et en utilisant le même nom de base de données que celui que vous avez utilisé précédemment, et les mêmes nom d’utilisateur et mot de passe que ceux que vous avez notés à la première étape.

1. Redémarrez l’application. La liaison doit maintenant fonctionner.

1. Pour vérifier que la liaison de service est correcte, sélectionnez le nom de liaison et examinez ses détails. Le champ `property` doit se présenter comme ceci :
    ```
    spring.datasource.url=jdbc:mysql://some-server.mysql.database.azure.com:3306/testdb?useSSL=true&requireSSL=false&useLegacyDatetimeCode=false&serverTimezone=UTC
    spring.datasource.username=admin@some-server
    spring.datasource.password=abc******
    spring.jpa.properties.hibernate.dialect=org.hibernate.dialect.MySQL5InnoDBDialect
    ```

## <a name="next-steps"></a>Étapes suivantes

Dans cet article, vous avez appris à lier une application Azure Spring Cloud à une instance Azure Database pour MySQL. Pour en savoir plus sur la liaison de services à une application, consultez [Lier une base de données Azure Cosmos DB à une application Azure Spring Cloud](spring-cloud-tutorial-bind-cosmos.md).

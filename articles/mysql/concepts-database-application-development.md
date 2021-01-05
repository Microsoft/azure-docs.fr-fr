---
title: Développement d’applications - Azure Database pour MySQL
description: Présente les considérations relatives à la conception que les développeurs doivent suivre pour écrire du code d’application permettant de se connecter à la base de données Azure pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: conceptual
ms.date: 3/18/2020
ms.openlocfilehash: 20c6841bb596c7f4a32ef689423515a8da59a945
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541553"
---
# <a name="application-development-overview-for-azure-database-for-mysql"></a>Vue d’ensemble du développement d’applications pour la base de données Azure pour MySQL 
Cet article aborde les considérations relatives à la conception que les développeurs doivent suivre pour écrire du code d’application permettant de se connecter à Azure Database pour MySQL. 

> [!TIP]
> Vous trouverez un didacticiel pour apprendre à créer un serveur, créer un pare-feu sur le serveur, afficher les propriétés du serveur, créer une base de données, se connecter et effectuer des requêtes à l’aide de workbench et de mysql.exe à la page [Concevoir une première base de données Azure Database pour MySQL](tutorial-design-database-using-portal.md).

## <a name="language-and-platform"></a>Langage et plateforme
Plusieurs exemples de code sont à votre disposition pour divers langages et plateformes de programmation. Vous trouverez des liens vers des exemples de code à la page [Bibliothèques de connectivité utilisées pour se connecter à la base de données Azure pour MySQL](concepts-connection-libraries.md).

## <a name="tools"></a>Outils
La base de données Azure pour MySQL utilise la version Community de MySQL, compatible avec les outils de gestion MySQL courants, notamment les utilitaires Workbench et MySQL, par exemple mysql.exe, [phpMyAdmin](https://www.phpmyadmin.net/) ou [Navicat](https://www.navicat.com/products/navicat-for-mysql). Vous pouvez également utiliser le Portail Azure, Azure CLI et les API REST pour interagir avec le service de base de données.

## <a name="resource-limitations"></a>Limitations des ressources
Azure Database pour MySQL gère les ressources accessibles à un serveur selon deux mécanismes différents : 
- gouvernance des ressources ;
- application des limites.

## <a name="security"></a>Sécurité
Azure Database pour MySQL fournit des ressources permettant de limiter l’accès, de protéger les données, de configurer les utilisateurs et les rôles et de surveiller les activités sur une base de données MySQL.

## <a name="authentication"></a>Authentication
Azure Database pour MySQL prend en charge l’authentification serveur des utilisateurs et des connexions.

## <a name="resiliency"></a>Résilience
Si une erreur temporaire se produit au cours de la connexion à une base de données MySQL, votre code doit effectuer une nouvelle tentative d’appel. Nous vous recommandons d’utiliser une logique de nouvelle tentative basée sur une logique d’interruption, afin d’éviter que la base de données SQL ne soit inondée de tentatives simultanées de plusieurs clients.

- Exemples de code : vous trouverez des exemples de code qui illustrent la logique de nouvelle tentative dans le langage de votre choix à la page [Bibliothèques de connectivité utilisées pour se connecter à la base de données Azure pour MySQL](concepts-connection-libraries.md).

## <a name="managing-connections"></a>Gestion des connexions
Les connexions de base de données étant une ressource limitée, nous vous recommandons d’en faire un usage raisonnable lorsque vous accédez à votre base de données MySQL, afin d’améliorer les performances.
- Accédez à la base de données en utilisant le regroupement de connexions ou les connexions persistantes.
- Accédez à la base de données sur une courte durée de connexion. 
- Utilisez la logique de nouvelle tentative dans votre application au moment de la tentative de connexion, de façon à intercepter les défaillances dues à des connexions simultanées qui ont atteint le nombre maximal autorisé. Dans cette logique, définissez un délai court et attendez pendant une durée aléatoire avant les autres tentatives de connexion.
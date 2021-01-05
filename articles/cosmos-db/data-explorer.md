---
title: Utiliser l’explorateur Azure Cosmos DB pour gérer vos données
description: L’explorateur Azure Cosmos DB est une interface web autonome qui vous permet de voir et de gérer les données stockées dans Azure Cosmos DB.
author: deborahc
ms.service: cosmos-db
ms.topic: how-to
ms.date: 09/23/2020
ms.author: dech
ms.openlocfilehash: d4baa92fe4aa2ed402c394198684c4deec2bf9f1
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96348600"
---
# <a name="work-with-data-using-azure-cosmos-db-explorer"></a>Utiliser des données à l’aide de l’explorateur Azure Cosmos DB 
[!INCLUDE[appliesto-all-apis](includes/appliesto-all-apis.md)]

L’explorateur Azure Cosmos DB est une interface web autonome qui vous permet de voir et de gérer les données stockées dans Azure Cosmos DB. L’explorateur Azure Cosmos DB est l’équivalent de l’onglet **Explorateur de données** qui se trouve dans le portail Azure lorsque vous créez un compte Azure Cosmos DB. Les principaux avantages de l’explorateur Azure Cosmos DB par rapport à l’Explorateur de données existant sont les suivants :

* Vous disposez d’une véritable vue plein écran pour consulter vos données, exécuter des requêtes, des procédures stockées, des déclencheurs et voir leurs résultats.  

* Vous pouvez fournir un accès temporaire ou permanent, en lecture-écriture ou en lecture seule, de votre compte de base de données et de ses collections, à d’autres utilisateurs qui n’ont pas accès à l’abonnement ou au portail Azure.  

* Vous pouvez partager les résultats de requête avec d’autres utilisateurs qui n’ont pas accès à l’abonnement ou au portail Azure.  

## <a name="access-azure-cosmos-db-explorer"></a>Accéder à l’explorateur Azure Cosmos DB

1. Connectez-vous au [portail Azure](https://portal.azure.com/). 

2. À partir de **Toutes les ressources**, recherchez et accédez à votre compte Azure Cosmos DB, sélectionnez Clés et copiez la **Chaîne de connexion principale**.  

3. Accédez à https://cosmos.azure.com/, collez la chaîne de connexion, puis sélectionnez **Connexion**. À l’aide de la chaîne de connexion, vous pouvez accéder à l’explorateur Azure Cosmos DB sans aucune limitation de durée.  

   Si vous souhaitez fournir à d’autres utilisateurs un accès temporaire à votre compte Azure Cosmos DB, vous pouvez le faire par le biais d’URL d’accès en lecture-écriture et en lecture. 

4. Ouvrez le panneau **Explorateur de données** et sélectionnez **Ouvrir en mode plein écran**. Dans la boîte de dialogue contextuelle, vous pouvez voir deux URL d’accès : **Lecture-écriture** et **Lecture**. Ces URL vous permettent de partager temporairement votre compte Azure Cosmos DB avec d’autres utilisateurs. L’accès au compte expire au bout de 24 heures, après quoi vous pouvez vous reconnecter au moyen d’une nouvelle URL d’accès, ou de la chaîne de connexion. 

   **Lecture-écriture** : lorsque vous partagez l’URL en lecture-écriture avec d’autres utilisateurs, ceux-ci peuvent voir et modifier les bases de données, collections, requêtes et d’autres ressources associées à ce compte particulier.

   **Lecture** : lorsque vous partagez l’URL en lecture seule avec d’autres utilisateurs, ceux-ci peuvent afficher les bases de données, collections, requêtes et d’autres ressources associées à ce compte particulier. Par exemple, si vous souhaitez partager les résultats d’une requête avec vos collègues qui n’ont pas accès au portail Azure ou à votre compte Azure Cosmos DB, vous pouvez leur fournir cette URL.

   Choisissez le type d’accès avec lequel vous souhaitez ouvrir le compte et cliquez sur **Ouvrir**. Une fois l’Explorateur ouvert, l’expérience est identique à celle que vous aviez avec l’onglet Explorateur de données dans le portail Azure.

   :::image type="content" source="./media/data-explorer/open-data-explorer-with-access-url.png" alt-text="Ouvrir l’explorateur Azure Cosmos DB":::

## <a name="known-issues"></a>Problèmes connus

Pour le moment, l’expérience **Ouvrir en mode plein écran** qui vous permet de partager un accès temporaire en lecture-écriture ou en lecture seule n’est pas encore prise en charge pour les comptes Azure Cosmos DB Gremlin et API Table. Vous pouvez toujours afficher vos comptes Gremlin et API Table en passant la chaîne de connexion à l’explorateur Azure Cosmos DB. 

Actuellement, l’affichage des documents qui contiennent un UUID n’est pas pris en charge dans l’Explorateur de données. Cela n’affecte pas le chargement des collections, mais uniquement l’affichage des documents individuels ou des requêtes qui incluent ces documents. Pour afficher et gérer ces documents, les utilisateurs doivent continuer à utiliser l’outil initialement utilisé pour créer ces documents.

Si les clients reçoivent des erreurs HTTP-401, cela peut être dû à des autorisations Azure RBAC insuffisantes pour le compte Azure du client, en particulier si le compte a un rôle personnalisé. Tous les rôles personnalisés doivent avoir l’action `Microsoft.DocumentDB/databaseAccounts/listKeys/*` pour utiliser Explorateur de données s’ils se connectent en utilisant leurs informations d’identification Azure Active Directory.

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez découvert comment démarrer avec l’explorateur Azure Cosmos DB pour gérer vos données, vous pouvez :

* Commencer à définir des [requêtes](./sql-query-getting-started.md) à l’aide de la syntaxe SQL, et faire de la [programmation côté serveur](stored-procedures-triggers-udfs.md) en vous servant de procédures stockées, d’UDF et de déclencheurs.

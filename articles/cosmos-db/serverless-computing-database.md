---
title: Traitement de base de données serverless avec Azure Cosmos DB et Azure Functions
description: Découvrez comment Azure Cosmos DB et Azure Functions peuvent être utilisés ensemble pour créer des applications informatiques sans serveur basées sur les événements.
author: SnehaGunda
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 07/17/2019
ms.author: sngun
ms.openlocfilehash: 73a34cc27eaba33d04f4d31585c7f494f58e7274
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93334070"
---
# <a name="serverless-database-computing-using-azure-cosmos-db-and-azure-functions"></a>Traitement de base de données serverless à l’aide d’Azure Cosmos DB et d’Azure Functions
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

L’informatique sans serveur permet de se concentrer sur des éléments de logique individuels renouvelables et sans état. Ces éléments ne nécessitent aucune gestion d’infrastructure et consomment uniquement des ressources le temps de leur exécution (secondes ou millisecondes). Au cœur du mouvement informatique sans serveur se trouvent des fonctions, qui sont mises à disposition dans l’écosystème Azure par [Azure Functions](https://azure.microsoft.com/services/functions). Pour en savoir plus sur les autres environnements d’exécution serverless dans Azure, consultez la page [Le Serverless dans Azure](https://azure.microsoft.com/solutions/serverless/). 

Grâce à l’intégration native entre [Azure Cosmos DB](https://azure.microsoft.com/services/cosmos-db) et Azure Functions, vous pouvez créer des déclencheurs de base de données, des liaisons d’entrée et des liaisons de sortie directement à partir de votre compte Azure Cosmos DB. À l’aide d’Azure Functions et d’Azure Cosmos DB, vous pouvez créer et déployer des applications sans serveur basées sur les événements avec un accès à faible latence aux données enrichies pour une base d’utilisateurs globale.

## <a name="overview"></a>Vue d’ensemble

Azure Cosmos DB et Azure Functions permettent d’intégrer vos bases de données et applications sans serveur comme suit :

* Créez un **déclencheur Azure Functions pour Cosmos DB** basé sur les événements. Ce déclencheur se base sur les [flux de modification](change-feed.md) pour effectuer le monitoring des modifications du conteneur Azure Cosmos. Quand des modifications sont apportées à un conteneur, le flux de modification est envoyé au déclencheur, qui appelle la fonction Azure.
* Vous pouvez également lier une fonction Azure à un conteneur Azure Cosmos à l'aide d'une **liaison d'entrée**. Les liaisons d’entrée lisent les données d’un conteneur lorsqu’une fonction s’exécute.
* Liez une fonction à un conteneur Azure Cosmos à l'aide d'une **liaison de sortie**. Les liaisons de sortie écrivent des données dans un conteneur après l’exécution d’une fonction.

> [!NOTE]
> Actuellement, les liaisons d’entrée, les liaisons de sortie et le déclencheur Azure Functions pour Cosmos DB sont uniquement pris en charge pour une utilisation avec l’API SQL. Pour toutes les autres API Azure Cosmos DB, vous devez accéder à la base de données depuis votre fonction en utilisant le client statique pour votre API.


Le diagramme suivant illustre chacune de ces trois intégrations : 

:::image type="content" source="./media/serverless-computing-database/cosmos-db-azure-functions-integration.png" alt-text="Mode d’intégration d’Azure Cosmos DB et d’Azure Functions" border="false":::

Le déclencheur Azure Functions, la liaison d’entrée et la liaison de sortie Azure Cosmos DB peuvent être utilisés dans les combinaisons suivantes :

* Un déclencheur Azure Functions pour Cosmos DB peut être utilisé avec une liaison de sortie vers un autre conteneur Azure Cosmos. Une fois qu’une fonction a exécuté une action sur un élément dans le flux de modification, vous pouvez l’écrire dans un autre conteneur (son écriture dans le même conteneur d’origine créerait de fait une boucle récursive). Vous pouvez également utiliser un déclencheur Azure Functions pour Cosmos DB afin de migrer efficacement tous les éléments modifiés d’un conteneur vers un autre à l’aide d’une liaison de sortie.
* Les liaisons d’entrée et les liaisons de sortie pour Azure Cosmos DB peuvent être utilisées dans la même fonction Azure. Cela est utile quand vous souhaitez trouver certaines données avec la liaison d’entrée, les modifier dans la fonction Azure, puis les enregistrer dans le même conteneur ou dans un autre conteneur après la modification.
* Une liaison d'entrée vers un conteneur Azure Cosmos peut être utilisée dans la même fonction qu'un déclencheur Azure Functions pour Cosmos DB et peut également être utilisée avec ou sans liaison de sortie. Vous pouvez utiliser cette combinaison pour appliquer des informations de change à jour (extraites à l’aide d’une liaison d’entrée vers un conteneur de change) pour le flux de modification de nouvelles commandes dans votre service de panier. Le total du panier mis à jour, avec la conversion monétaire actuelle appliquée, peut être écrit dans un troisième conteneur à l’aide d’une liaison de sortie.

## <a name="use-cases"></a>Cas d'utilisation

Les cas d’usage suivants montrent plusieurs manières de tirer le meilleur parti de vos données Azure Cosmos DB, en connectant vos données à des fonctions Azure basées sur les événements.

### <a name="iot-use-case---azure-functions-trigger-and-output-binding-for-cosmos-db"></a>Cas d’usage IoT - Déclencheur Azure Functions et liaison de sortie pour Cosmos DB

Dans les implémentations IoT, vous pouvez appeler une fonction quand le voyant de vérification du moteur est allumé dans une voiture connectée.

**Implémentation :** Utiliser un déclencheur Azure Functions et une liaison de sortie pour Cosmos DB

1. Un **déclencheur Azure Functions pour Cosmos DB** est utilisé pour déclencher des événements liés à des alertes de voiture, tels que le voyant de vérification du moteur qui s’allume dans une voiture connectée.
2. Quand le voyant de vérification du moteur s’allume, les données de capteur sont envoyées à Azure Cosmos DB.
3. Azure Cosmos DB crée ou met à jour de nouveaux documents de données de capteur. Ces modifications sont ensuite transmises au déclencheur Azure Functions pour Cosmos DB.
4. Le déclencheur est appelé à chaque modification de données de la collection de données de capteur, étant donné que toutes les modifications sont transmises via le flux de modification.
5. Une condition de seuil est utilisée dans la fonction pour envoyer les données de capteur au service de garanties.
6. Si la température dépasse une certaine valeur, une alerte est également envoyée au propriétaire.
7. La **liaison de sortie** sur la fonction met à jour l'enregistrement de la voiture dans un autre conteneur Azure Cosmos pour stocker les informations relatives à l'événement de vérification du moteur.

L’illustration suivante montre le code écrit dans le portail Azure pour ce déclencheur.

:::image type="content" source="./media/serverless-computing-database/cosmos-db-trigger-portal.png" alt-text="Créer un déclencheur Azure Functions pour Cosmos DB dans le portail Azure":::

### <a name="financial-use-case---timer-trigger-and-input-binding"></a>Cas d’usage financier - Déclencheur de minuteur et liaison d’entrée

Dans les implémentations financières, vous pouvez appeler une fonction lorsqu’un solde de compte bancaire est inférieur à une certaine quantité.

**Implémentation :** Déclencheur de minuteur avec une liaison d’entrée Azure Cosmos DB

1. À l'aide d'un [déclencheur de minuteur](../azure-functions/functions-bindings-timer.md), vous pouvez récupérer les informations relatives aux soldes de comptes bancaires stockées dans un conteneur Azure Cosmos à intervalles réguliers à l'aide d'une **liaison d'entrée**.
2. Si le solde est inférieur au seuil de solde faible défini par l’utilisateur, effectuez un suivi à l’aide d’une action à partir de la fonction Azure.
3. La liaison de sortie peut être une [intégration SendGrid](../azure-functions/functions-bindings-sendgrid.md) qui envoie un e-mail à partir d’un compte de service aux adresses e-mail identifiées pour chacun des comptes dont le solde est faible.

Les illustrations suivantes montrent le code dans le portail Azure pour ce scénario.

:::image type="content" source="./media/serverless-computing-database/cosmos-db-functions-financial-trigger.png" alt-text="Fichier index.js pour un déclencheur de minuteur dans un scénario financier":::

:::image type="content" source="./media/serverless-computing-database/azure-function-cosmos-db-trigger-run.png" alt-text="Fichier run.csx pour un déclencheur de minuteur dans un scénario financier":::

### <a name="gaming-use-case---azure-functions-trigger-and-output-binding-for-cosmos-db"></a>Cas d’usage de jeu - Déclencheur Azure Functions et liaison de sortie pour Cosmos DB 

En matière de gaming, quand un utilisateur est créé, vous pouvez rechercher d’autres utilisateurs que vous connaissez peut-être à l’aide de l’[API Gremlin Azure Cosmos DB](graph-introduction.md). Vous pouvez ensuite écrire les résultats dans une base de données SQL Azure Cosmos DB pour faciliter leur récupération.

**Implémentation :** Utiliser un déclencheur Azure Functions et une liaison de sortie pour Cosmos DB

1. À l’aide d’une [base de données de graphes](graph-introduction.md) Azure Cosmos DB pour stocker tous les utilisateurs, vous pouvez créer une nouvelle fonction avec un déclencheur Azure Functions pour Cosmos DB. 
2. Chaque fois qu’un nouvel utilisateur est inséré, la fonction est appelée, puis le résultat est stocké à l’aide d’une **liaison de sortie**.
3. La fonction interroge la base de données des graphes pour rechercher tous les utilisateurs qui sont directement associés au nouvel utilisateur et retourne un jeu de données à la fonction.
4. Ces données sont ensuite stockées dans une base de données Azure Cosmos DB, qui peut être facilement récupérée par la suite par toute application frontale qui montre au nouvel utilisateur ses amis connectés.

### <a name="retail-use-case---multiple-functions"></a>Cas d’usage de vente au détail - Fonctions multiples

Dans les implémentations de vente au détail, lorsqu’un utilisateur ajoute un élément à son panier, vous pouvez désormais créer et appeler des fonctions pour des composants de pipeline commercial facultatifs.

**Implémentation :** Plusieurs déclencheurs Azure Functions pour Cosmos DB écoutant un conteneur

1. Vous pouvez créer plusieurs fonctions Azure en ajoutant à chacune d’elles des déclencheurs Azure Functions pour Cosmos DB, qui écoutent tous le même flux de modifications des données des paniers. Notez que lorsque plusieurs fonctions écoutent le même flux de modification, une nouvelle collection de baux est requise pour chaque fonction. Pour plus d’informations sur les collections de baux, consultez [Présentation de la bibliothèque du processeur de flux de modification](change-feed-processor.md).
2. Lorsqu’un utilisateur ajoute un nouvel élément à son panier, chaque fonction est appelée indépendamment par le flux de modification à partir du conteneur de panier.
   * Une fonction peut utiliser le contenu du panier actuel pour modifier l’affichage d’autres éléments qui pourraient intéresser l’utilisateur.
   * Une autre fonction peut mettre à jour les totaux des inventaires.
   * Une autre fonction peut envoyer des informations client pour certains produits au service marketing, qui leur envoie ensuite un publipostage promotionnel. 

     Tout service peut créer un déclencheur Azure Functions pour Cosmos DB en écoutant le flux de modifications et s’assurer ainsi de ne retarder aucun événement de traitement des commandes critique dans le processus.

Dans tous ces cas d’usage, étant donné que la fonction a découplé l’application proprement dite, vous n’avez pas besoin de lancer de nouvelles instances de l’application à tout moment. À la place, Azure Functions lance des fonctions individuelles pour terminer les processus discrets selon les besoins.

## <a name="tooling"></a>Outils

L’intégration native entre Azure Cosmos DB et Azure Functions est disponible dans le portail Azure et dans Visual Studio 2019.

* Dans le portail Azure Functions, vous pouvez créer un déclencheur. Pour obtenir des instructions de démarrage rapide, consultez [Créer un déclencheur Azure Functions pour Cosmos DB dans le portail Azure](../azure-functions/functions-create-cosmos-db-triggered-function.md).
* Dans le portail Azure Cosmos DB, vous pouvez ajouter un déclencheur Azure Functions pour Cosmos DB à une application Azure Functions existante dans le même groupe de ressources.
* Dans Visual Studio 2019, vous pouvez créer le déclencheur en utilisant [Azure Functions Tools](../azure-functions/functions-develop-vs.md) :

    >[!VIDEO https://www.youtube.com/embed/iprndNsUeeg]

## <a name="why-choose-azure-functions-integration-for-serverless-computing"></a>Pourquoi choisir l’intégration d’Azure Functions pour l’informatique sans serveur ?

Azure Functions permet de créer des unités évolutives de travail ou des éléments de logique concis qui peuvent être exécutés à la demande, sans avoir à configurer ni gérer d’infrastructure. L'utilisation d'Azure Functions évite d'avoir à créer une application complète pour répondre aux modifications de la base de données Azure Cosmos. À la place, vous pouvez créer de petites fonctions réutilisables pour des tâches spécifiques. Vous pouvez également utiliser les données Azure Cosmos DB comme entrée ou sortie vers une fonction Azure en réponse à des événements tels que des requêtes HTTP ou un déclencheur régulier.

Azure Cosmos DB est la base de données recommandée pour l’architecture informatique sans serveur pour les raisons suivantes :

* **Accès instantané à toutes vos données** : vous disposez d’un accès granulaire à toutes les valeurs stockées, car Azure Cosmos DB [indexe automatiquement](index-policy.md) toutes les données par défaut et met immédiatement ces index à disposition. Cela signifie que vous pouvez en permanence interroger, mettre à jour et ajouter de nouveaux éléments à la base de données et disposer d’un accès instantané via Azure Functions.

* **Sans schéma** : Azure Cosmos DB est une base de données sans schéma, ce qui lui permet en particulier de gérer toute sortie de données à partir d’une fonction Azure. Cette approche de « gestion totale » simplifie la création de toute une variété de fonctions qui ont toutes une sortie vers Azure Cosmos DB.

* **Débit évolutif** : il est possible de monter ou descendre en puissance le débit de manière instantanée dans Azure Cosmos DB. Si vous avez des centaines ou des milliers de fonctions interrogeant et écrivant dans le même conteneur, vous pouvez monter en puissance votre [RU/s](request-units.md) pour gérer la charge. Toutes les fonctions peuvent fonctionner en parallèle à l’aide de votre RU/s alloué, et la [cohérence](consistency-levels.md) de vos données est garantie.

* **Réplication globale** : vous pouvez répliquer des données Azure Cosmos DB [dans le monde entier](distribute-data-globally.md) pour réduire la latence, en géolocalisant vos données le plus proche possible de vos utilisateurs. Comme avec toutes les requêtes Azure Cosmos DB, les données provenant de déclencheurs basés sur les événements sont lues à partir de la base de données Azure Cosmos DB la plus proche de l’utilisateur.

Si vous envisagez une intégration avec Azure Functions pour le stockage de données et que vous n’ayez pas besoin d’effectuer d’indexation complète ou de stocker de pièces jointes ou fichiers multimédias, le [déclencheur Stockage Blob Azure](../azure-functions/functions-bindings-storage-blob.md) peut être une meilleure option.

Avantages d’Azure Functions : 

* **Basé sur les événements** : les fonctions Azure sont basées sur les événements et peuvent écouter un flux de modification à partir d’Azure Cosmos DB. Cela signifie que vous n’avez pas besoin de créer une logique d’écoute. Il vous suffit simplement de surveiller les modifications que vous écoutez. 

* **Sans limite** : les fonctions s’exécutent en parallèle et le service lance celles dont vous avez besoin. Vous définissez les paramètres.

* **Idéal pour les tâches rapides** : le service lance de nouvelles instances de fonctions chaque fois qu’un événement se déclenche, puis les ferme dès que la fonction a été exécutée. Vous payez uniquement pour la durée d’exécution de vos fonctions.

Si vous n’êtes pas sûr de la meilleure option entre Flow, Logic Apps, Azure Functions et WebJobs pour votre implémentation, consultez [Choisir entre Flow, Logic Apps, Functions et WebJobs](../azure-functions/functions-compare-logic-apps-ms-flow-webjobs.md).

## <a name="next-steps"></a>Étapes suivantes

À présent, connectons réellement Azure Cosmos DB et Azure Functions : 

* [Créer un déclencheur Azure Functions pour Cosmos DB dans le portail Azure](../azure-functions/functions-create-cosmos-db-triggered-function.md)
* [Créer un déclencheur HTTP Azure Functions avec une liaison d’entrée Azure Cosmos DB](../azure-functions/functions-bindings-cosmosdb.md?tabs=csharp)
* [Liaisons et déclencheurs Azure Cosmos DB](../azure-functions/functions-bindings-cosmosdb-v2.md)

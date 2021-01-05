---
title: 'Démarrage rapide : Se connecter à Azure Synapse Link pour Cosmos DB'
description: Comment connecter une base de données Azure Cosmos DB à un espace de travail Synapse avec Synapse Link
services: synapse-analytics
author: ArnoMicrosoft
ms.service: synapse-analytics
ms.subservice: synapse-link
ms.topic: quickstart
ms.date: 04/21/2020
ms.author: acomet
ms.reviewer: jrasnick
ms.openlocfilehash: 14c71d5888aefc11485c03935c5d8c7ff7d17045
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95995877"
---
# <a name="quickstart-connect-to-azure-synapse-link-for-azure-cosmos-db"></a>Démarrage rapide : Se connecter à Azure Synapse Link pour Cosmos DB

Cet article explique comment accéder à une base de données Azure Cosmos DB à partir d’Azure Synapse Analytics Studio avec Synapse Link. 

## <a name="prerequisites"></a>Prérequis

Avant de connecter un compte Azure Cosmos DB à votre espace de travail, vous avez besoin de quelques éléments.

* Compte Azure Cosmos DB existant ou créer un compte après ce [démarrage rapide](https://docs.microsoft.com/azure/cosmos-db/how-to-manage-database-account)
* Espace de travail Synapse existant ou créer un espace de travail après ce [démarrage rapide](https://docs.microsoft.com/azure/synapse-analytics/quickstart-create-workspace) 

## <a name="enable-azure-cosmos-db-analytical-store"></a>Vous avez activé le magasin analytique Azure Cosmos DB

Pour exécuter l’analytique à grande échelle dans Azure Cosmos DB sans affecter les performances opérationnelles, nous vous recommandons d’activer Synapse Link pour Azure Cosmos DB. Cette fonction apporte la fonctionnalité HTAP à un conteneur et une prise en charge intégrée dans Azure Synapse. Suivez ce guide de démarrage rapide pour activer les conteneurs Synapse Link pour Cosmos DB.

## <a name="navigate-to-synapse-studio"></a>Accéder à Synapse Studio

Dans votre espace de travail Synapse, sélectionnez **Lancer Synapse Studio**. Sur la page d’accueil de Synapse Studio, sélectionnez **Data (Données), ce qui vous permet de vous connecter à **Data Object Explorer** (Explorateur d'objets de données).

## <a name="connect-an-azure-cosmos-db-database-to-a-synapse-workspace"></a>Connecter une base de données Azure Cosmos DB à un espace de travail Synapse

La connexion d’une base de données Azure Cosmos DB s’effectue en tant que service lié. Un service lié Cosmos DB permet aux utilisateurs de parcourir et d’explorer les données, de les lire et de les écrire à partir d’Apache Spark pour Azure Synapse Analytics ou SQL dans Azure Cosmos DB.

À partir de l’Explorateur d’objets de données, vous pouvez vous connecter directement à une base de données Azure Cosmos DB en procédant comme suit :

1. Sélectionnez l’icône **_+_* _ à proximité de Données
2. Sélectionnez _ *Se connecter à des données externes**
3. Sélectionnez l’API à laquelle vous souhaitez vous connecter : SQL ou MongoDB
4. Sélectionnez **_Continuer_* _
5. Nommez le service lié. Le nom sera affiché dans l’Explorateur d’objets et utilisé par les exécutables Synapse pour se connecter à la base de données et aux conteneurs. Nous vous recommandons d’utiliser un nom convivial.
6. Sélectionnez le _ *nom du compte Cosmos DB** et le **nom de la base de données**
7. (Facultatif) Si aucune région n’est spécifiée, les opérations d’exécution Synapse sont acheminées vers la région la plus proche dans laquelle le magasin analytique est activé. Toutefois, vous pouvez définir manuellement la région dans laquelle vous souhaitez que vos utilisateurs accèdent au magasin analytique Cosmos DB. Sélectionnez **Propriétés de connexion supplémentaires** puis **Nouveau**. Sous **Nom de la propriété**, tapez **_PreferredRegions_ *_ et définissez _* Valeur** sur la région de votre choix (exemple : WestUS2, sans espace entre les mots et les nombres)
8. Sélectionnez **_Créer_* _

Les bases de données Azure Cosmos DB sont visibles sous l’onglet _ *Lié** dans la section Azure Cosmos DB. Vous pouvez différencier un conteneur Azure Cosmos DB activé par HTAP d’un conteneur OLTP uniquement en vous reportant aux icônes suivantes :

**Conteneur Synapse** :

![Conteneur HTAP](./media/quickstart-connect-synapse-link-cosmosdb/htap-container.png)

**Conteneur OLTP uniquement** :

![Conteneur OLTP](./media/quickstart-connect-synapse-link-cosmosdb/oltp-container.png)

## <a name="quickly-interact-with-code-generated-actions"></a>Interagir rapidement avec des actions générées par le code

Lorsque vous cliquez avec le bouton droit sur un conteneur, vous obtenez une liste des mouvements qui déclencheront une exécution Spark ou SQL. L’écriture de données dans un conteneur aura lieu via le magasin transactionnel d’Azure Cosmos DB et consommera des unités de requête.  

## <a name="next-steps"></a>Étapes suivantes

* [Découvrir ce qui est pris en charge entre Synapse et Azure Cosmos DB](./synapse-link/concept-synapse-link-cosmos-db-support.md)
* [Découvrir comment interroger un magasin analytique avec Apache Spark pour Azure Synapse Analytics](synapse-link/how-to-query-analytical-store-spark.md)
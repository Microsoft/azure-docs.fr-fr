---
title: Diffuser des données en continu à l’aide d’une intégration Azure Stream Analytics (préversion)
description: Utilisez l'intégration Azure Stream Analytics pour diffuser des données en continu dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: development
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: how-to
author: ajetasin
ms.author: ajetasi
ms.reviewer: sstein
ms.date: 11/04/2019
ms.openlocfilehash: 36a96a1927aeedb5f841083241d487e0c61d6813
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96454023"
---
# <a name="stream-data-into-azure-sql-database-using-azure-stream-analytics-integration-preview"></a>Diffuser des données en continu dans Azure SQL Database à l’aide d’une intégration Azure Stream Analytics (préversion)

Les utilisateurs peuvent désormais ingérer, traiter, afficher et analyser des données de diffusion en temps réel dans une table directement à partir d’une base de données dans Azure SQL Database. Ils le font dans le portail Azure à l’aide d’[Azure Stream Analytics](../../stream-analytics/stream-analytics-introduction.md). Cette expérience permet un large éventail de scénarios, tels que la voiture connectée, la supervision à distance, la détection des fraudes et bien plus encore. Dans le Portail Azure, vous pouvez sélectionner une source de l’événement (Event Hub/IoT Hub), afficher les événements entrants en temps réel et sélectionner une table pour stocker des événements. Vous pouvez également écrire des requêtes en langage de requêtes Azure Stream Analytics dans le portail pour transformer des événements entrants et les stocker dans la table sélectionnée. Ce nouveau point d’entrée s’ajoute aux expériences de création et de configuration qui existent déjà dans Stream Analytics. Cette expérience commence dans le contexte de votre base de données, ce qui vous permet de configurer rapidement un travail Stream Analytics et de naviguer sans difficulté entre la base de données dans Azure SQL Database et les expériences Stream Analytics.

![Flux de Stream Analytics](./media/stream-data-stream-analytics-integration/stream-analytics-flow.png)

## <a name="key-benefits"></a>Principaux avantages

- Changement de contexte minimal : Vous pouvez démarrer à partir d'une base de données Azure SQL Database sur le portail et commencer à ingérer des données en temps réel dans une table sans passer par un autre service.
- Nombre d'étapes réduit : Le contexte de votre base de données et de votre table est utilisé pour préconfigurer un travail de Stream Analytics.
- Facilité d’utilisation accrue avec les données de préversion : Données de préversion entrantes à partir de la source des événements (Event Hub/IoT Hub) dans le contexte de la table sélectionnée

> [!IMPORTANT]
> Un travail Azure Stream Analytics peut faire l’objet d’une sortie dans Azure SQL Database, Azure SQL Managed Instance ou Azure Synapse Analytics. Pour plus d’informations, consultez la section [Sorties](../../stream-analytics/stream-analytics-define-outputs.md).

## <a name="prerequisites"></a>Prérequis

Pour effectuer les étapes de cet article, vous avez besoin des ressources suivantes :

- Un abonnement Azure. Si vous n’avez pas d’abonnement Azure, [créez un compte gratuit](https://azure.microsoft.com/free/).
- Une base de données Azure SQL Database. Pour des instructions plus détaillées, consultez [Créer une base de données unique dans Azure SQL Database](single-database-create-quickstart.md).
- Une règle de pare-feu permettant à votre ordinateur de se connecter au serveur. Pour des instructions plus détaillées, consultez [Créer une règle de pare-feu au niveau du serveur](firewall-create-server-level-portal-quickstart.md).

## <a name="configure-stream-analytics-integration"></a>Configurer une intégration Stream Analytics

1. Connectez-vous au portail Azure.
2. Accédez à la base de données dans laquelle vous souhaitez ingérer vos données de diffusion en continu. Sélectionnez **Stream Analytics (préversion)** .

    ![Stream Analytics](./media/stream-data-stream-analytics-integration/stream-analytics.png)

3. Pour commencer à recevoir vos données de diffusion en continu dans cette base de données, sélectionnez **Créer** et donnez un nom à votre travail de diffusion en continu, puis sélectionnez **Suivant : Entrée**.

    ![Configurer des principes de base de travail Stream Analytics](./media/stream-data-stream-analytics-integration/create-job.png)

4. Entrez les détails de votre source de l'événement, puis cliquez sur **Suivant : Sortie**.

   - **Type d’entrée** : Event Hub/IoT Hub
   - **Alias d’entrée** : Entrez un nom pour identifier votre source des événements
   - **Abonnement**: Le même abonnement que pour Azure SQL Database
   - **Espace de noms Event Hub** : Nom de l’espace de noms
   - **Nom du hub d’événements** : Nom de l’Event Hub dans l’espace de noms sélectionné
   - **Nom de la stratégie Event Hub** (Par défaut, créer un nouveau nom) : Donner un nom à la stratégie
   - **Groupe de consommateurs Event Hub** (Par défaut, créer un nouveau nom) : Donner un nom au groupe de consommateurs  

      Nous vous recommandons de créer un groupe de consommateurs et une stratégie pour chaque nouveau travail Azure Stream Analytics que vous créez à partir d’ici. Les groupes de consommateurs autorisent uniquement cinq lecteurs simultanés et dès lors, fournir un groupe de consommateurs dédié pour chaque travail permet d’éviter les erreurs liées à un dépassement de cette limite. Une stratégie dédiée vous permet de faire pivoter votre clé ou de révoquer des autorisations sans affecter d’autres ressources.

     ![configurer la sortie du travail Stream Analytics](./media/stream-data-stream-analytics-integration/create-job-output.png)

5. Sélectionnez la table dans laquelle vous souhaitez ingérer vos données de diffusion en continu. Une fois terminé, sélectionnez **Créer**.

   - **Nom d’utilisateur**, **Mot de passe** : Entrez vos informations d’identification pour l’authentification du serveur SQL. Sélectionnez **Valider**.
   - **Table**: Sélectionnez **Créer un nouveau** ou **Utiliser un existant**. Dans ce flux, nous allons sélectionner **Créer**. Une nouvelle table est créée lorsque vous démarrez le travail Stream Analytics.

     ![créer un travail Stream Analytics](./media/stream-data-stream-analytics-integration/create.png)

6. Une page de requête s’ouvre avec les détails suivants :

   - Votre **Entrée** (source des événements d’entrée) à partir de laquelle vous allez ingérer des données  
   - Votre **Sortie** (table de sortie) qui stockera des données transformées
   - Exemple de [requête SAQL](../../stream-analytics/stream-analytics-stream-analytics-query-patterns.md) avec l’instruction SELECT.
   - **Préversion de l’entrée** : Affiche l’instantané des dernières données entrantes à partir de la source des événements d’entrée.
     - Le type de sérialisation de vos données est automatiquement détecté (JSON ou CSV). Vous pouvez le remplacer manuellement par JSON/CSV/AVRO.
     - Vous pouvez afficher une préversion des données entrantes au format Tableau ou au format Brut.
     - Si les données qui apparaissent ne sont pas à jour, sélectionnez **Actualiser** pour afficher les événements les plus récents.
     - Sélectionnez **Sélectionner un intervalle de temps** pour tester votre requête par rapport à un intervalle de temps spécifique des événements entrants.
     - Sélectionnez **Charger l’exemple d’entrée** pour tester votre requête en téléchargeant un exemple de fichier JSON/CSV. Pour plus d’informations sur le test d’une requête SAQL, consultez [Tester un travail Azure Stream Analytics avec des exemples de données](../../stream-analytics/stream-analytics-test-query.md).

     ![tester la requête](./media/stream-data-stream-analytics-integration/test-query.png)

   - **Résultats de test** : Sélectionnez **Tester la requête** pour consulter les résultats de votre requête de diffusion en continu

     ![résultats de test](./media/stream-data-stream-analytics-integration/test-results.png)

   - **Schéma des résultats de test** : Affiche le schéma des résultats de votre requête de diffusion en continu après le test. Assurez-vous que le schéma des résultats de test correspond à votre schéma de sortie.

     ![schéma des résultats de test](./media/stream-data-stream-analytics-integration/test-results-schema.png)

   - **Schéma de sortie** : Contient le schéma de la table que vous avez sélectionnée à l’étape 5 (nouvelle ou existante).

      - Créer nouveau : Si vous avez sélectionné cette option à l’étape 5, vous ne verrez pas encore le schéma jusqu’à ce que vous démarriez le travail de diffusion en continu. Lorsque vous créez une nouvelle table, sélectionnez l’index de table approprié. Pour plus d’informations sur l’indexation des tables, consultez [Index en cluster et non-cluster décrits](/sql/relational-databases/indexes/clustered-and-nonclustered-indexes-described/).
      - Utiliser l’existant : Si vous avez sélectionné cette option à l’étape 5, vous verrez le schéma de la table sélectionnée.

7. Une fois que vous avez fini de créer et de tester la requête, sélectionnez **Enregistrer la requête**. Sélectionnez **Démarrer le travail Stream Analytics** pour commencer à ingérer des données transformées dans la table SQL. Une fois que vous avez finalisé les champs suivants, **démarrez** le travail.
   - **Heure de début de la sortie** : Cela définit l’heure de la première sortie du travail.  
     - Maintenant : Le travail démarre maintenant et traite les nouvelles données entrantes.
     - Personnalisé : Le travail démarre maintenant, mais traite des données à partir d’un point spécifique dans le temps (qui peut être dans le passé ou le futur). Pour plus d’informations, consultez [Comment démarrer un travail Azure Stream Analytics](../../stream-analytics/start-job.md).
   - **Unités de streaming** : Azure Stream Analytics est facturé selon le nombre d’unités de streaming requises pour traiter les données au sein du service. Pour plus d’informations, consultez [Tarification d’Azure Stream Analytics ](https://azure.microsoft.com/pricing/details/stream-analytics/).
   - **Gestion des erreurs des données de sortie** :  
     - Retry : Quand une erreur se produit, Azure Stream Analytics effectue indéfiniment de nouvelles tentatives d’écriture de l’événement jusqu’à la réussite de l’écriture. Il n’existe pas de délai d’expiration pour les nouvelles tentatives. Au final, le traitement de tous les événements suivants est bloqué par l’événement qui fait l’objet des nouvelles tentatives. Cette option est la stratégie de gestion des erreurs de sortie par défaut.
     - Déposer : Azure Stream Analytics supprime tous les événements de sortie qui entraînent une erreur de conversion de données. Les événements supprimés ne peuvent pas être récupérés pour un retraitement ultérieur. Toutes les erreurs temporaires (par exemple les erreurs de réseau) font l’objet de nouvelles tentatives, quelle que soit la configuration de la stratégie de gestion des erreurs de sortie.
   - **Paramètres de sortie de SQL Database** : Option qui permet d’hériter du schéma de partition de l’étape de requête précédente afin d’obtenir une topologie entièrement parallèle avec plusieurs rédacteurs dans la table. Pour plus d'informations, consultez [Sortie d'Azure Stream Analytics dans Azure SQL Database](../../stream-analytics/stream-analytics-sql-output-perf.md).
   - **Nombre maximal de lots** : Limite supérieure recommandée pour le nombre d’enregistrements envoyés avec chaque transaction d’insertion en bloc.  
    Pour plus d’informations sur la gestion des erreurs de sortie, consultez [Stratégies d’erreur en sortie dans Azure Stream Analytics](../../stream-analytics/stream-analytics-output-error-policy.md).  

     ![démarrer le travail](./media/stream-data-stream-analytics-integration/start-job.png)

8. Une fois que vous avez démarré le travail, le travail en cours d’exécution s’affiche dans la liste et vous pouvez effectuer les actions suivantes :
   - **Démarrer/arrêter le travail** : Si le travail est en cours d’exécution, vous pouvez l’arrêter. Si le travail est arrêté, vous pouvez le démarrer.
   - **Modifier le travail** : Vous pouvez modifier la requête. Si vous souhaitez effectuer d’autres modifications dans le travail, par exemple, ajoutez d’autres entrées/sorties, puis ouvrez le travail dans Stream Analytics. L’option Modifier est désactivée lorsque le travail est en cours d’exécution.
   - **Aperçu de la table de sortie** : Vous pouvez afficher un aperçu de la table dans l’éditeur de requête SQL.
   - **Ouvrir dans Stream Analytics** : Ouvrez le travail dans Stream Analytics pour afficher les détails de supervision et de débogage du travail.

     ![Travaux Stream Analytics](./media/stream-data-stream-analytics-integration/jobs.png)

## <a name="next-steps"></a>Étapes suivantes

- [Documentation d’Azure Stream Analytics](../../stream-analytics/index.yml)
- [Modèles de solution Azure Stream Analytics](../../stream-analytics/stream-analytics-solution-patterns.md)

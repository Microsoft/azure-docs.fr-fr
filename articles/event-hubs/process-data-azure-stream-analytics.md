---
title: Traiter des données Azure Event Hubs à l’aide de Stream Analytics | Microsoft Docs
description: Cet article vous explique comment traiter des données de votre instance Azure Event Hub à l’aide d’un travail Azure Stream Analytics.
ms.date: 06/23/2020
ms.topic: article
ms.openlocfilehash: 2db4fba59a1a06a24ee2939c51ecdf65aa06cef3
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85312891"
---
# <a name="process-data-from-your-event-hub-using-azure-stream-analytics"></a>Traiter des données de votre instance Event Hub avec Azure Stream Analytics 
Le service Azure Stream Analytics facilite l'ingestion, le traitement et l'analyse des données de streaming Azure Event Hubs, et offre de précieux insights permettant des actions en temps réel. Cette intégration vous permet de créer rapidement un pipeline d’analyse de chemin réactif. Vous pouvez utiliser le portail Azure pour visualiser les données entrantes et écrire une requête Stream Analytics. Une fois votre requête prête, vous pouvez la déplacer en production en quelques clics. 

## <a name="key-benefits"></a>Principaux avantages
Voici les principaux avantages de l’intégration d’Azure Event Hubs et d'Azure Stream Analytics : 
- **Aperçu des données** : vous pouvez afficher un aperçu des données entrantes à partir d’un Event Hub dans le portail Azure.
- **Tester votre requête** : préparez une requête de transformation et testez-la directement dans le portail Azure. Pour la syntaxe du langage de requête, consultez la documentation [Langage de requête Stream Analytics](/stream-analytics-query/built-in-functions-azure-stream-analytics).
- **Déployer votre requête en production** : vous pouvez déployer la requête en production en créant et en démarrant un travail Azure Stream Analytics.

## <a name="end-to-end-flow"></a>Flux de bout en bout

1. Connectez-vous au [portail Azure](https://portal.azure.com). 
1. Accédez à votre **espace de noms Event Hubs**, puis au **Event Hub** contenant les données entrantes. 
1. Sélectionnez **Traiter des données** sur la page Event Hub.  

    ![Vignette Traiter des données](./media/process-data-azure-stream-analytics/process-data-tile.png)
1. Sélectionnez **Explorer** sur la vignette **Activer des insights en temps réel à partir d’événements**. 

    ![Sélection de Stream Analytics](./media/process-data-azure-stream-analytics/process-data-page-explore-stream-analytics.png)
1. Une page de requête contenant les valeurs déjà définies pour les champs suivants s'affiche :
    1. Votre **Event Hub** en tant qu’entrée pour la requête.
    1. Un exemple de **requête SQL** avec l’instruction SELECT. 
    1. Un alias de **sortie** pour référencer vos résultats de test de requête. 

        ![Éditeur de requête](./media/process-data-azure-stream-analytics/query-editor.png)
        
        > [!NOTE]
        >  La première fois que vous utilisez cette fonctionnalité, cette page vous demande l'autorisation de créer un groupe de consommateurs et une stratégie pour permettre à votre Event Hub d'afficher un aperçu des données entrantes.
1. Sélectionnez **Créer** dans le volet **Aperçus des données entrantes** comme illustré sur l'image précédente. 
1. Vous verrez immédiatement un instantané des dernières données entrantes dans cet onglet.
    - Le type de sérialisation de vos données est automatiquement détecté (JSON ou CSV). Vous pouvez le remplacer manuellement par JSON/CSV/AVRO.
    - Vous pouvez afficher un aperçu des données entrantes au format tableau ou au format brut. 
    - Si les données qui apparaissent ne sont pas à jour, sélectionnez **Actualiser** pour afficher les événements les plus récents. 

        Voici un exemple de données au **format tableau** :   ![Résultats au format tableau](./media/process-data-azure-stream-analytics/snapshot-results.png)

        Voici un exemple de données au **format brut** : 

        ![Résultats au format brut](./media/process-data-azure-stream-analytics/snapshot-results-raw-format.png)
1. Sélectionnez **Tester la requête** pour afficher l’instantané des résultats du test de votre requête dans l'onglet **Résultats des tests**. Vous pouvez également télécharger les résultats.

    ![Résultats du test de la requête](./media/process-data-azure-stream-analytics/test-results.png)
1. Écrivez votre propre requête de transformation des données. Consultez [Référence sur le langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).
1. Une fois la requête testée, pour la déplacer en production, sélectionnez **Déployer la requête**. Pour déployer la requête, créez un travail Azure Stream Analytics vous permettant de définir une sortie pour votre travail, et démarrer le travail. Pour créer un travail Stream Analytics, spécifiez un nom pour ce travail, puis sélectionnez **Créer**.

      ![Créer un travail Azure Stream Analytics](./media/process-data-azure-stream-analytics/create-stream-analytics-job.png)

      > [!NOTE] 
      >  Nous vous recommandons de créer un groupe de consommateurs et une stratégie pour chaque nouveau travail Azure Stream Analytics que vous créez à partir de la page Event Hubs. Les groupes de consommateurs autorisent uniquement cinq lecteurs simultanés et dès lors, fournir un groupe de consommateurs dédié pour chaque travail permet d’éviter les erreurs liées à un dépassement de cette limite. Une stratégie dédiée vous permet de faire pivoter votre clé ou de révoquer des autorisations sans affecter d’autres ressources. 
1. Votre travail Stream Analytics est maintenant créé dans le même contexte que celui où vous avez testé votre requête, et l’entrée correspond à votre Event Hub. 

9.  Pour terminer le pipeline, définissez la **sortie** de la requête, puis sélectionnez **Démarrer** pour démarrer le travail.

    > [!NOTE]
    > Avant de démarrer le travail, n’oubliez pas de remplacer l’alias de sortie par le nom de sortie que vous avez créé dans Azure Stream Analytics.

      ![Définir la sortie et démarrer le travail](./media/process-data-azure-stream-analytics/set-output-start-job.png)


## <a name="known-limitations"></a>Limitations connues
Lors du test de votre requête, les résultats correspondants mettent environ 6 secondes à charger. Nous mettons tout en œuvre pour améliorer le niveau de performance des tests. Toutefois, lors d'un déploiement en production, Azure Stream Analytics affiche une latence inférieure à une seconde.

## <a name="streaming-units"></a>Unités de diffusion en continu
Votre travail Azure Stream Analytics affiche par défaut trois unités de streaming. Pour ajuster ce paramètre, sélectionnez **Mettre à l’échelle**dans le menu de gauche de la page **Travail Stream Analytics** du portail Azure. Pour plus d’informations sur les unités de streaming, consultez [Comprendre et ajuster les unités de streaming](../stream-analytics/stream-analytics-streaming-unit-consumption.md).

![Mettre à l'échelle les unités de streaming](./media/process-data-azure-stream-analytics/scale.png)

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations sur les requêtes Stream Analytics, consultez [Langage de requête Stream Analytics](/stream-analytics-query/built-in-functions-azure-stream-analytics)

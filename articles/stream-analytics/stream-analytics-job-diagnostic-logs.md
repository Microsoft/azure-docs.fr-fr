---
title: Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources
description: Cet article décrit comment analyser les journaux de ressources dans Azure Stream Analytics.
author: jseb225
ms.author: jeanb
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: troubleshooting
ms.custom: contperf-fy21q1
ms.date: 06/18/2020
ms.openlocfilehash: b29e0f99cb4549370be49dc5a1b11d367e30d8c0
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97029139"
---
# <a name="troubleshoot-azure-stream-analytics-by-using-resource-logs"></a>Résoudre les problèmes liés à Azure Stream Analytics à l’aide des journaux de ressources

Il arrive parfois qu’un travail Azure Stream Analytics s’arrête de manière inattendue. Il est important d’être en mesure de résoudre ce type de problème. Les échecs peuvent être liés à un résultat de requête inattendu, à une mauvaise connectivité aux périphériques ou à un arrêt inattendu du service. Les journaux de ressources dans Stream Analytics peuvent vous aider à identifier la cause des problèmes lorsqu’ils surviennent et à réduire le délai de récupération.

Il est vivement recommandé d’activer les journaux de ressources pour tous les travaux, ce qui facilite grandement le débogage et la surveillance.

## <a name="log-types"></a>Types de journaux

Stream Analytics fournit deux types de journaux d’activité :

* Les [journaux d’activité](../azure-monitor/platform/platform-logs-overview.md) (activés en permanence) fournissent des détails sur les opérations effectuées sur les travaux.

* Les [journaux de ressources](../azure-monitor/platform/platform-logs-overview.md) (configurables) offrent des informations plus détaillées sur tous les événements liés à un travail. Les journaux de ressources enregistrent les événements entre le moment où le travail est créé et celui où le travail est supprimé. Ils consignent les événements qui se produisent lorsque le travail est mis à jour et lors de son exécution.

> [!NOTE]
> Vous pouvez utiliser des services, tels que Stockage Azure, Azure Event Hubs et les journaux Azure Monitor pour analyser les données non conformes. Les frais qui vous seront facturés varient selon le modèle de tarification de chaque service.

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="debugging-using-activity-logs"></a>Débogage à l’aide des journaux d’activité

Les journaux d’activité sont activés par défaut et proposent des informations détaillées sur les opérations effectuées par votre travail Stream Analytics. Les informations présentes dans les journaux d’activité vous aident à déterminer la cause racine des problèmes ayant un impact sur votre travail. Pour utiliser les journaux d’activité dans Stream Analytics, procédez comme suit :

1. Connectez-vous au portail Azure et sélectionnez **Journal d’activité** sous **Vue d’ensemble**.

   ![Journal d’activité Stream Analytics](./media/stream-analytics-job-diagnostic-logs/stream-analytics-menu.png)

2. Vous pouvez consulter la liste des opérations effectuées. Toute opération ayant entraîné l’échec de votre travail est accompagnée d'une info-bulle rouge.

3. Cliquez sur une opération pour afficher un résumé la concernant. Les informations qui s'y trouvent sont souvent limitées. Pour plus d’informations sur l’opération, cliquez sur **JSON**.

   ![Résumé des opérations du journal d'activité Stream Analytics](./media/stream-analytics-job-diagnostic-logs/operation-summary.png)

4. Faites défiler jusqu'à la section **Propriétés** du JSON, qui fournit des détails sur l’erreur à l'origine de l'échec de l'opération. Dans cet exemple, cet échec était dû à une erreur d'exécution liée à des valeurs de latitude hors limites. Des incohérences dans les données qui sont traitées par un travail Stream Analytics provoque une erreur de données. Vous pouvez en savoir plus sur les différentes [erreurs de données d’entrée et de sortie, et la raison de leur survenue](./data-errors.md).

   ![Détails d'erreur JSON](./media/stream-analytics-job-diagnostic-logs/error-details.png)

5. Vous pouvez effectuer des actions correctives en fonction du message d’erreur dans JSON. Dans cet exemple, il convient d'ajouter des vérifications à la requête pour s'assurer que la valeur de latitude est comprise entre -90 degrés et 90 degrés.

6. Si le message d’erreur présent dans les journaux d’activité ne permet pas d’identifier la cause racine, activez les journaux de ressources et utilisez les journaux Azure Monitor.

## <a name="send-diagnostics-to-azure-monitor-logs"></a>Envoyer des diagnostics aux journaux Azure Monitor

Il est vivement conseillé d’activer les journaux de ressources et de les envoyer aux journaux Azure Monitor. Ils sont **désactivés** par défaut. Pour les activer, suivez ces étapes :

1.  Créez un espace de travail Log Analytics si vous n’en avez pas déjà un. Il est recommandé d’avoir votre espace de travail Log Analytics dans la même région que votre travail Stream Analytics.

2.  Connectez-vous au portail Azure et accédez à votre travail Stream Analytics. Sous **Supervision**, sélectionnez **Journaux de diagnostic**. Ensuite, sélectionnez **Activer les diagnostics**.

    ![Navigation du panneau aux journaux de ressources](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs-monitoring.png)  

2.  Indiquez un **nom** dans le champ **Nom des paramètres de diagnostic** et cochez les cases **Exécution** et **Création** sous **Journal**, ainsi que **AllMetrics** sous **Métrique**. Sélectionnez ensuite **Envoyer à Log Analytics** et choisissez votre espace de travail. Cliquez sur **Enregistrer**.

    ![Paramètres des journaux de ressources](./media/stream-analytics-job-diagnostic-logs/logs-setup.png)

3. Lorsque votre tâche Stream Analytics démarre, les journaux de ressources sont routés vers votre espace de travail Log Analytics. Pour afficher les journaux de ressources de votre travail, sélectionnez **Journaux** sous la section **Surveillance**.

   ![La capture d’écran montre le menu Général avec l’option Journaux sélectionnée.](./media/stream-analytics-job-diagnostic-logs/diagnostic-logs.png)

4. Stream Analytics fournit des requêtes prédéfinies qui vous permettent de rechercher facilement les journaux qui vous intéressent. Vous pouvez sélectionner n’importe quelle requête prédéfinie dans le volet gauche, puis sélectionner **Exécuter**. Les résultats de la requête s’affichent dans le volet inférieur. 

   ![La capture d’écran montre l’option Journaux pour un travail Stream Analytics.](./media/stream-analytics-job-diagnostic-logs/logs-example.png)

## <a name="resource-log-categories"></a>Catégories de journaux de ressources

Azure Stream Analytics capture deux catégories de journaux de ressources :

* **Création** : capture les événements de journal liés aux opérations de création de travaux (création du travail, ajout et suppression d’entrées et de sorties, ajout et mise à jour de la requête, démarrage et arrêt du travail).

* **Exécution** : capture les événements qui se produisent pendant l’exécution du travail.
    * Erreurs de connectivité
    * Erreurs de traitement des données, notamment :
        * Événements non conformes à la définition de la requête (types et valeurs de champs ne correspondant pas, champs manquants, etc.)
        * Erreurs d’évaluation d’expression
    * Autres erreurs et événements

## <a name="resource-logs-schema"></a>Schéma des journaux de ressource

Tous les journaux d’activité sont stockés au format JSON. Chaque entrée comprend les champs de chaîne courants suivants :

Nom | Description
------- | -------
time | L’horodatage (heure UTC) du journal.
resourceId | L’ID de la ressource sur laquelle l’opération a eu lieu, en majuscules. Comprend l’ID d’abonnement, le groupe de ressources et le nom du travail. Par exemple, **/SUBSCRIPTIONS/6503D296-DAC1-4449-9B03-609A1F4A1C87/RESOURCEGROUPS/MY-RESOURCE-GROUP/PROVIDERS/MICROSOFT.STREAMANALYTICS/STREAMINGJOBS/MYSTREAMINGJOB**.
catégorie | La catégorie de journal, **Exécution** ou **Création**.
operationName | Le nom de l’opération qui est journalisée. Par exemple, **Événements d’envoi : Échec d’écriture de la sortie SQL sur mysqloutput**.
status | État de l’opération. Par exemple, **Échec** ou **Réussite**.
level | Le niveau du journal. Par exemple, **Erreur**, **Avertissement** ou **Informations**.
properties | Détail spécifique de l’entrée du journal, sérialisé comme chaîne JSON. Pour plus d’informations, consultez les sections suivantes de cet article.

### <a name="execution-log-properties-schema"></a>Schéma de propriétés des journaux d’exécution

Les journaux d’activité d’exécution contiennent des informations sur les événements qui se sont produits pendant l’exécution du travail Stream Analytics. Le schéma de propriétés varie selon que l’événement est une erreur de données ou un événement générique.

### <a name="data-errors"></a>Erreurs de données

Toute erreur qui se produit lorsque le travail traite des données est consignée dans cette catégorie de journaux d’activité. La plupart du temps, ces journaux d’activité sont créés au cours des opérations de lecture, de sérialisation et d’écriture des données. Ces journaux d’activité n’incluent pas les erreurs de connectivité. Les erreurs de connectivité sont traitées comme des événements génériques. Vous pouvez en apprendre davantage sur la cause des différentes [erreurs de données d’entrée et de sortie](./data-errors.md).

Nom | Description
------- | -------
Source | Nom de l’entrée ou de la sortie du travail où l’erreur s’est produite.
Message | Message associé à l’erreur.
Type | Type d'erreur Par exemple, **DataConversionError**, **CsvParserError** ou **ServiceBusPropertyColumnMissingError**.
Données | Contient des données utiles pour localiser avec précision la source de l’erreur. Troncation possible en fonction de la taille.

En fonction de la valeur **operationName**, les erreurs de données ont le schéma suivant :

* Les **événements de sérialisation** se produisent pendant les opérations de lecture d’événements, lorsque les données en entrée ne répondent pas aux conditions du schéma de requête pour l’une des raisons suivantes :

   * *Incompatibilité de type pendant la (dé)sérialisation de l’événement :* identifie le champ à l’origine de l’erreur.

   * *Impossible de lire un événement, sérialisation non valide* : fournit des informations sur l’emplacement où l’erreur s’est produite dans les données d’entrée. (nom d’objet blob pour une entrée d’objet blob, décalage et exemple de données).

* Les **événements d’envoi** surviennent pendant les opérations d’écriture. Ils identifient l’événement de streaming à l’origine de l’erreur.

### <a name="generic-events"></a>Événements génériques

Les événements génériques couvrent tout le reste.

Nom | Description
-------- | --------
Error | (facultatif) Informations sur l’erreur, en général des informations sur l’exception si celles-ci sont disponibles.
Message| Message de journal.
Type | Type de message, correspond à la catégorisation interne des erreurs. Par exemple, **JobValidationError** ou **BlobOutputAdapterInitializationFailure**.
ID de corrélation : | GUID qui identifie de façon unique l’exécution du travail. Toutes les entrées du journal d’exécution générées depuis le démarrage du travail jusqu’à son arrêt ont le même **ID de corrélation**.

## <a name="next-steps"></a>Étapes suivantes

* [Erreurs de données Stream Analytics](./data-errors.md)
* [Informations de référence sur le langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
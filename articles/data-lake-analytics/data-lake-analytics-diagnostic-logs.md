---
title: Activer et afficher les journaux de diagnostic d’Azure Data Lake Analytics
description: Découvrez comment configurer les journaux de diagnostic pour Azure Data Lake Analytics et comment y accéder.
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 02/12/2018
ms.openlocfilehash: cd339729f2300ff7e13e7422bf73373b4ce4658e
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92221007"
---
# <a name="accessing-diagnostic-logs-for-azure-data-lake-analytics"></a>Accès aux journaux de diagnostic d’Azure Data Lake Analytics

La journalisation de diagnostic vous permet de collecter les pistes d’audit d’accès aux données. Ces journaux d’activité fournissent des informations comme :

* Une liste des utilisateurs qui ont accédé aux données.
* La fréquence à laquelle les données sont consultées.
* La quantité de données stockées dans le compte.

## <a name="enable-logging"></a>Activation de la journalisation

1. Connectez-vous au [Portail Azure](https://portal.azure.com).

2. Ouvrez votre compte Data Lake Analytics et sélectionnez **Journaux de diagnostic** dans la section __Surveiller__. Ensuite, sélectionnez __Activer les diagnostics__.

    ![Capture d’écran montrant l’action « Journaux de diagnostic » sélectionnée et « Activer les diagnostics pour collecter les journaux suivants » en évidence.](./media/data-lake-analytics-diagnostic-logs/turn-on-logging.png)

3. Dans __Paramètres de diagnostic__, entrez un __Nom__ pour cette configuration de journalisation, puis sélectionnez les options de journalisation.

    ![Activer les diagnostics pour collecter des journaux d’activité d’audit et de requêtes](./media/data-lake-analytics-diagnostic-logs/enable-diagnostic-logs.png "Activer les journaux de diagnostic")

   * Vous pouvez choisir de stocker/traiter les données de trois manières différentes.

     * Sélectionnez __Archive to a storage account (Archiver dans un compte de stockage)__ pour stocker les journaux d’activité dans un compte de stockage Azure. Utilisez cette option si vous souhaitez archiver les données. Si vous sélectionnez cette option, vous devez fournir un compte de stockage Azure dans lequel enregistrer les journaux d’activité.

     * Sélectionnez **Stream to an Event hub (Transmettre à un Event Hub)** pour transmettre les données journalisées à un Event Hub Azure. Utilisez cette option si vous disposez d’un pipeline de traitement en aval qui analyse les journaux d’activité entrants en temps réel. Si vous sélectionnez cette option, vous devez fournir les informations relatives au Event Hub Azure que vous souhaitez utiliser.

     * Sélectionnez __Envoyer à Log Analytics__ pour envoyer les données au service Azure Monitor. Utilisez cette option si vous souhaitez utiliser les journaux Azure Monitor pour recueillir et analyser les journaux.
   * Spécifiez si vous souhaitez obtenir des journaux d’audit ou des journaux d’activité de demande ou les deux.  Un journal des requêtes capture chaque demande d’API. Un journal d’audit enregistre toutes les opérations qui sont déclenchées par cette demande d’API.

   * Pour __Archiver dans un compte de stockage__, spécifiez le nombre de jours pendant lesquels conserver les données.

   * Cliquez sur __Enregistrer__.

        > [!NOTE]
        > Vous devez sélectionner __Archiver dans un compte de stockage__, __Diffuser vers Event Hub__ ou __Envoyer à Log Analytics__ avant de cliquer sur le bouton __Enregistrer__.

### <a name="use-the-azure-storage-account-that-contains-log-data"></a>Utiliser le compte de Stockage Azure qui contient les données du journal

1. Pour afficher les conteneurs d’objets blob qui contiennent les données de journalisation, ouvrez le compte de stockage Azure utilisé par Data Lake Analytics pour la journalisation, puis cliquez sur __Objets blob__.

   * Le conteneur **insights-logs-audit** contient les journaux d’audit.
   * Le conteneur **insights-logs-requests** contient les journaux d’activité de demande.

2. Les journaux d’activité sont stockés dans ces conteneurs, selon la structure de fichiers suivante :

   ```text
   resourceId=/
     SUBSCRIPTIONS/
       <<SUBSCRIPTION_ID>>/
         RESOURCEGROUPS/
           <<RESOURCE_GRP_NAME>>/
             PROVIDERS/
               MICROSOFT.DATALAKEANALYTICS/
                 ACCOUNTS/
                   <DATA_LAKE_ANALYTICS_NAME>>/
                     y=####/
                       m=##/
                         d=##/
                           h=##/
                             m=00/
                               PT1H.json
   ```

   > [!NOTE]
   > Le panneau `##` dans le chemin d’accès contiennent l’année, le mois, le jour et l’heure auxquels le journal a été créé. Data Lake Analytics crée un fichier toutes les heures, par conséquent, `m=` contient toujours une valeur de `00`.

    Par exemple, le chemin d’accès complet à un journal d’audit peut être :

    `https://adllogs.blob.core.windows.net/insights-logs-audit/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/mydatalakeanalytics/y=2016/m=07/d=18/h=04/m=00/PT1H.json`

    De même, le chemin d’accès complet à un journal de demande peut être :

    `https://adllogs.blob.core.windows.net/insights-logs-requests/resourceId=/SUBSCRIPTIONS/<sub-id>/RESOURCEGROUPS/myresourcegroup/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/mydatalakeanalytics/y=2016/m=07/d=18/h=14/m=00/PT1H.json`

## <a name="log-structure"></a>Structure journal

Les journaux d’activité d’audit et de demande présentent un format JSON structuré.

### <a name="request-logs"></a>journaux d’activité de demande

Voici un exemple d’entrée dans le journal de demande au format JSON. Chaque objet blob a un objet racine appelé **enregistrements** qui contient un tableau d’objets du journal.

```json
{
"records":
  [
    . . . .
    ,
    {
         "time": "2016-07-07T21:02:53.456Z",
         "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/<data_lake_analytics_account_name>",
         "category": "Requests",
         "operationName": "GetAggregatedJobHistory",
         "resultType": "200",
         "callerIpAddress": "::ffff:1.1.1.1",
         "correlationId": "4a11c709-05f5-417c-a98d-6e81b3e29c58",
         "identity": "1808bd5f-62af-45f4-89d8-03c5e81bac30",
         "properties": {
             "HttpMethod":"POST",
             "Path":"/JobAggregatedHistory",
             "RequestContentLength":122,
             "ClientRequestId":"3b7adbd9-3519-4f28-a61c-bd89506163b8",
             "StartTime":"2016-07-07T21:02:52.472Z",
             "EndTime":"2016-07-07T21:02:53.456Z"
             }
    }
    ,
    . . . .
  ]
}
```

#### <a name="request-log-schema"></a>Schéma du journal de requête

| Nom | Type | Description |
| --- | --- | --- |
| time |String |L’horodatage (heure UTC) du journal. |
| resourceId |String |L’identificateur de la ressource sur laquelle l’opération a eu lieu. |
| catégorie |String |La catégorie du journal. Par exemple, **Demandes**. |
| operationName |String |Le nom de l’opération qui est journalisée. Par exemple, GetAggregatedJobHistory. |
| resultType |String |L’état de l’opération. Par exemple, 200. |
| callerIpAddress |String |L’adresse IP du client qui a effectué la demande. |
| correlationId |String |L’identificateur du journal. Cette valeur peut être utilisée pour regrouper un ensemble d’entrées de journal associées. |
| identité |Object |L’identité qui a généré le journal. |
| properties |JSON |Consultez la section suivante (Schéma des propriétés de journal de demande) pour plus d’informations |

#### <a name="request-log-properties-schema"></a>Schéma des propriétés de journal de demande

| Nom | Type | Description |
| --- | --- | --- |
| HttpMethod |String |La méthode HTTP utilisée pour l’opération. Par exemple, GET. |
| Path |String |Le chemin d’accès vers l’emplacement où l’opération a eu lieu. |
| RequestContentLength |int |La longueur du contenu de la demande HTTP. |
| ClientRequestId |String |L’identificateur qui identifie de façon unique cette demande. |
| StartTime |String |L’heure à laquelle le serveur a reçu la demande. |
| EndTime |String |L’heure à laquelle le serveur a envoyé une réponse. |

### <a name="audit-logs"></a>Journaux d’audit

Voici un exemple d’entrée dans le journal d’audit au format JSON. Chaque objet blob a un objet racine appelé **enregistrements** qui contient un tableau d’objets du journal.

```json
{
"records":
  [
    {
         "time": "2016-07-28T19:15:16.245Z",
         "resourceId": "/SUBSCRIPTIONS/<subscription_id>/RESOURCEGROUPS/<resource_group_name>/PROVIDERS/MICROSOFT.DATALAKEANALYTICS/ACCOUNTS/<data_lake_ANALYTICS_account_name>",
         "category": "Audit",
         "operationName": "JobSubmitted",
         "identity": "user@somewhere.com",
         "properties": {
             "JobId":"D74B928F-5194-4E6C-971F-C27026C290E6",
             "JobName": "New Job",
             "JobRuntimeName": "default",
             "SubmitTime": "7/28/2016 7:14:57 PM"
             }
    }
  ]
}
```

#### <a name="audit-log-schema"></a>Schéma du journal d’audit

| Nom | Type | Description |
| --- | --- | --- |
| time |String |L’horodatage (heure UTC) du journal. |
| resourceId |String |L’identificateur de la ressource sur laquelle l’opération a eu lieu. |
| catégorie |String |La catégorie du journal. Par exemple, **Audit**. |
| operationName |String |Le nom de l’opération qui est journalisée. Par exemple, JobSubmitted. |
| resultType |String |Un sous-état de l’état de la tâche (operationName). |
| resultSignature |String |Informations supplémentaires sur l’état de la tâche (operationName). |
| identité |String |L’utilisateur qui a demandé l’opération. Par exemple : susan@contoso.com. |
| properties |JSON |Consultez la section suivante (Schéma des propriétés de journal d’audit) pour plus d’informations |

> [!NOTE]
> **resultType** et **resultSignature** fournissent des informations sur le résultat d’une opération et contiennent uniquement une valeur si une opération est terminée. Par exemple, ils contiennent uniquement une valeur quand **operationName** contient la valeur **JobStarted** ou **JobEnded**.
>
>

#### <a name="audit-log-properties-schema"></a>Schéma des propriétés de journal d’audit

| Nom | Type | Description |
| --- | --- | --- |
| JobId |String |L’ID affecté à la tâche. |
| JobName |String |Le nom fourni pour la tâche. |
| JobRunTime |String |Le runtime utilisé pour traiter la tâche. |
| SubmitTime |String |L’heure (UTC) à laquelle la tâche a été envoyée. |
| StartTime |String |L’heure à laquelle l’exécution de la tâche a commencé après la soumission (UTC). |
| EndTime |String |L’heure à laquelle la tâche s’est terminée. |
| Parallélisme |String |Le nombre d’unités Data Lake Analytics demandées pour cette tâche pendant la soumission. |

> [!NOTE]
> **SubmitTime**, **StartTime**, **EndTime** et **Parallélisme** fournissent des informations sur une opération. Ces entrées ne contiennent une valeur que si cette opération a démarré ou est terminée. Par exemple, **SubmitTime** contient uniquement une valeur après que **operationName** a la valeur **JobSubmitted**.

## <a name="process-the-log-data"></a>Traiter les données de journal

Azure Data Lake Analytics fournit un exemple de traitement et d’analyse des données de journal. Vous pouvez trouver l’exemple à l’adresse [https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample](https://github.com/Azure/AzureDataLake/tree/master/Samples/AzureDiagnosticsSample).

## <a name="next-steps"></a>Étapes suivantes

[Présentation d’Azure Data Lake Analytics](data-lake-analytics-overview.md)

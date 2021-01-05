---
title: Archiver des données de l’espace de travail Log Analytics dans le Stockage Azure à l’aide d’une application logique
description: Décrit une méthode permettant d’utiliser Azure Logic Apps pour interroger des données à partir d’un espace de travail Log Analytics et les envoyer au Stockage Azure.
ms.service: azure-monitor
ms.subservice: logs
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/02/2020
ms.openlocfilehash: 04f1eb0d9db00a2be1a4619cafe38aa18145fc78
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185995"
---
# <a name="archive-data-from-log-analytics-workspace-to-azure-storage-using-logic-app"></a>Archiver des données de l’espace de travail Log Analytics dans le Stockage Azure à l’aide d’une application logique
Cet article décrit une méthode permettant d’utiliser [Azure Logic Apps](../../logic-apps/index.yml) pour interroger des données à partir d’un espace de travail Log Analytics dans Azure Monitor et les envoyer au Stockage Azure. Utilisez ce processus lorsque vous devez exporter vos données de journal Azure Monitor pour des scénarios d’audit et de conformité, ou pour permettre à un autre service de récupérer ces données.  

## <a name="other-export-methods"></a>Autres méthodes d’exportation
La méthode décrite dans cet article décrit une exportation planifiée à partir d’une requête de journal à l’aide d’une application logique. Voici d’autres options d’exportation de données pour des scénarios particuliers :

- Pour exporter toutes les données de votre espace de travail Log Analytics vers un hub d’événements ou un compte de stockage Azure, utilisez la fonctionnalité d’exportation de données de l’espace de travail Log Analytics des journaux Azure Monitor. Consultez [Exportation des données de l’espace de travail Log Analytics dans Azure Monitor (préversion)](logs-data-export.md).
- Exportation ponctuelle à l’aide d’une application logique. Consultez [Connecteur Azure Monitor Logs pour Logic Apps et Power Automate](logicapp-flow-connector.md).
- Exportation ponctuelle vers l’ordinateur local à l’aide d’un script PowerShell. Consultez [Invoke-AzOperationalInsightsQueryExport]](https://www.powershellgallery.com/packages/Invoke-AzOperationalInsightsQueryExport).

## <a name="overview"></a>Vue d’ensemble
Cette procédure utilise le [connecteur Azure Monitor logs](/connectors/azuremonitorlogs/) qui vous permet d’exécuter une requête de journal à partir d’une application logique et d’utiliser sa sortie dans d’autres actions du flux de travail. Le [connecteur de Stockage Blob Azure](/connectors/azureblob/) est utilisé dans cette procédure pour envoyer la sortie de la requête vers le stockage Azure. Les autres actions sont décrites dans les sections ci-dessous.

![Vue d’ensemble de l’application logique](media/logs-export-logicapp/logic-app-overview.png)

Lorsque vous exportez des données à partir d’un espace de travail Log Analytics, vous devez filtrer et agréger vos données de journal et optimiser votre requête pour limiter la quantité des données traitées par le workflow de votre application logique aux données requises. Par exemple, si vous avez besoin d’archiver les événements de connexion, vous pouvez filtrer les événements requis et projeter uniquement les champs obligatoires à l’aide de la requête suivante : 

```json
SecurityEvent
| where EventID == 4624 or EventID == 4625
| project TimeGenerated , Account , AccountType , Computer
```

Lorsque vous exportez les données selon une planification, utilisez la fonction ingestion_time() dans votre requête pour vous assurer de ne pas manquer les données d’arrivée tardive. Si les données sont retardées en raison de problèmes de réseau ou de plateforme, l’utilisation du temps d’ingestion garantit qu’elles seront incluses dans la prochaine exécution de l’application logique. Consultez [Ajouter l’action Azure Monitor Logs](#add-azure-monitor-logs-action) pour obtenir un exemple.

## <a name="prerequisites"></a>Prérequis
Voici les prérequis à satisfaire avant d’effectuer cette procédure.

- Espace de travail Log Analytics. L’utilisateur qui crée l’application logique doit disposer au moins d’une autorisation de lecture sur l’espace de travail. 
- Compte de stockage Azure. Il n’est pas nécessaire que le compte de stockage figure dans le même abonnement que votre espace de travail Log Analytics. L’utilisateur qui crée l’application logique doit disposer d’une autorisation d’écriture sur le compte de stockage. 


## <a name="connector-limits"></a>Limites des connecteurs
L’espace de travail Log Analytics et les requêtes de journalisation dans Azure Monitor sont des services multilocataires qui incluent des limites qui protègent et isolent les clients et maintiennent la qualité de service. Lors de l’interrogation d’une grande quantité de données, vous devez prendre en compte les limites suivantes, qui peuvent affecter la façon dont vous configurez la périodicité de l’application logique et votre requête de journal :

- Les requêtes de journal ne peuvent pas retourner plus de 500 000 lignes.
- Les requêtes de journal ne peuvent pas retourner plus de 64 000 000 d’octets.
- Par défaut, les requêtes de journal ne peuvent pas s’exécuter pendant plus de 10 minutes. 
- Le connecteur Log Analytics est limité à 100 appels par minute.


## <a name="create-container-in-the-storage-account"></a>Créer un conteneur dans le compte de stockage
Utilisez la procédure décrite dans [Créer un conteneur](../../storage/blobs/storage-quickstart-blobs-portal.md#create-a-container) pour ajouter un conteneur à votre compte de stockage afin de stocker les données exportées. Le nom utilisé pour le conteneur dans cet article est **loganalytics-data**, mais vous pouvez utiliser n’importe quel nom.


## <a name="create-logic-app"></a>Créer une application logique

Accédez à **Logic Apps** sur le portail Azure, puis cliquez sur **Ajouter**. Sélectionnez un **abonnement** , un **groupe de ressources** et une **région** où stocker la nouvelle application logique, puis attribuez-lui un nom unique. Vous pouvez activer le paramètre **Log Analytics** pour collecter des informations sur les événements et les données du runtime, comme décrit dans [Configurer les journaux Azure Monitor et collecter des données de diagnostic pour Azure Logic Apps](../../logic-apps/monitor-logic-apps-log-analytics.md). Ce paramètre n’est pas nécessaire pour utiliser le connecteur Azure Monitor Logs.

![Créer une application logique](media/logs-export-logicapp/create-logic-app.png)


Cliquez sur **Vérifier + créer**, puis sur **Créer**. Une fois le déploiement terminé, cliquez sur **Accéder à la ressource** pour ouvrir le **concepteur Logic Apps**.

## <a name="create-a-trigger-for-the-logic-app"></a>Créer un déclencheur pour l’application logique
En dessous de **Démarrer avec un déclencheur courant**, sélectionnez **Récurrence**. Une application logique qui s’exécute automatiquement à intervalles réguliers est alors créée. Dans la zone **Fréquence** de l’action, sélectionnez **Heure** puis, dans la zone **Intervalle**, entrez **1** pour exécuter le workflow une fois par jour.

![Action de récurrence](media/logs-export-logicapp/recurrence-action.png)


### <a name="add-azure-monitor-logs-action"></a>Ajouter l’action Azure Monitor Logs
Cliquez sur **+ Nouvelle étape** pour ajouter une action qui s’exécute après l’action de récurrence. En dessous de **Choisir une action**, tapez **azure monitor**, puis sélectionnez **Azure Monitor Logs**.

![Action Azure Monitor Logs](media/logs-export-logicapp/select-azure-monitor-connector.png)

Cliquez sur **Azure Log Analytics – Exécuter la requête et répertorier les résultats**.

![Capture d’écran d’une nouvelle action ajoutée à une étape dans le Concepteur d’application logique. Azure Monitor Logs est mis en surbrillance sous Choisir une action.](media/logs-export-logicapp/select-query-action-list.png)

Vous serez invité à sélectionner un locataire et à accorder l’accès à l’espace de travail Log Analytics avec le compte que le workflow utilisera pour exécuter la requête.


## <a name="add-azure-monitor-logs-action"></a>Ajouter l’action Azure Monitor Logs
L’action Azure Monitor logs vous permet de spécifier la requête à exécuter. La requête de journal utilisée dans cet exemple est optimisée pour une périodicité horaire et collecte les données ingérées pendant la durée d’exécution particulière. Par exemple, si le workflow s’exécute à 04h35, l’intervalle de temps est de 04h00 à 05h00. Si vous modifiez l’application logique pour qu’elle s’exécute à une autre fréquence, vous devez également modifier la requête. Par exemple, si vous définissez la périodicité sur une exécution quotidienne, vous définissez startTime dans la requête sur startofday(make_datetime(year,month,day,0,0)). 

Sélectionnez l’**abonnement** et le **groupe de ressources** pour votre espace de travail Log Analytics. Sélectionnez *Espace de travail Log Analytics* pour **Type de ressource**, puis sélectionnez le nom de l’espace de travail en dessous de **Nom de la ressource**.

Dans la zone **Requête**, ajoutez la requête de journal suivante.  

```Kusto
let dt = now();
let year = datetime_part('year', dt);
let month = datetime_part('month', dt);
let day = datetime_part('day', dt);
let hour = datetime_part('hour', dt);
let startTime = make_datetime(year,month,day,hour,0)-1h;
let endTime = startTime + 1h - 1tick;
AzureActivity
| where ingestion_time() between(startTime .. endTime)
| project 
    TimeGenerated,
    BlobTime = startTime, 
    OperationName ,
    OperationNameValue ,
    Level ,
    ActivityStatus ,
    ResourceGroup ,
    SubscriptionId ,
    Category ,
    EventSubmissionTimestamp ,
    ClientIpAddress = parse_json(HTTPRequest).clientIpAddress ,
    ResourceId = _ResourceId 
```

L’**intervalle de temps** spécifie les enregistrements qui seront inclus dans la requête en fonction de la colonne **TimeGenerated**. Il doit être défini sur une valeur égale ou supérieure à l’intervalle de temps sélectionné dans la requête. Étant donné que cette requête n’utilise pas la colonne **TimeGenerated**, l’option **Définir dans la requête** n’est pas disponible. Consultez [Étendue de requête](../log-query/scope.md) pour plus d’informations sur l’intervalle de temps. 

Sélectionnez **4 dernières heures** pour l’**intervalle de temps**. Cela garantit que tous les enregistrements avec une durée d’ingestion supérieure à **TimeGenerated** sont inclus dans les résultats.
   
![Capture d’écran des paramètres de la nouvelle action Azure Monitor Logs nommée Exécuter une requête et visualiser les résultats.](media/logs-export-logicapp/run-query-list-action.png)


### <a name="add-parse-json-activity-optional"></a>Ajouter une activité Analyse JSON (facultatif)
La sortie de l’action **Exécuter la requête et répertorier les résultats** est au format JSON. Vous pouvez analyser ces données et les manipuler dans le cadre de la préparation de l’action **Composer**. 

Vous pouvez fournir un schéma JSON qui décrit la charge utile que vous comptez recevoir. Le concepteur analyse le contenu JSON à l’aide de ce schéma et génère des jetons conviviaux qui représentent les propriétés dans votre contenu JSON. Vous pouvez ensuite facilement référencer et utiliser ces propriétés dans l’ensemble de flux de travail de votre application logique. 


Cliquez sur **+ Nouvelle étape**, puis sur **+ Ajouter une action**. Sous **Choisir une action**, tapez **json**, puis sélectionnez **Analyser JSON**.

![Sélectionner l’activité Analyser JSON](media/logs-export-logicapp/select-parse-json.png)

Cliquez dans la zone **Contenu** pour afficher la liste des valeurs des activités précédentes. Sélectionnez **Corps** dans l’action **Exécuter la requête et répertorier les résultats**. Il s’agit de la sortie de la requête de journal.

[![Sélectionnez Corps](media/logs-export-logicapp/select-body.png)](media/logs-export-logicapp/select-body.png#lightbox)

5.  Cliquez sur **Utiliser l’exemple de charge utile pour générer le schéma**. Exécutez la requête de journal et copiez la sortie à utiliser pour l’exemple de charge utile.  Pour l’exemple de requête ici, vous pouvez utiliser la sortie suivante :


```json
{
    "TimeGenerated": "2020-09-29T23:11:02.578Z",
    "BlobTime": "2020-09-29T23:00:00Z",
    "OperationName": "Returns Storage Account SAS Token",
    "OperationNameValue": "MICROSOFT.RESOURCES/DEPLOYMENTS/WRITE",
    "Level": "Informational",
    "ActivityStatus": "Started",
    "ResourceGroup": "monitoring",
    "SubscriptionId": "00000000-0000-0000-0000-000000000000",
    "Category": "Administrative",
    "EventSubmissionTimestamp": "2020-09-29T23:11:02Z",
    "ClientIpAddress": "192.168.1.100",
    "ResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourcegroups/monitoring/providers/microsoft.storage/storageaccounts/my-storage-account"
}
```



![Charge utile de l’activité Analyser JSON](media/logs-export-logicapp/parse-json-payload.png)

## <a name="add-the-compose-action"></a>Ajouter l’action Composer
L’action **Composer** prend la sortie JSON analysée et crée l’objet que vous devez stocker dans l’objet Blob.

Cliquez sur **+ Nouvelle étape**, puis sur **+ Ajouter une action**. Sous **Choisir une action**, tapez **composer**, puis sélectionnez l’action **Composer**.

![Sélectionner l’action Composer](media/logs-export-logicapp/select-compose.png)


Cliquez dans la zone **Entrées** pour afficher la liste des valeurs des activités précédentes. Sélectionnez **Corps** dans l’action **Analyser JSON**. Il s’agit de la sortie analysée de la requête de journal.

[![Sélectionnez le corps pour l’action Composer](media/logs-export-logicapp/select-body-compose.png)](media/logs-export-logicapp/select-body-compose.png#lightbox)


## <a name="add-the-create-blob-action"></a>Ajouter l’action Créer un objet blob
L’action Créer un objet blob écrit le code JSON composé dans le stockage.

Cliquez sur **+ Nouvelle étape**, puis sur **+ Ajouter une action**. Sous **Choisir une action**, tapez **blob**, puis sélectionnez l’action **Créer un objet blob**.

![Sélectionner Créer un objet blob](media/logs-export-logicapp/select-create-blob.png)

Tapez un nom pour la connexion à votre compte de stockage dans **Nom de la connexion**, puis cliquez sur l’icône de dossier dans la zone **Chemin d’accès du dossier** pour sélectionner le conteneur dans votre compte de stockage. Cliquez sur le **Nom de l’objet blob** pour afficher la liste des valeurs des activités précédentes. Cliquez sur **Expression** et entrez une expression qui correspond à votre intervalle de temps. Pour cette requête qui est exécutée toutes les heures, l’expression suivante définit le nom de l’objet blob selon l’heure précédente : 

```json
subtractFromTime(formatDateTime(utcNow(),'yyyy-MM-ddTHH:00:00'), 1,'Hour')
```

[![Expression d’objet blob](media/logs-export-logicapp/blob-expression.png)](media/logs-export-logicapp/blob-expression.png#lightbox)

Cliquez dans la zone **Contenu de l’objet blob** pour afficher la liste des valeurs des activités précédentes, puis sélectionnez **Sorties** dans la section **Composer**.


![Expression Créer un objet blob](media/logs-export-logicapp/create-blob.png)


## <a name="test-the-logic-app"></a>Tester l’application logique
Testez le workflow en cliquant sur **Exécuter**. Si le workflow comporte des erreurs, cela est indiqué à l’étape présentant le problème. Vous pouvez afficher les exécutions et explorer chaque étape pour afficher l’entrée et la sortie, afin d’examiner les échecs. Consultez [Résoudre les problèmes et diagnostiquer les échecs de flux de travail dans Azure Logic Apps](../../logic-apps/logic-apps-diagnosing-failures.md) si nécessaire.

[![Historique des exécutions](media/logs-export-logicapp/runs-history.png)](media/logs-export-logicapp/runs-history.png#lightbox)


## <a name="view-logs-in-storage"></a>Afficher les journaux dans Stockage
Accédez au menu **Comptes de stockage** dans le portail Azure et sélectionnez votre compte de stockage. Cliquez sur la vignette **Objets blob** et sélectionnez le conteneur que vous avez spécifié dans l’action Créer un objet blob. Sélectionnez l’un des objets blob puis **Modifier l’objet blob**.

[![Données d’objet blob](media/logs-export-logicapp/blob-data.png)](media/logs-export-logicapp/blob-data.png#lightbox)

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [requêtes de journal dans Azure Monitor](../log-query/log-query-overview.md).
- Découvrez plus en détail [Logic Apps](../../logic-apps/index.yml).
- En savoir plus sur [Power Automate](https://flow.microsoft.com).
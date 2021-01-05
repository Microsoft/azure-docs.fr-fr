---
title: Mesures Azure Storage Analytics (classique)
description: Découvrez comment utiliser les mesures Storage Analytics dans le Stockage Azure. En savoir plus sur les métriques de transaction et de capacité, leur mode de stockage, leur activation et bien plus encore.
author: normesta
ms.service: storage
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: normesta
ms.reviewer: fryu
ms.subservice: common
ms.custom: monitoring, devx-track-csharp
ms.openlocfilehash: f1ab2be598a24a2448fed44742733633a8e0fc8f
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92787599"
---
# <a name="azure-storage-analytics-metrics-classic"></a>Mesures Azure Storage Analytics (classique)

Le Stockage Azure utilise Storage Analytics pour stocker des mesures qui comprennent les statistiques de transactions agrégées et les données de capacité relatives aux requêtes adressées à un service de stockage. Les transactions sont signalées au niveau de l’opération d’API et au niveau du service de stockage. La capacité est indiquée au niveau du service de stockage. Les données des mesures peuvent être utilisées pour :
- analyser l’utilisation du service de stockage ;
- diagnostiquer les problèmes liés aux requêtes envoyées au service de stockage ;
- améliorer les performances des applications qui utilisent un service.

 Les métriques de Storage Analytics sont activées par défaut pour les nouveaux comptes de stockage. Vous pouvez configurer les mesures dans le [Portail Azure](https://portal.azure.com/). Pour plus d’informations, consultez [Surveiller un compte de stockage dans le portail Azure](./storage-monitor-storage-account.md). Vous pouvez également activer Storage Analytics par programmation via l'API REST ou la bibliothèque cliente. Utilisez les opérations Set Service Properties pour activer Storage Analytics pour chaque service.  

> [!NOTE]
> Les mesures Storage Analytics sont disponibles pour le stockage d’objets blob Azure, Azure Files, le stockage File d’attente Azure et le stockage de tables Azure.
> Les mesures Storage Analytics sont maintenant des mesures classiques. Nous vous recommandons d’utiliser les [mesures de stockage dans Azure Monitor](../blobs/monitor-blob-storage.md) au lieu des mesures Storage Analytics.

## <a name="transaction-metrics"></a>Métriques de transaction  
 Un ensemble fiable de données est enregistré toutes les heures ou chaque minute pour chaque service de stockage et opération d’API demandée, notamment les entrées et sorties, la disponibilité, les erreurs et les pourcentages de requête triés par catégorie. La liste complète des détails de transaction est disponible dans la rubrique [Schéma de table de mesures Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema).  

 Les données de transaction sont enregistrées au niveau du service et au niveau de l’opération API. Au niveau du service, des statistiques résumant toutes les opérations d’API demandées sont écrites dans une entité de table toutes les heures, même si aucune requête n’a été adressée au service. Au niveau de l'opération d'API, les statistiques sont écrites uniquement dans une entité si l'opération a été demandée dans l'heure.  

 Par exemple, si vous exécutez une opération **GetBlob** sur votre service BLOB, les mesures Storage Analytics enregistrent la requête et l’incluent dans les données agrégées pour le service BLOB et l’opération **GetBlob**. Si aucune opération **GetBlob** n’est demandée pendant l’heure, aucune entité n’est écrite dans *$MetricsTransactionsBlob* pour cette opération.  

 Les mesures de transaction sont enregistrées pour les requêtes utilisateur et les demandes effectuées par Storage Analytics. Par exemple, les demandes par Storage Analytics pour écrire des journaux d’activité et des entités de table sont enregistrées.

## <a name="capacity-metrics"></a>Métriques de capacité  

> [!NOTE]
>  Actuellement, les mesures de capacité sont disponibles uniquement pour le service BLOB.

 Les données de capacité sont enregistrées quotidiennement pour le service BLOB d’un compte de stockage, et deux entités de table sont écrites. Une entité fournit des statistiques sur les données utilisateur et l'autre, sur le conteneur d'objets blob `$logs` utilisé par Storage Analytics. La table *$MetricsCapacityBlob* inclut les statistiques suivantes :  

- **Capacity**  : Quantité de stockage utilisée par le service BLOB du compte de stockage, en octets.  
- **ContainerCount**  : Nombre de conteneurs d’objets blob dans le service BLOB du compte de stockage.  
- **ObjectCount**  : Nombre d’objets blob de blocs ou de pages validés et non validés dans le service BLOB du compte de stockage.  

  Pour plus d’informations sur les mesures de capacité, voir [Schéma de table de mesures Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema).  

## <a name="how-metrics-are-stored"></a>Stockage des métriques  

 Toutes les données de mesures des services de stockage sont stockées dans trois tables réservées à ce service : une table pour les informations sur les transactions, une autre pour les informations sur les transactions par minute, et une troisième pour les informations sur la capacité. Les informations relatives aux transactions et aux transactions par minute se composent des données de requête et de réponse. Quant aux informations de capacité, elles comprennent des données sur l’utilisation du stockage. Les mesures par heure, les mesures par minute et la capacité pour le service blob d’un compte de stockage sont accessibles dans des tables nommées comme indiqué dans la table ci-dessous.  

|Niveau de métriques|Noms de tables|Versions prises en charge|  
|-------------------|-----------------|----------------------------|  
|Métriques toutes les heures, emplacement principal|-   $MetricsTransactionsBlob<br />-   $MetricsTransactionsTable<br />-   $MetricsTransactionsQueue|Versions antérieures au 15 août 2013, uniquement. Bien que ces noms soient toujours pris en charge, il est recommandé d’utiliser les tables répertoriées ci-dessous.|  
|Métriques toutes les heures, emplacement principal|-   $MetricsHourPrimaryTransactionsBlob<br />-   $MetricsHourPrimaryTransactionsTable<br />-   $MetricsHourPrimaryTransactionsQueue<br />-   $MetricsHourPrimaryTransactionsFile|Toutes les versions. La prise en charge des mesures de service Fichier est disponible uniquement dans la version du 5 avril 2015 et les versions ultérieures.|  
|Métriques par minute, emplacement principal|-   $MetricsMinutePrimaryTransactionsBlob<br />-   $MetricsMinutePrimaryTransactionsTable<br />-   $MetricsMinutePrimaryTransactionsQueue<br />-   $MetricsMinutePrimaryTransactionsFile|Toutes les versions. La prise en charge des mesures de service Fichier est disponible uniquement dans la version du 5 avril 2015 et les versions ultérieures.|  
|Métriques toutes les heures, emplacement secondaire|-   $MetricsHourSecondaryTransactionsBlob<br />-   $MetricsHourSecondaryTransactionsTable<br />-   $MetricsHourSecondaryTransactionsQueue|Toutes les versions. La géo-réplication redondante avec accès en lecture doit être activée.|  
|Métriques par minute, emplacement secondaire|-   $MetricsMinuteSecondaryTransactionsBlob<br />-   $MetricsMinuteSecondaryTransactionsTable<br />-   $MetricsMinuteSecondaryTransactionsQueue|Toutes les versions. La géo-réplication redondante avec accès en lecture doit être activée.|  
|Capacité (service BLOB uniquement)|$MetricsCapacityBlob|Toutes les versions.|  

 Ces tables sont automatiquement créées lorsque Storage Analytics est activé pour un point de terminaison de service de stockage. Elles sont accessibles via l’espace de noms du compte de stockage, par exemple : `https://<accountname>.table.core.windows.net/Tables("$MetricsTransactionsBlob")`. Les tables de mesures n’apparaissent pas dans une opération de liste et sont accessibles directement via le nom de la table.

## <a name="enable-metrics-by-using-the-azure-portal"></a>Activer les mesures à l’aide du Portail Azure
Pour activer les mesures dans le [Portail Azure](https://portal.azure.com), suivez les étapes suivantes :

1. Accédez à votre compte de stockage.
1. Sélectionnez **Paramètres de diagnostic (classique)** dans le volet du menu.
1. Vérifiez que l’option **État** est définie sur **Activé**.
1. Sélectionnez les mesures des services que vous souhaitez surveiller.
1. Spécifiez une stratégie de rétention pour indiquer la durée de conservation des métriques et de journalisation des données.
1. Sélectionnez **Enregistrer**.

Notez qu’actuellement le [Portail Azure](https://portal.azure.com) ne vous permet pas de configurer des mesures par minute dans votre compte de stockage. Vous devez les activer avec PowerShell ou de manière programmatique.

## <a name="enable-storage-metrics-by-using-powershell"></a>Activer les mesures de stockage avec PowerShell  
Vous pouvez utiliser PowerShell sur votre ordinateur local pour configurer des mesures de stockage dans votre compte de stockage : utilisez la cmdlet Azure PowerShell **Get-AzStorageServiceMetricsProperty** pour récupérer les paramètres actuels, et la cmdlet **Set-AzStorageServiceMetricsProperty** pour changer les paramètres actuels.  

Les cmdlets qui contrôlent les mesures de stockage utilisent les paramètres suivants :  

* **ServiceType** : les valeurs possibles sont **Blob** , **File d’attente** , **Table** et **Fichier**.
* **MetricsType** : peut avoir la valeur **Heure** ou **Minute**.  
* **MetricsLevel** : Les valeurs possibles sont les suivantes :
   * **Aucun**  : Désactive la surveillance.
   * **Service**  : Collecte des mesures telles que l’entrée et la sortie, la disponibilité, la latence et les pourcentages de réussite, qui sont agrégées pour les services BLOB, File d’attente, Table et Fichier.
   * **ServiceAndApi** : Outre les mesures de service, collecte le même ensemble de mesures pour chaque opération de stockage de l’API du service Stockage Azure.

Par exemple, la commande suivante active les métriques par minute pour le service BLOB dans votre compte de stockage avec une période de rétention de cinq jours : 

> [!NOTE]
> Cette commande part du principe que vous vous êtes connecté à votre abonnement Azure à l’aide de la commande `Connect-AzAccount`.

```powershell
$storageAccount = Get-AzStorageAccount -ResourceGroupName "<resource-group-name>" -AccountName "<storage-account-name>"

Set-AzStorageServiceMetricsProperty -MetricsType Minute -ServiceType Blob -MetricsLevel ServiceAndApi  -RetentionDays 5 -Context $storageAccount.Context
```  

* Remplacez la valeur d’espace réservé `<resource-group-name>` par le nom de votre groupe de ressources.      
* Remplacez la valeur d’espace réservé `<storage-account-name>` par le nom de votre compte de stockage.



La commande suivante récupère le niveau actuel des métriques par heure et la période de rétention en jours pour le service BLOB dans votre compte de stockage par défaut :  

```powershell
Get-AzStorageServiceMetricsProperty -MetricsType Hour -ServiceType Blob -Context $storagecontext.Context
```  

Pour plus d’informations sur la configuration des cmdlets Azure PowerShell avec votre abonnement Azure et sur la sélection du compte de stockage par défaut à utiliser, voir [Installation et configuration d’Azure PowerShell](/powershell/azure/).  

## <a name="enable-storage-metrics-programmatically"></a>Activer les mesures de stockage de manière programmatique  
Outre l’utilisation du Portail Azure ou des cmdlets Azure PowerShell pour contrôler les mesures de stockage, vous pouvez également utiliser l’une des API Stockage Azure. Par exemple, si vous exploitez un langage .NET, vous pouvez utiliser la bibliothèque cliente du Stockage Azure.  

Les classes **CloudBlobClient** , **CloudQueueClient** , **CloudTableClient** et **CloudFileClient** ont toutes des méthodes comme **SetServiceProperties** et **SetServicePropertiesAsync** qui utilisent un objet **ServiceProperties** en tant que paramètre. Vous pouvez utiliser l’objet **ServiceProperties** pour configurer les mesures de stockage. Ainsi, l’extrait de code C# suivant montre comment modifier les jours de rétention et le niveau de métrique pour la métrique de file d’attente horaire :  

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.HourMetrics.MetricsLevel = MetricsLevel.Service;  
serviceProperties.HourMetrics.RetentionDays = 10;  

queueClient.SetServiceProperties(serviceProperties);  
```  

Pour plus d’informations sur l’utilisation d’un langage .NET afin de configurer les mesures de stockage, consultez les [bibliothèques clientes de Stockage Azure pour .NET](/dotnet/api/overview/azure/storage).  

Pour obtenir des informations générales sur la configuration des mesures de stockage avec l’API REST, voir [Activation et configuration de Storage Analytics](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics).  

##  <a name="view-storage-metrics"></a>Afficher les mesures de stockage  
Après que vous avez configuré les métriques d’analyse du stockage pour surveiller votre compte de stockage, l’analyse du stockage enregistre les métriques dans des tables connues dans votre compte de stockage. Vous pouvez configurer des graphiques permettant de consulter des mesures horaires dans le [Portail Azure](https://portal.azure.com) :

1. Accédez à votre compte de stockage dans le [Portail Azure](https://portal.azure.com).
1. Sélectionnez **Métriques (classique)** dans le panneau de menu du service dont vous souhaitez afficher les mesures.
1. Sélectionnez le graphique que vous souhaitez configurer.
1. Dans le volet **Modifier le graphique** , sélectionnez l’ **Intervalle de temps** , le **Type de graphique** et les mesures que vous souhaitez afficher dans le graphique.

Dans la section **Supervision (classique)** du volet de menu du compte de stockage dans le Portail Azure, vous pouvez configurer le champ [Règles d’alerte](#metrics-alerts). Par exemple, vous pouvez envoyer des alertes par e-mail qui vous indiquent quand une mesure spécifique atteint une certaine valeur.

Si vous souhaitez télécharger les métriques pour un stockage à long terme ou pour les analyser localement, vous devez utiliser un outil ou écrire du code pour lire les tables. Vous devez télécharger les métriques par minute pour analyse. Les tables ne sont pas visibles si vous répertoriez toutes les tables dans votre compte de stockage, mais vous pouvez y accéder directement par nom. De nombreux outils de consultation du stockage prennent en charge ces tables et vous permettent de les afficher directement. Pour obtenir la liste des outils disponibles, consultez la rubrique relative aux [outils clients du Stockage Azure](./storage-explorers.md).

|Mesures|Noms de tables|Notes| 
|-|-|-|  
|Métriques toutes les heures|$MetricsHourPrimaryTransactionsBlob<br /><br /> $MetricsHourPrimaryTransactionsTable<br /><br /> $MetricsHourPrimaryTransactionsQueue<br /><br /> $MetricsHourPrimaryTransactionsFile|Dans les versions antérieures au 15 août 2013, ces tables étaient connues sous les noms suivants :<br /><br /> $MetricsTransactionsBlob<br /><br /> $MetricsTransactionsTable<br /><br /> $MetricsTransactionsQueue<br /><br /> Les mesures pour le service Fichier sont disponibles depuis la version du 5 avril 2015.|  
|Métriques par minute|$MetricsMinutePrimaryTransactionsBlob<br /><br /> $MetricsMinutePrimaryTransactionsTable<br /><br /> $MetricsMinutePrimaryTransactionsQueue<br /><br /> $MetricsMinutePrimaryTransactionsFile|Ne peut être activé qu’avec PowerShell ou de manière programmatique.<br /><br /> Les mesures pour le service Fichier sont disponibles depuis la version du 5 avril 2015.|  
|Capacité|$MetricsCapacityBlob|Service Blob uniquement.|  

Vous trouverez des informations complètes sur les schémas de ces tables dans la section [Schéma de table de mesures Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema). Les exemples de lignes ci-dessous montrent uniquement un sous-ensemble des colonnes disponibles, mais illustrent les différentes façons dont les mesures de stockage enregistrent ces informations :  

|PartitionKey|RowKey|Timestamp|TotalRequests|TotalBillableRequests|TotalIngress|TotalEgress|Disponibilité|AverageE2ELatency|AverageServerLatency|PercentSuccess| 
|-|-|-|-|-|-|-|-|-|-|-|  
|20140522T1100|user;All|2014-05-22T11:01:16.7650250Z|7|7|4003|46801|100|104.4286|6.857143|100|  
|20140522T1100|user;QueryEntities|2014-05-22T11:01:16.7640250Z|5|5|2694|45951|100|143.8|7.8|100|  
|20140522T1100|user;QueryEntity|2014-05-22T11:01:16.7650250Z|1|1|538|633|100|3|3|100|  
|20140522T1100|user;UpdateEntity|2014-05-22T11:01:16.7650250Z|1|1|771|217|100|9|6|100|  

Dans cet exemple de données de mesures par minute, la clé de partition utilise une résolution d’une minute. La clé de ligne identifie le type d’informations stockées dans la ligne. Les informations incluent le type d’accès et de requête :  

-   Le type d’accès a la valeur **user** ou **system** , **user** correspondant à toutes les requêtes de l’utilisateur au service de stockage et **system** , à toutes les demandes formulées par Storage Analytics.  
-   Le type de requête peut avoir la valeur **all** , auquel cas il s’agit d’une ligne de résumé, ou identifie l’API spécifique comme **QueryEntity** ou **UpdateEntity**.  

Les exemples de données ci-dessus montrent tous les enregistrements pour une seule minute (à partir de 11h00). Ainsi, la somme des requêtes **QueryEntities** , plus le nombre de requêtes **QueryEntity** et **UpdateEntity** donne un résultat de sept, ce qui correspond au total indiqué sur la ligne **user:All**. De même, vous pouvez déduire la latence de bout en bout moyenne (104,4286) sur la ligne **user:All** en effectuant le calcul suivant : ((143,8 * 5) + 3 + 9)/7.  

## <a name="metrics-alerts"></a>Alertes liées aux métriques
Il peut être intéressant de définir des alertes dans le [Portail Azure](https://portal.azure.com), afin d’être averti automatiquement de tout changement important de comportement de vos services de stockage. Si vous utilisez un outil Explorateur Stockage pour télécharger ces données de mesures dans un format délimité, vous pouvez utiliser Microsoft Excel pour les analyser. Pour obtenir la liste des outils Explorateur Stockage disponibles, consultez la rubrique relative aux [outils clients du Stockage Azure](./storage-explorers.md). Vous pouvez configurer des alertes dans le panneau **Alerte (classique)** , accessible sous **Surveillance (classique)** dans le volet du menu du compte de stockage.

> [!IMPORTANT]
> Il peut y avoir un décalage entre un événement de stockage et l’enregistrement des données de mesures par heure ou par minute correspondantes. Dans le cas des mesures par minute, plusieurs minutes de données peuvent être écrites à la fois. Cela peut entraîner l’agrégation des transactions des minutes précédentes dans la transaction pour la minute actuelle. Dans ce cas, le service d’alerte peut ne pas disposer de toutes les données de mesures disponibles pour l’intervalle d’alerte configuré, et cela peut entraîner des déclenchements d’alerte inattendus.
>

## <a name="access-metrics-data-programmatically"></a>Accès aux données de mesures par programmation  
L’exemple de code suivant en C# accède aux mesures par minute pour une plage de minutes et affiche les résultats dans une fenêtre de console. L’exemple de code utilise la version 4.x ou une version ultérieure de la bibliothèque cliente de stockage Azure. Celle-ci comprend la classe **CloudAnalyticsClient** , qui simplifie l’accès aux tables de mesures de stockage. 

> [!NOTE]
> La classe **CloudAnalyticsClient** n’est pas incluse dans la bibliothèque de client Stockage Blob Azure v12 pour .NET. Au **31 août 2023** , les métriques Storage Analytics, également appelées *métriques classiques* seront retirées. Pour plus d’informations, consultez l’[annonce officielle](https://azure.microsoft.com/updates/azure-storage-classic-metrics-will-be-retired-on-31-august-2023/). Si vous utilisez des métriques classiques, nous vous recommandons de passer aux métriques dans Azure Monitor avant cette date. 

```csharp
private static void PrintMinuteMetrics(CloudAnalyticsClient analyticsClient, DateTimeOffset startDateTime, DateTimeOffset endDateTime)  
{  
 // Convert the dates to the format used in the PartitionKey.  
 var start = startDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  
 var end = endDateTime.ToUniversalTime().ToString("yyyyMMdd'T'HHmm");  

 var services = Enum.GetValues(typeof(StorageService));  
 foreach (StorageService service in services)  
 {  
     Console.WriteLine("Minute Metrics for Service {0} from {1} to {2} UTC", service, start, end);  
     var metricsQuery = analyticsClient.CreateMinuteMetricsQuery(service, StorageLocation.Primary);  
     var t = analyticsClient.GetMinuteMetricsTable(service);  
     var opContext = new OperationContext();  
     var query =  
             from entity in metricsQuery  
             // Note, you can't filter using the entity properties Time, AccessType, or TransactionType  
             // because they are calculated fields in the MetricsEntity class.  
             // The PartitionKey identifies the DataTime of the metrics.  
             where entity.PartitionKey.CompareTo(start) >= 0 && entity.PartitionKey.CompareTo(end) <= 0   
             select entity;  

     // Filter on "user" transactions after fetching the metrics from Azure Table storage.  
     // (StartsWith is not supported using LINQ with Azure Table storage.)  
     var results = query.ToList().Where(m => m.RowKey.StartsWith("user"));  
     var resultString = results.Aggregate(new StringBuilder(), (builder, metrics) => builder.AppendLine(MetricsString(metrics, opContext))).ToString();  
     Console.WriteLine(resultString);  
 }  
}  

private static string MetricsString(MetricsEntity entity, OperationContext opContext)  
{  
 var entityProperties = entity.WriteEntity(opContext);  
 var entityString =  
         string.Format("Time: {0}, ", entity.Time) +  
         string.Format("AccessType: {0}, ", entity.AccessType) +  
         string.Format("TransactionType: {0}, ", entity.TransactionType) +  
         string.Join(",", entityProperties.Select(e => new KeyValuePair<string, string>(e.Key.ToString(), e.Value.PropertyAsObject.ToString())));  
 return entityString;  
}  
```  

## <a name="billing-on-storage-metrics"></a>Facturation sur les métriques de stockage
Les demandes d’écriture pour créer des entités de table pour les métriques sont facturées au tarif standard applicable à toutes les opérations Azure Storage.  

Les demandes de lecture et de suppression formulées par un client sur des données de métriques sont aussi facturées au tarif standard. Si vous avez configuré une stratégie de conservation des données, vous n’êtes pas facturé quand le stockage Azure supprime les anciennes données de mesures. Toutefois, si vous supprimez des données analytiques, les opérations de suppression sont facturées à votre compte.  

La capacité utilisée par les tables de métriques est également facturable. Utilisez les informations suivantes pour estimer la capacité utilisée pour stocker les données de mesures :  

-   En une heure, pour un service qui utilise toutes les API de chaque service, environ 148 Ko de données sont stockés par heure dans les tables de transaction de mesures si vous avez activé la synthèse au niveau des services et de l’API.  
-   Si, en une heure, un service utilise toutes les API du service, alors environ 12 Ko de données sont stockés par heure dans les tables de transaction de mesures si vous avez uniquement activé la synthèse au niveau des services.  
-   Deux lignes supplémentaires sont ajoutées chaque jour dans la table de capacité pour les objets blob, si vous avez choisi de recevoir les journaux. Cela implique que la taille de cette table augmente d’environ 300 octets maximum chaque jour.

## <a name="next-steps"></a>Étapes suivantes
* [Surveiller un compte de stockage](https://www.windowsazure.com/manage/services/storage/how-to-monitor-a-storage-account/)   
* [Schéma de table de mesures Storage Analytics](/rest/api/storageservices/storage-analytics-metrics-table-schema)   
* [Opérations et messages d’état enregistrés Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)   
* [Journalisation Storage Analytics](storage-analytics-logging.md)
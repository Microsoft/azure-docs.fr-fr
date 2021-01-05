---
title: Journalisation Azure Storage Analytics
description: Utilisez Storage Analytics pour consigner des détails sur les demandes de stockage Azure. Consultez les demandes journalisées, la façon dont les journaux sont stockés, comment activer la journalisation du stockage, et bien plus encore.
author: normesta
ms.service: storage
ms.subservice: common
ms.topic: conceptual
ms.date: 07/23/2020
ms.author: normesta
ms.reviewer: fryu
ms.custom: monitoring, devx-track-csharp
ms.openlocfilehash: eb71de223e2d840e0caa0444b837e16e1f091414
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96484785"
---
# <a name="azure-storage-analytics-logging"></a>Journalisation Azure Storage Analytics

Storage Analytics enregistre des informations détaillées sur les demandes ayant réussi ou échoué pour un service de stockage. Ces informations peuvent servir à analyser des demandes individuelles et à diagnostiquer les problèmes au niveau d'un service de stockage. Les demandes sont enregistrées sur la base du meilleur effort.

 La journalisation Storage Analytics n’est pas activée par défaut pour votre compte de stockage. Vous pouvez l’activer dans le [portail Azure](https://portal.azure.com/). Pour plus d’informations, consultez l’article [Surveillance d’un compte de stockage dans le portail Azure](./storage-monitor-storage-account.md). Vous pouvez également activer Storage Analytics par programmation via l'API REST ou la bibliothèque cliente. Utilisez les opérations [Get Blob Service Properties](/rest/api/storageservices/Blob-Service-REST-API), [Get Queue Service Properties](/rest/api/storageservices/Get-Queue-Service-Properties) et [Get Table Service Properties](/rest/api/storageservices/Get-Table-Service-Properties) pour activer Storage Analytics pour chaque service.

 Les entrées de journal sont créées uniquement si des demandes sont effectuées sur le point de terminaison de service. Par exemple, si un compte de stockage a une activité dans son point de terminaison Blob, mais pas dans ses points de terminaison de Table ou de File d’attente, seuls des journaux d’activité relatifs au service BLOB seront créés.

> [!NOTE]
>  La journalisation Storage Analytics est disponible pour les services Blob, File d’attente et de Table. La journalisation Storage Analytics est également disponible pour les comptes [BlockBlobStorage](../blobs/storage-blob-create-account-block-blob.md) avec des performances Premium. Toutefois, elle n’est pas disponible pour les comptes v2 à usage général avec des performances Premium.

## <a name="requests-logged-in-logging"></a>Requêtes consignées lors de la journalisation
### <a name="logging-authenticated-requests"></a>Enregistrement des demandes authentifiées

 Les types de demandes authentifiées suivants sont enregistrés :

- Demandes ayant réussi
- Demandes ayant échoué, y compris les erreurs de délai d’expiration, limitation, réseau, autorisation et autres erreurs
- Requêtes utilisant une signature d’accès partagé (SAS) ou OAuth, y compris les requêtes ayant réussi et ayant échoué
- Demandes de données d’analyse

  Les demandes effectuées par Storage Analytics lui-même, telles que la création ou la suppression d'un journal, ne sont pas enregistrées. La liste complète des données enregistrées est disponible dans les rubriques [Opérations et messages d’état enregistrés Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) et [Format de journal de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

### <a name="logging-anonymous-requests"></a>Journalisation des demandes anonymes

 Les types de demandes anonymes suivants sont enregistrés :

- Demandes ayant réussi
- Erreurs de serveur
- Erreurs de délai d’expiration pour le client et le serveur
- Demandes GET ayant échoué avec le code d’erreur 304 (non modifié)

  Aucune autre demande anonyme ayant échoué n'est enregistrée. La liste complète des données enregistrées est disponible dans les rubriques [Opérations et messages d’état enregistrés Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages) et [Format de journal de Storage Analytics](/rest/api/storageservices/storage-analytics-log-format).

## <a name="how-logs-are-stored"></a>Mode de stockage des journaux d’activité

Tous les journaux d’activité sont stockés dans des objets blob de blocs dans un conteneur nommé `$logs`, qui est automatiquement créé lorsque Storage Analytics est activé pour un compte de stockage. Le conteneur `http://<accountname>.blob.core.windows.net/$logs` se trouve dans l’espace de noms d’objets blob du compte de stockage, par exemple : `$logs`. Ce conteneur ne peut pas être supprimé une fois Storage Analytics activé, mais son contenu peut l'être. Si vous utilisez votre outil de navigation du stockage pour accéder directement au conteneur, vous verrez tous les objets Blob contenant vos données de journalisation.

> [!NOTE]
>  Le conteneur `$logs` ne s’affiche pas lorsqu’une opération d’énumération des conteneurs est en cours, comme l’opération Répertorier les conteneurs. Vous devez y accéder directement. Par exemple, vous pouvez utiliser l’opération Liste des blobs pour accéder aux objets blob dans le conteneur `$logs`.

À mesure que des demandes sont enregistrées, Storage Analytics télécharge les résultats intermédiaires en tant que blocs. Périodiquement, Storage Analytics valide ces blocs et les rend accessibles sous forme d'objets blob. Une heure peut être nécessaire avant l’affichage des données de journaux dans les objets Blob du conteneur **$logs** en raison de la fréquence à laquelle le service de stockage vide les enregistreurs de journal. Il peut exister des enregistrements en double pour les journaux d’activité créés dans la même heure. Vous pouvez déterminer si un enregistrement est un doublon en vérifiant les nombres **RequestId** et **Operation**.

Si vous avez un volume élevé de données de journal avec plusieurs fichiers pour chaque heure, vous pouvez utiliser les métadonnées d’objets Blob pour déterminer les données que le journal contient en examinant les champs de métadonnées d’objet Blob. Ceci est également utile car il peut parfois y avoir un délai d’écriture des données dans les fichiers journaux : les métadonnées d’objet Blob indiquent plus précisément le contenu d’objet Blob que le nom d’objet Blob.

La plupart des outils de navigation du stockage vous permettent d’afficher les métadonnées d’objets Blob. Vous pouvez également lire ces informations à l’aide de PowerShell ou par programmation. L’extrait de code PowerShell suivant est un exemple de filtrage de la liste d’objets Blob de journal par nom pour spécifier une heure et par métadonnées pour identifier uniquement les journaux qui contiennent des opérations d’**écriture**.  

 ```powershell
 Get-AzStorageBlob -Container '$logs' |  
 Where-Object {  
     $_.Name -match 'table/2014/05/21/05' -and   
     $_.ICloudBlob.Metadata.LogType -match 'write'  
 } |  
 ForEach-Object {  
     "{0}  {1}  {2}  {3}" –f $_.Name,   
     $_.ICloudBlob.Metadata.StartTime,   
     $_.ICloudBlob.Metadata.EndTime,   
     $_.ICloudBlob.Metadata.LogType  
 }  
 ```  

Pour plus d’informations sur la liste d’objets Blob par programmation, consultez [Énumération de ressources d’objet Blob](/rest/api/storageservices/Enumerating-Blob-Resources) et [Définition et récupération de propriétés et métadonnées de ressources d’objets Blob](/rest/api/storageservices/Setting-and-Retrieving-Properties-and-Metadata-for-Blob-Resources).  

### <a name="log-naming-conventions"></a>Conventions d'appellation de journal

 Chaque journal est écrit au format suivant :

 `<service-name>/YYYY/MM/DD/hhmm/<counter>.log`

 Le tableau suivant décrit chaque attribut du nom du journal :

|Attribut|Description|
|---------------|-----------------|
|`<service-name>`|Nom du service de stockage. Par exemple : `blob`, `table` ou `queue`|
|`YYYY`|Année à quatre chiffres pour le journal. Par exemple : `2011`|
|`MM`|Mois à deux chiffres pour le journal. Par exemple : `07`|
|`DD`|Jour à deux chiffres pour le journal. Par exemple : `31`|
|`hh`|Heure à deux chiffres qui indique l’heure de début pour les journaux d’activité, au format UTC 24 heures. Par exemple : `18`|
|`mm`|Nombre à deux chiffres qui indique la minute de début pour les journaux d’activité. **Remarque :**  Cette valeur n’étant pas prise en charge dans la version actuelle de Storage Analytics, elle est toujours égale à `00`.|
|`<counter>`|Compteur de base zéro à six chiffres qui indique le nombre d'objets blob de journal générés pour le service de stockage durant une période d'une heure. Ce compteur commence à `000000`. Par exemple : `000001`|

 Voici un exemple complet de nom de journal qui combine les exemples ci-dessus :

 `blob/2011/07/31/1800/000001.log`

 Voici un exemple d’URI qui peut être utilisé pour accéder au journal ci-dessus :

 `https://<accountname>.blob.core.windows.net/$logs/blob/2011/07/31/1800/000001.log`

 Lorsqu'une demande de stockage est enregistrée, le nom du journal qui en résulte correspond à l'heure de fin de l'opération demandée. Par exemple, si une requête GetBlob se termine à 18h30 le 31 juillet 2011, le journal est écrit avec le préfixe suivant : `blob/2011/07/31/1800/`

### <a name="log-metadata"></a>Métadonnées du journal

 Tous les objets blob de journal sont stockés avec des métadonnées qui peuvent être utilisées pour identifier les données de journalisation contenues dans l'objet blob. Le tableau suivant décrit chaque attribut de métadonnées :

|Attribut|Description|
|---------------|-----------------|
|`LogType`|Décrit si le journal contient des informations relatives aux opérations de lecture, écriture ou suppression. Cette valeur peut inclure un type ou une combinaison des trois, séparés par des virgules.<br /><br /> Exemple 1 : `write`<br /><br /> Exemple 2 : `read,write`<br /><br /> Exemple 3 : `read,write,delete`|
|`StartTime`|Heure la plus antérieure d’une entrée dans le journal, au format `YYYY-MM-DDThh:mm:ssZ`. Par exemple : `2011-07-31T18:21:46Z`|
|`EndTime`|Heure la plus récente d’une entrée dans le journal, au format `YYYY-MM-DDThh:mm:ssZ`. Par exemple : `2011-07-31T18:22:09Z`|
|`LogVersion`|Version du format du journal.|

 La liste suivante présente un exemple de métadonnées complètes utilisant les exemples ci-dessus :

-   `LogType=write`
-   `StartTime=2011-07-31T18:21:46Z`
-   `EndTime=2011-07-31T18:22:09Z`
-   `LogVersion=1.0`

## <a name="enable-storage-logging"></a>Activer la journalisation du stockage

Vous pouvez activer la journalisation du stockage avec le Portail Azure, PowerShell et des Kits de développement logiciel (SDK) de stockage.

### <a name="enable-storage-logging-using-the-azure-portal"></a>Activer la journalisation du stockage à l’aide du Portail Azure  

Dans le Portail Azure, utilisez le panneau **Paramètres de diagnostic (classiques)** pour contrôler la journalisation du stockage, accessible depuis la section **Supervision (classique)** du **panneau Menu** d’un compte de stockage.

Vous pouvez spécifier les services de stockage que vous souhaitez consigner et la période de rétention (en jours) des données journalisées.  

### <a name="enable-storage-logging-using-powershell"></a>Activer la journalisation du stockage à l’aide de PowerShell  

 Vous pouvez utiliser PowerShell sur votre ordinateur local pour configurer la journalisation du stockage dans votre compte de stockage. Utilisez la cmdlet Azure PowerShell **Get-AzStorageServiceLoggingProperty** pour récupérer les paramètres actuels et la cmdlet **Set-AzStorageServiceLoggingProperty** pour modifier les paramètres actuels.  

 Les cmdlets qui contrôlent la journalisation du stockage utilisent un paramètre **LoggingOperations**, une chaîne contenant une liste de types de requêtes à consigner séparée par des virgules. Les trois types de requêtes possibles sont **lire**, **écrire** et **supprimer**. Pour désactiver la journalisation, utilisez la valeur **aucun** pour le paramètre **LoggingOperations**.  

 La commande suivante active la journalisation des requêtes de lecture, d’écriture et de suppression dans le service de File d’attente de votre compte de stockage par défaut avec une rétention de cinq jours :  

```powershell
Set-AzStorageServiceLoggingProperty -ServiceType Queue -LoggingOperations read,write,delete -RetentionDays 5  
```  

 La commande suivante désactive la journalisation du service de table dans votre compte de stockage par défaut :  

```powershell
Set-AzStorageServiceLoggingProperty -ServiceType Table -LoggingOperations none  
```  

 Pour plus d’informations sur la configuration des applets de commande Azure PowerShell avec votre abonnement Azure et sur la sélection du compte de stockage par défaut à utiliser, consultez : [Guide pratique pour installer et configurer Azure PowerShell](/powershell/azure/).  

### <a name="enable-storage-logging-programmatically"></a>Activer la journalisation du stockage par programmation  

 Outre l’utilisation du Portail Azure ou des cmdlets Azure PowerShell pour contrôler la journalisation du stockage, vous pouvez également utiliser l’une des API Stockage Azure. Par exemple, si vous utilisez un langage .NET, vous pouvez utiliser la bibliothèque cliente de stockage.  

# <a name="net-v12-sdk"></a>[SDK \.NET v12](#tab/dotnet)

:::code language="csharp" source="~/azure-storage-snippets/queues/howto/dotnet/dotnet-v12/Monitoring.cs" id="snippet_EnableDiagnosticLogs":::

# <a name="net-v11-sdk"></a>[SDK \.NET v11](#tab/dotnet11)

```csharp
var storageAccount = CloudStorageAccount.Parse(connStr);  
var queueClient = storageAccount.CreateCloudQueueClient();  
var serviceProperties = queueClient.GetServiceProperties();  

serviceProperties.Logging.LoggingOperations = LoggingOperations.All;  
serviceProperties.Logging.RetentionDays = 2;  

queueClient.SetServiceProperties(serviceProperties);  
```  

---


 Pour plus d’informations sur l’utilisation d’un langage .NET pour configurer la journalisation du stockage, consultez la [référence sur la bibliothèque cliente de stockage](/previous-versions/azure/dn261237(v=azure.100)).  

 Pour obtenir des informations générales sur la configuration de la journalisation du stockage avec l’API REST, consultez [Activation et configuration de Storage Analytics](/rest/api/storageservices/Enabling-and-Configuring-Storage-Analytics).  

## <a name="download-storage-logging-log-data"></a>Télécharger les données du journal de journalisation du stockage

 Pour afficher et analyser vos données de journal, vous devez télécharger les objets Blob qui contiennent les données de journal souhaitées sur un ordinateur local. De nombreux outils de consultation du stockage vous permettent de télécharger des objets Blob de votre compte de stockage. Vous pouvez également utiliser l’outil de copie Azure de ligne de commande fourni par l’équipe de Stockage Azure [AzCopy](storage-use-azcopy-v10.md) pour télécharger vos données de journal.  
 
>[!NOTE]
> Le conteneur `$logs` n’étant pas intégré avec Event Grid, vous ne recevez pas de notifications lors de l’écriture de fichiers journaux. 

 Pour vous assurer de télécharger les données de journal souhaitées et éviter de télécharger les mêmes données plusieurs fois :  

-   Utilisez la convention d’affectation de noms de l’horodatage pour les objets Blob contenant les données de journal pour suivre les objets Blob que vous avez déjà téléchargés pour analyse afin d’éviter de télécharger plusieurs fois les mêmes données.  

-   Utilisez les métadonnées des objets Blob contenant des données de journal pour identifier la période pendant laquelle l’objet Blob conserve les données de journal afin d’identifier l’objet Blob exact que vous devez télécharger.  

Pour commencer à utiliser AzCopy, consultez [Prise en main d’AzCopy](storage-use-azcopy-v10.md) 

L’exemple suivant montre comment vous pouvez télécharger les données de journal du service de file d’attente pour les heures en commençant à 9h, 10h et 11h le 20 mai 2014.

```
azcopy copy 'https://mystorageaccount.blob.core.windows.net/$logs/queue' 'C:\Logs\Storage' --include-path '2014/05/20/09;2014/05/20/10;2014/05/20/11' --recursive
```

Pour en savoir plus sur le téléchargement de fichiers spécifiques, consultez [Télécharger des fichiers spécifiques](./storage-use-azcopy-blobs.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#download-specific-files).

Lorsque vous avez téléchargé vos données de journal, vous pouvez afficher les entrées de journal dans les fichiers. Ces fichiers journaux utilisent un format de texte délimité que de nombreux outils de lecture de journaux sont en mesure d’analyser (pour plus d’informations, consultez le guide [Surveiller, diagnostiquer et résoudre les problèmes liés au service Stockage Microsoft Azure](storage-monitoring-diagnosing-troubleshooting.md)). Différents outils possèdent des fonctionnalités différentes de mise en forme, de filtrage, de tri et de recherche de contenu de vos fichiers journaux. Pour plus d’informations sur le format du fichier journal de journalisation du stockage et son contenu, consultez [Format du journal de l’analyse de stockage](/rest/api/storageservices/storage-analytics-log-format) et [Opérations et messages d’état enregistrés Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages).

## <a name="next-steps"></a>Étapes suivantes

* [Format du journal de l’analyse de stockage](/rest/api/storageservices/storage-analytics-log-format)
* [Opérations et messages d'état enregistrés Storage Analytics](/rest/api/storageservices/storage-analytics-logged-operations-and-status-messages)
* [Métriques Storage Analytics (classiques)](storage-analytics-metrics.md)
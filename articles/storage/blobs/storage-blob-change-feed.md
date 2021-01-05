---
title: Flux de modification dans Stockage Blob Azure | Microsoft Docs
description: En savoir plus sur les journaux de flux de modification dans Stockage Blob Azure et leur utilisation.
author: normesta
ms.author: normesta
ms.date: 09/08/2020
ms.topic: how-to
ms.service: storage
ms.subservice: blobs
ms.reviewer: sadodd
ms.openlocfilehash: 7174f7dd53387de9a569a5ddcadc08c32692c749
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95997101"
---
# <a name="change-feed-support-in-azure-blob-storage"></a>Prise en charge du flux de modification dans Stockage Blob Azure

L’objectif du flux de modification est de fournir des journaux des transactions de toutes les modifications apportées aux objets blob et aux métadonnées d’objets blob dans votre compte de stockage. Le flux de modification fournit un journal **ordonné**, **garanti**, **durable**, **immuable** et **en lecture seule** de ces changements. Les applications clientes peuvent lire ces journaux à tout moment, soit en diffusion en continu, soit en mode de traitement par lot. Le flux de modification vous permet de créer des solutions efficaces et évolutives qui traitent les événements de modification qui se produisent dans votre compte Stockage Blob à moindre coût.

[!INCLUDE [storage-data-lake-gen2-support](../../../includes/storage-data-lake-gen2-support.md)]

Le flux de modification est stocké en tant que [blobs](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs) dans un conteneur spécial dans votre compte de stockage au [tarif standard des objets blob](https://azure.microsoft.com/pricing/details/storage/blobs/). Vous pouvez contrôler la période de rétention de ces fichiers en fonction de vos exigences (voir les [conditions](#conditions) de la version actuelle). Les événements de modification sont ajoutés au flux de modification sous forme d’enregistrements dans la spécification de format [Apache Avro](https://avro.apache.org/docs/1.8.2/spec.html) : un format binaire, compact et rapide qui fournit des structures de données enrichies avec un schéma inlined. Ce format est largement utilisé dans l’écosystème Hadoop, Stream Analytics et Azure Data Factory.

Vous pouvez traiter ces journaux de manière asynchrone, incrémentielle ou complète. Un nombre quelconque d’applications clientes peuvent lire indépendamment le flux de modification, en parallèle, et à leur propre rythme. Les applications d’analytique que sont notamment [Apache Drill](https://drill.apache.org/docs/querying-avro-files/) ou [Apache Spark](https://spark.apache.org/docs/latest/sql-data-sources-avro.html) peuvent utiliser des journaux directement comme fichiers Avro, ce qui vous permet de les traiter à très faible coût, avec une bande passante élevée et sans avoir à écrire une application personnalisée.

La prise en charge des flux de modification convient parfaitement aux scénarios qui traitent les données en fonction d’objets qui ont été modifiés. Par exemple, les applications peuvent :

  - Mettre à jour un index secondaire, se synchroniser avec un cache, un moteur de recherche ou tout autre scénario de gestion de contenu.
  
  - Extraire des informations et des métriques d’analytique métier, en fonction des modifications apportées à vos objets, en mode de diffusion en continu ou de traitement par lot.
  
  - Stocker, auditer et analyser les modifications apportées à vos objets, sur n’importe quelle période de temps, à des fins de sécurité, de conformité ou de renseignement pour la gestion des données d’entreprise.

  - Créer des solutions pour sauvegarder, mettre en miroir ou répliquer l’état des objets dans votre compte pour la gestion d’urgence ou la conformité.

  - Créer des pipelines d’application connectés qui réagissent aux événements de modification ou planifier des exécutions en fonction de l’objet créé ou modifié.
  
Le flux de modification est une fonctionnalité prérequise pour la [réplication d’objets](object-replication-overview.md) et la [restauration dans le temps pour les objets blob de blocs](point-in-time-restore-overview.md).

> [!NOTE]
> Le flux de modification fournit un modèle de journal durable et ordonné des modifications apportées à un objet blob. Les modifications sont écrites et mises à disposition dans votre journal de flux de modification quelques minutes après la modification. Si votre application doit réagir aux événements plus rapidement que cela, envisagez d’utiliser des [événements Stockage Blob](storage-blob-event-overview.md) à la place. Les [événements Stockage Blob](storage-blob-event-overview.md) sont des événements uniques en temps réel qui permettent à vos fonctions ou applications Azure de réagir rapidement aux modifications apportées à un objet blob. 

## <a name="enable-and-disable-the-change-feed"></a>Activer et désactiver le flux de modification

Vous devez activer le flux de modification sur votre compte de stockage pour commencer la capture et l’enregistrement des modifications. Désactivez le flux de modification pour arrêter la capture des modifications. Vous pouvez activer et désactiver les modifications à l’aide de modèles Azure Resource Manager sur le Portail ou sur PowerShell.

Voici quelques éléments à prendre en compte lorsque vous activez le flux de modification.

- Il n’existe qu’un seul flux de modification pour le service blob dans chaque compte de stockage ; il est stocké dans le conteneur **$blobchangefeed**.

- Les modifications de création, de mise à jour et de suppression sont capturées uniquement au niveau du service blob.

- Le flux de modification capture *toutes* les modifications pour tous les événements disponibles qui se produisent sur le compte. Les applications clientes peuvent filtrer les types d’événements selon les besoins. (Voir les [conditions](#conditions) de la version actuelle).

- Seuls les comptes de stockage GPv2 et Blob peuvent activer le flux de modification. Les comptes BlockBlobStorage Premium et les comptes prenant en charge les espaces de noms hiérarchiques ne sont actuellement pas pris en charge. Les comptes de stockage GPv1 ne sont pas pris en charge, mais peuvent être mis à niveau vers GPv2 sans temps d’arrêt. Pour plus d’informations, consultez [Mettre à niveau vers un compte de stockage GPv2](../common/storage-account-upgrade.md).

### <a name="portal"></a>[Portail](#tab/azure-portal)

Activez le flux de modification sur votre compte de stockage à l’aide du portail Azure :

1. Dans le [Portail Azure](https://portal.azure.com/), sélectionnez votre compte de stockage.

2. Accédez à l’option **Protection des données** sous **Service blob**.

3. Cliquez sur **Activé** sous **Flux de modification d’objets blob**.

4. Choisissez le bouton **Enregistrer** pour confirmer vos paramètres de **protection des données**.

    ![Capture d’écran montrant les paramètres de protection des données.](media/soft-delete-blob-enable/storage-blob-soft-delete-portal-configuration.png)

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Activez le flux de modification à l’aide de PowerShell :

1. Installez la dernière version de PowerShellGet.

   ```powershell
   Install-Module PowerShellGet –Repository PSGallery –Force
   ```

2. Fermez, puis rouvrez la console PowerShell.

3. Installez la version 2.5.0 ou une version ultérieure du module **Az.Storage**.

   ```powershell
   Install-Module Az.Storage –Repository PSGallery -RequiredVersion 2.5.0 –AllowClobber –Force
   ```

4. Connectez-vous à votre abonnement Azure avec la commande `Connect-AzAccount` et suivez les instructions à l’écran pour l’authentification.

   ```powershell
   Connect-AzAccount
   ```

5. Activez le flux de modification pour votre compte de stockage.

   ```powershell
   Update-AzStorageBlobServiceProperty -EnableChangeFeed $true
   ```

### <a name="template"></a>[Modèle](#tab/template)
Utilisez un modèle Azure Resource Manager pour activer le flux de modification sur votre compte de stockage existant par le biais du portail Azure :

1. Dans le Portail Azure, choisissez **Créer une ressource**.

2. Dans **Rechercher sur la Place de marché**, tapez **déploiement de modèle**, puis appuyez sur **Entrée**.

3. Choisissez **[Déployer un modèle personnalisé](https://portal.azure.com/#create/Microsoft.Template)** , puis **Créer votre propre modèle dans l’éditeur**.

4. Dans l’éditeur de modèle, collez le code JSON suivant. Remplacez la valeur d’espace réservé `<accountName>` par le nom de votre compte de stockage.

   ```json
   {
       "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
       "contentVersion": "1.0.0.0",
       "parameters": {},
       "variables": {},
       "resources": [{
           "type": "Microsoft.Storage/storageAccounts/blobServices",
           "apiVersion": "2019-04-01",
           "name": "<accountName>/default",
           "properties": {
               "changeFeed": {
                   "enabled": true
               }
           } 
        }]
   }
   ```
    
5. Choisissez le bouton **Enregistrer**, spécifiez le groupe de ressources du compte, puis choisissez le bouton **Acheter** pour déployer le modèle et activer le flux de modification.

---

## <a name="consume-the-change-feed"></a>Utiliser le flux de modification

Le flux de modification produit plusieurs métadonnées et fichiers journaux. Ces fichiers se trouvent dans le conteneur **$blobchangefeed** du compte de stockage. 

> [!NOTE]
> Dans la version actuelle, le conteneur $blobchangefeed est visible uniquement sur le portail Azure, il n’est pas visible dans l’Explorateur Stockage Azure. Actuellement, vous ne pouvez pas voir le conteneur $blobchangefeed lorsque vous appelez l’API ListContainers, mais vous pouvez appeler l’API ListBlobs directement sur le conteneur pour voir les objets blob.

Vos applications clientes peuvent consommer le flux de modification en utilisant la bibliothèque du processeur de flux de modification d’objets blob fournie avec le SDK du processeur de flux de modification. 

Consultez [Traiter les journaux de flux de modification dans Stockage Blob Azure](storage-blob-change-feed-how-to.md).

## <a name="understand-change-feed-organization"></a>Comprendre l’organisation du flux de modification

<a id="segment-index"></a>

### <a name="segments"></a>Segments

Le flux de changements est un journal de changements qui sont organisés en *segments* **horaires**, mais ajoutés et mis à jour à intervalles réguliers de quelques minutes. Ces segments sont créés uniquement quand des événements de modification d’objet blob se produisent dans l’heure. Cela permet à votre application cliente de consommer des modifications qui se produisent dans des plages de temps spécifiques sans avoir à effectuer une recherche dans le journal entier. Pour en savoir plus, consultez [Spécifications](#specifications).

Un segment horaire disponible du flux de modification est décrit dans un fichier manifeste qui spécifie les chemins d’accès aux fichiers de flux de modification pour ce segment. La liste du répertoire virtuel `$blobchangefeed/idx/segments/` montre ces segments dans l’ordre chronologique. Le chemin d’accès du segment décrit le début de l’intervalle horaire représenté par le segment. Vous pouvez utiliser cette liste pour filtrer les segments de journaux qui vous intéressent.

```text
Name                                                                    Blob Type    Blob Tier      Length  Content Type    
----------------------------------------------------------------------  -----------  -----------  --------  ----------------
$blobchangefeed/idx/segments/1601/01/01/0000/meta.json                  BlockBlob                      584  application/json
$blobchangefeed/idx/segments/2019/02/22/1810/meta.json                  BlockBlob                      584  application/json
$blobchangefeed/idx/segments/2019/02/22/1910/meta.json                  BlockBlob                      584  application/json
$blobchangefeed/idx/segments/2019/02/23/0110/meta.json                  BlockBlob                      584  application/json
```

> [!NOTE]
> Le fichier `$blobchangefeed/idx/segments/1601/01/01/0000/meta.json` est automatiquement créé lorsque vous activez le flux de modification. Vous pouvez ignorer sans risque ce fichier. Il s’agit d’un fichier d’initialisation toujours vide. 

Le fichier manifeste de segment (`meta.json`) affiche le chemin d’accès des fichiers de flux de modification pour ce segment dans la propriété `chunkFilePaths`. Voici un exemple d’un fichier manifeste de segment.

```json
{
    "version": 0,
    "begin": "2019-02-22T18:10:00.000Z",
    "intervalSecs": 3600,
    "status": "Finalized",
    "config": {
        "version": 0,
        "configVersionEtag": "0x8d698f0fba563db",
        "numShards": 2,
        "recordsFormat": "avro",
        "formatSchemaVersion": 1,
        "shardDistFnVersion": 1
    },
    "chunkFilePaths": [
        "$blobchangefeed/log/00/2019/02/22/1810/",
        "$blobchangefeed/log/01/2019/02/22/1810/"
    ],
    "storageDiagnostics": {
        "version": 0,
        "lastModifiedTime": "2019-02-22T18:11:01.187Z",
        "data": {
            "aid": "55e507bf-8006-0000-00d9-ca346706b70c"
        }
    }
}
```

> [!NOTE]
> Le conteneur `$blobchangefeed` s’affiche uniquement une fois que vous avez activé la fonctionnalité du flux de modification sur votre compte. Vous devrez patienter quelques minutes après avoir activé le flux de modification pour pouvoir énumérer les objets blob du conteneur. 

<a id="log-files"></a>

### <a name="change-event-records"></a>Enregistrements d’événements de modification

Les fichiers de flux de modification contiennent une série d’enregistrements d’événements de modification. Chaque enregistrement d’événement de modification correspond à une modification apportée à un objet blob individuel. Les enregistrements sont sérialisés et écrits dans le fichier à l’aide de la spécification du format [Apache Avro](https://avro.apache.org/docs/1.8.2/spec.html). Les enregistrements peuvent être lus à l’aide de la spécification de format de fichier Avro. Plusieurs bibliothèques sont disponibles pour traiter les fichiers dans ce format.

Les fichiers de flux de modification sont stockés dans le répertoire virtuel `$blobchangefeed/log/` sous forme de [blobs ajoutés](/rest/api/storageservices/understanding-block-blobs--append-blobs--and-page-blobs#about-append-blobs). Le premier fichier de flux de modification sous chaque chemin d’accès comportera `00000` dans le nom de fichier (par exemple `00000.avro`). Le nom de chaque fichier journal suivant ajouté à ce chemin d’accès est incrémenté de 1 (par exemple : `00001.avro`).

Les types d’événements suivants sont capturés dans les enregistrements de flux de modification :
- BlobCreated
- BlobDeleted
- BlobPropertiesUpdated
- BlobSnapshotCreated

Voici un exemple d’enregistrement d’événement de modification à partir d’un fichier de flux de modification converti en JSON.

```json
{
     "schemaVersion": 1,
     "topic": "/subscriptions/dd40261b-437d-43d0-86cf-ef222b78fd15/resourceGroups/sadodd/providers/Microsoft.Storage/storageAccounts/mytestaccount",
     "subject": "/blobServices/default/containers/mytestcontainer/blobs/mytestblob",
     "eventType": "BlobCreated",
     "eventTime": "2019-02-22T18:12:01.079Z",
     "id": "55e5531f-8006-0000-00da-ca3467000000",
     "data": {
         "api": "PutBlob",
         "clientRequestId": "edf598f4-e501-4750-a3ba-9752bb22df39",
         "requestId": "00000000-0000-0000-0000-000000000000",
         "etag": "0x8D698F13DCB47F6",
         "contentType": "application/octet-stream",
         "contentLength": 128,
         "blobType": "BlockBlob",
         "url": "",
         "sequencer": "000000000000000100000000000000060000000000006d8a",
         "storageDiagnostics": {
             "bid": "11cda41c-13d8-49c9-b7b6-bc55c41b3e75",
             "seq": "(6,5614,28042,28038)",
             "sid": "591651bd-8eb3-c864-1001-fcd187be3efd"
         }
  }
}
```

Pour obtenir une description de chaque propriété, consultez [Schéma d’événement Azure Event Grid pour Stockage Blob](../../event-grid/event-schema-blob-storage.md?toc=%2fazure%2fstorage%2fblobs%2ftoc.json#event-properties). Les événements BlobPropertiesUpdated et BlobSnapshotCreated sont actuellement exclusifs pour le flux de modification et ne sont pas encore pris en charge pour les événements de stockage Blob.

> [!NOTE]
> Les fichiers de flux de modification d’un segment n’apparaissent pas immédiatement après la création d’un segment. La durée du délai est comprise dans l’intervalle normal de latence de publication du flux de modification, ce qui représente quelques minutes après la modification.

<a id="specifications"></a>

## <a name="specifications"></a>Spécifications

- Les enregistrements d’événements de modification sont uniquement ajoutés au flux de modification. Une fois ces enregistrements ajoutés, ils sont immuables et la position d’enregistrement est stable. Les applications clientes peuvent conserver leur propre point de contrôle à la position de lecture du flux de modification.

- Les enregistrements d’événements de modification sont ajoutés dans l’ordre de quelques minutes après la modification. Les applications clientes peuvent choisir de consommer des enregistrements à mesure qu’ils sont ajoutés pour un accès en continu ou en bloc à un autre moment.

- Les enregistrements d’événements de modification sont classés par ordre de modification **par blob**. L’ordre des modifications entre les objets blob n’est pas défini dans Stockage Blob Azure. Toutes les modifications apportées à un segment précédent sont antérieures à toute modification apportée aux segments suivants.

- Les enregistrements d’événements de modification sont sérialisés dans le fichier journal à l’aide de la spécification de format [Apache Avro 1.8.2](https://avro.apache.org/docs/1.8.2/spec.html).

- Les enregistrements d’événements de modification pour lesquelles `eventType`a une valeur de `Control` sont des enregistrements système internes et ne reflètent pas une modification apportée aux objets de votre compte. Vous pouvez ignorer ces enregistrements sans risque.

- Les valeurs du conteneur des propriétés `storageDiagnonstics` sont destinées à un usage interne uniquement et ne sont pas conçues pour être utilisées par votre application. Vos applications ne doivent pas avoir de dépendance contractuelle sur ces données. Vous pouvez ignorer ces propriétés sans risque.

- Le temps représenté par le segment est **approximatif** avec des limites de 15 minutes. Ainsi, pour garantir l’utilisation de tous les enregistrements dans un intervalle de temps spécifié, utilisez le segment consécutif de l’heure précédente et de l’heure suivante.

- Chaque segment peut avoir un nombre différent de `chunkFilePaths` en raison du partitionnement interne du flux de journaux pour gérer le débit de publication. Les fichiers journaux de chaque `chunkFilePath` sont assurés de contenir des objets blob mutuellement exclusifs et peuvent être utilisés et traités en parallèle sans violer l’ordre des modifications par blob pendant l’itération.

- Les segments commencent avec l’état `Publishing`. Une fois que l’ajout des enregistrements au segment est terminé, l’état est `Finalized`. Les fichiers journaux d’un segment dont la date est postérieure à celle de la propriété `LastConsumable` du fichier `$blobchangefeed/meta/Segments.json` ne doivent pas être consommés par votre application. Voici un exemple de la propriété `LastConsumable`dans un fichier `$blobchangefeed/meta/Segments.json` :

```json
{
    "version": 0,
    "lastConsumable": "2019-02-23T01:10:00.000Z",
    "storageDiagnostics": {
        "version": 0,
        "lastModifiedTime": "2019-02-23T02:24:00.556Z",
        "data": {
            "aid": "55e551e3-8006-0000-00da-ca346706bfe4",
            "lfz": "2019-02-22T19:10:00.000Z"
        }
    }
}

```

<a id="conditions"></a>

## <a name="conditions-and-known-issues"></a>Conditions et problèmes connus

Cette section décrit les problèmes connus et les conditions de la version actuelle du flux de modification. 

- Les enregistrements d’événements de modification d’une modification unique peuvent apparaître plusieurs fois dans votre flux de modification.
- Vous ne pouvez pas encore gérer la durée de vie des fichiers journaux du flux de modification en définissant une stratégie de rétention basée sur la durée et vous ne pouvez pas supprimer les objets Blob.
- La propriété `url` du fichier journal est actuellement toujours vide.
- La propriété `LastConsumable` du fichier segments.json ne répertorie pas le tout premier segment que le flux de modification finalise. Ce problème se produit uniquement après la finalisation du premier segment. Tous les segments suivants après la première heure sont capturés avec précision dans la propriété `LastConsumable`.
- Actuellement, vous ne pouvez pas voir le conteneur **$blobchangefeed** quand vous appelez l’API ListContainers, et le conteneur n’apparaît pas dans le Portail Azure ou l’Explorateur Stockage. Vous pouvez afficher le contenu en appelant l’API ListBlobs directement sur le conteneur $blobchangefeed.
- Les comptes de stockage qui ont lancé précédemment un [basculement de compte](../common/storage-disaster-recovery-guidance.md) peuvent rencontrer des problèmes de non-affichage du fichier journal. Tout basculement de compte ultérieur peut également avoir un impact sur le fichier journal.

## <a name="faq"></a>Questions fréquentes (FAQ)

### <a name="what-is-the-difference-between-change-feed-and-storage-analytics-logging"></a>Quelle est la différence entre le flux de modification et la journalisation Storage Analytics ?
Les journaux d’analytique contiennent des enregistrements sur toutes les opérations de lecture, d’écriture, d’énumération et de suppression, avec une indication de réussite ou d’échec de requête pour toutes les opérations. Les journaux d’analytique sont régis par le principe du « meilleur effort », et aucun classement n’est garanti.

Le flux de modification est une solution qui fournit un journal transactionnel des mutations réussies ou des modifications apportées à votre compte, telles que la création, la modification et la suppression d’objets blob. Le flux de modification garantit l’enregistrement et l’affichage de tous les événements dans l’ordre des modifications réussies par objet blob. Vous n’avez donc pas besoin de filtrer le bruit d’un énorme volume d’opérations de lecture ou de requêtes ayant échoué. Le flux de modification est essentiellement conçu et optimisé pour le développement d’applications qui demandent certaines garanties.

### <a name="should-i-use-change-feed-or-storage-events"></a>Dois-je utiliser le flux de modification ou les événements de stockage ?
Vous pouvez tirer parti des deux fonctionnalités, car le flux de modification et les [événements Stockage Blob](storage-blob-event-overview.md) fournissent les mêmes informations avec la même garantie de fiabilité de remise, la principale différence concernant la latence, le classement et le stockage des enregistrements d’événements. Le flux de modification publie des enregistrements dans le journal dans les quelques minutes qui suivent la modification, et garantit aussi l’ordre des opérations de modification par objet blob. Les événements de stockage sont envoyés (push) en temps réel et peuvent ne pas être classés. Les événements de flux de modification sont stockés durablement dans votre compte de stockage en tant que journaux stables en lecture seule avec vos propres paramètres de conservation définis, tandis que les événements de stockage sont temporaires et consommés par le gestionnaire d’événements, sauf si vous les stockez de façon explicite. Avec le flux de modification, toutes vos applications peuvent consommer les journaux à leur convenance à l’aide des kits SDK ou des API d’objets blob. 

## <a name="next-steps"></a>Étapes suivantes

- Consultez un exemple de lecture du flux de modification à l’aide d’une application cliente .NET. Consultez [Traiter les journaux de flux de modification dans Stockage Blob Azure](storage-blob-change-feed-how-to.md).
- Découvrez comment réagir aux événements en temps réel. Consultez [Réaction aux événements Stockage Blob](storage-blob-event-overview.md)
- Apprenez-en plus sur les informations de journalisation détaillées des opérations ayant réussi et échoué pour toutes les demandes. Consultez [Journalisation Azure Storage Analytics](../common/storage-analytics-logging.md).
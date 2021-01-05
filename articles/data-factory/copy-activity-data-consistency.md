---
title: Vérification de la cohérence des données dans l’activité de copie
description: Découvrez comment activer la vérification de la cohérence des données dans l’activité de copie dans Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: ''
ms.reviewer: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 3/27/2020
ms.author: yexu
ms.openlocfilehash: 3591bfe046fa1c3e1e55aa49a0ae3ad698bc57b3
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593669"
---
#  <a name="data-consistency-verification-in-copy-activity"></a>Vérification de la cohérence des données dans l’activité de copie

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Lorsque vous déplacez des données du magasin source au magasin de destination, l’activité de copie d’Azure Data Factory vous offre la possibilité de faire une vérification supplémentaire de la cohérence des données pour vous assurer que les données sont non seulement copiées du magasin source au magasin de destination, mais également que leur cohérence entre les deux magasins de données est vérifiée. Une fois que des fichiers incohérents ont été détectés pendant le déplacement des données, vous pouvez soit abandonner l’activité de copie, soit continuer à copier le reste en activant le paramètre de tolérance de panne afin d’ignorer les fichiers incohérents. Vous pouvez récupérer les noms de fichiers ignorés en activant le paramètre de journal de session dans l’activité de copie. Pour plus d’informations, reportez-vous au [journal de session dans l’activité de copie](copy-activity-log.md).

## <a name="supported-data-stores-and-scenarios"></a>Magasins de données et scénarios pris en charge

-   La vérification de la cohérence des données est prise en charge par tous les connecteurs, à l’exception de FTP, sFTP et HTTP. 
-   La vérification de la cohérence des données n’est pas prise en charge dans le scénario de copie intermédiaire.
-   Lors de la copie de fichiers binaires, la vérification de la cohérence des données est disponible uniquement lorsque le comportement « PreserveHierarchy » est défini dans l’activité de copie.
-   Lorsque vous copiez plusieurs fichiers binaires dans le cadre d’une seule activité de copie avec vérification de la cohérence des données activée, vous avez la possibilité d’abandonner l’activité de copie ou de continuer à copier le reste en activant le paramètre de tolérance de panne afin d’ignorer les fichiers incohérents. 
-   Lors de la copie d’une table dans une activité de copie unique avec vérification de la cohérence des données activée, l’activité de copie échoue si le nombre de lignes lues depuis la source est différent du nombre de lignes copiées dans la destination plus le nombre de lignes incompatibles qui ont été ignorées.


## <a name="configuration"></a>Configuration
L’exemple suivant fournit une définition JSON pour activer la vérification de la cohérence des données dans l’activité de copie : 

```json
"typeProperties": { 
"source": { 
        "type": "BinarySource", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreReadSettings", 
            "recursive": true 
        } 
    }, 
    "sink": { 
        "type": "BinarySink", 
        "storeSettings": { 
            "type": "AzureDataLakeStoreWriteSettings" 
        } 
}, 
    "validateDataConsistency": true, 
    "skipErrorFile": { 
        "dataInconsistency": true 
    }, 
    "logSettings": {
        "enableCopyActivityLog": true,
        "copyActivityLogSettings": {
            "logLevel": "Warning",
            "enableReliableLogging": false
        },
        "logLocationSettings": {
            "linkedServiceName": {
               "referenceName": "ADLSGen2",
               "type": "LinkedServiceReference"
            },
            "path": "sessionlog/"
        }
    }
} 
```

Propriété | Description | Valeurs autorisées | Obligatoire
-------- | ----------- | -------------- | -------- 
validateDataConsistency | Si vous définissez cette propriété sur true, lors de la copie de fichiers binaires, l’activité de copie vérifie la taille de fichier, la valeur lastModifiedDate et la somme de contrôle MD5 pour chaque fichier binaire copié du magasin source au magasin de destination pour garantir la cohérence des données entre les deux magasins de données. Lors de la copie de données tabulaires, l’activité de copie vérifie le nombre total de lignes une fois le travail terminé pour s’assurer que le nombre total de lignes lues depuis la source est égal au nombre de lignes copiées dans la destination plus le nombre de lignes incompatibles qui ont été ignorées. Soyez conscient que l’activation de cette option aura une incidence sur les performances de copie.  | True<br/>False (valeur par défaut) | Non
dataInconsistency | L’une des paires clé-valeur dans le conteneur des propriétés skipErrorFile pour déterminer si vous souhaitez ignorer les fichiers incohérents. <br/> - True : vous souhaitez continuer la copie en ignorant les fichiers incohérents.<br/> - False : vous souhaitez abandonner l’activité de copie lorsque des fichiers incohérents sont détectés.<br/>Sachez que cette propriété n’est valide que lorsque vous copiez des fichiers binaires et définissez validateDataConsistency sur la valeur True.  | True<br/>False (valeur par défaut) | Non
logSettings | Groupe de propriétés qui peut être spécifié pour permettre au journal de session de consigner les fichiers ignorés. | | Non
linkedServiceName | Service lié de [Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) pour stocker les fichiers journaux de session. | Noms de services liés de type `AzureBlobStorage` ou `AzureBlobFS` faisant référence à l’instance que vous souhaitez utiliser pour stocker les fichiers journaux. | Non
path | Chemin d’accès des fichiers journaux. | Spécifiez le chemin d’accès que vous souhaitez utiliser pour stocker les fichiers journaux. Si vous ne spécifiez pas le chemin d’accès, le service crée un conteneur à votre place. | Non

>[!NOTE]
>- Lorsque vous copiez des fichiers binaires depuis ou vers Azure Blob ou Azure Data Lake Storage Gen2, ADF effectue une vérification de la somme de contrôle MD5 au niveau du bloc en tirant parti de l’[API Azure Blob](/dotnet/api/microsoft.azure.storage.blob.blobrequestoptions?view=azure-dotnet-legacy) et de l’[API Azure Data Lake Storage Gen2](/rest/api/storageservices/datalakestoragegen2/path/update#request-headers). Si des fichiers ContentMD5 se trouvent dans Azure Blob ou Azure Data Lake Storage Gen2 sous la forme de sources de données, ADF effectue également une vérification de la somme de contrôle MD5 au niveau du fichier après la lecture des fichiers. Après la copie des fichiers vers Azure Blob ou Azure Data Lake Storage Gen2 en tant que destination des données, ADF écrit des données ContentMD5 dans Azure Blob ou Azure Data Lake Storage Gen2, qui pourront être consommées plus tard par les applications en aval en vue de vérifier la cohérence des données.
>- Lorsque vous copiez des fichiers binaires d’un magasin de stockage à l’autre, ADF vérifie la taille de ces fichiers.

## <a name="monitoring"></a>Surveillance

### <a name="output-from-copy-activity"></a>Sortie de l’activité de copie
Après l’exécution de l’activité de copie terminée, vous pouvez voir le résultat de la vérification de la cohérence des données à partir de la sortie de chaque exécution d’activité de copie :

```json
"output": {
            "dataRead": 695,
            "dataWritten": 186,
            "filesRead": 3,  
            "filesWritten": 1, 
            "filesSkipped": 2, 
            "throughput": 297,
            "logFilePath": "myfolder/a84bf8d4-233f-4216-8cb5-45962831cd1b/",
            "dataConsistencyVerification": 
           { 
                "VerificationResult": "Verified", 
                "InconsistentData": "Skipped" 
           } 
        }

```
Vous pouvez voir les détails de la vérification de la cohérence des données dans la propriété « dataConsistencyVerification ».

Valeur de **VerificationResult** : 
-   **Verified** :  la cohérence de vos données copiées a été vérifiée entre les magasins source et de destination. 
-   **NotVerified** : la cohérence de vos données copiées n’a pas été vérifiée, car vous n’avez pas activé la propriété validateDataConsistency dans l’activité de copie. 
-   **Non pris en charge** : la cohérence de vos données copiées n’a pas été vérifiée, car la vérification de la cohérence des données n’est pas prise en charge pour cette paire de copies particulière. 

Valeur d’**InconsistentData** : 
-   **Found** : l’activité de copie ADF a détecté des données incohérentes. 
-   **Ignoré** : l’activité de copie ADF a détecté des données incohérentes et les a ignorées. 
-   **Aucun** : l’activité de copie ADF n’a détecté aucune donnée incohérente. Cela peut être dû au fait que la cohérence de vos données a été vérifiée entre les magasins source et de destination ou parce que vous avez désactivé la propriété validateDataConsistency dans l’activité de copie. 

### <a name="session-log-from-copy-activity"></a>Journal de session de l’activité de copie

Si vous effectuez la configuration de manière à enregistrer le fichier incohérent, vous pouvez trouver le fichier journal à l’emplacement suivant : `https://[your-blob-account].blob.core.windows.net/[path-if-configured]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv`.  Les fichiers journaux seront les fichiers CSV. 

Le schéma d’un fichier journal est le suivant :

Colonne | Description 
-------- | -----------  
Timestamp | Timestamp lorsqu’ADF ignore les fichiers incohérents.
Level | Niveau de journalisation de cet élément. Il sera au niveau « Avertissement » pour l’élément indiquant que le fichier est ignoré.
NomOpération | Comportement opérationnel de l’activité de copie Azure Data Factory sur chaque fichier. Le fichier à ignorer sera indiqué par la mention « FileSkip ».
OperationItem | Nom du fichier à ignorer.
Message | Plus d’informations pour illustrer la raison pour laquelle les fichiers sont ignorés.

Voici un exemple de fichier journal : 
```
Timestamp, Level, OperationName, OperationItem, Message
2020-02-26 06:22:56.3190846, Warning, FileSkip, "sample1.csv", "File is skipped after read 548000000 bytes: ErrorCode=DataConsistencySourceDataChanged,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Source file 'sample1.csv' is changed by other clients during the copy activity run.,Source=,'." 
```
Dans le fichier journal ci-dessus, vous pouvez voir que sample1.csv a été ignoré, car sa cohérence n’a pas été vérifiée entre les magasins source et de destination. Vous pouvez voir plus en détails que l’incohérence de sample1.csv est due au fait que le fichier a été modifié par d’autres applications alors que l’activité de copie ADF était en cours. 



## <a name="next-steps"></a>Étapes suivantes
Voir les autres articles relatifs à l’activité de copie :

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Tolérance de panne de l’activité de copie](copy-activity-fault-tolerance.md)
---
title: Journal de session dans l’activité de copie
description: Découvrez comment activer le journal de session dans l’activité de copie d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: dearandyxu
manager: ''
ms.reviewer: ''
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 11/11/2020
ms.author: yexu
ms.openlocfilehash: e56a840da07a2f6e966867699506f0122a0e7956
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593640"
---
#  <a name="session-log-in-copy-activity"></a>Journal de session dans l’activité de copie

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Vous pouvez journaliser les noms de fichiers copiés lors de l’activité de copie, ce qui peut vous aider à mieux vous assurer que les données sont non seulement bien copiées du magasin source vers le magasin de destination, mais qu’elles sont également cohérentes entre les deux magasins en examinant les fichiers copiés dans les journaux de session de l’activité de copie.  

Quand vous activez le paramètre de tolérance de panne dans l’activité de copie de façon à ignorer les données défectueuses, les fichiers ignorés et les lignes ignorées peuvent aussi être journalisés.  Pour obtenir plus de détails, consultez [Tolérance de panne de l’activité de copie](copy-activity-fault-tolerance.md). 

## <a name="configuration"></a>Configuration
L’exemple suivant fournit une définition JSON pour activer le journal de session dans l’activité de copie : 

```json
"typeProperties": {
    "source": {
        "type": "BinarySource",
        "storeSettings": {
            "type": "AzureDataLakeStoreReadSettings",
            "recursive": true
        },
        "formatSettings": {
            "type": "BinaryReadSettings"
        }
    },
    "sink": {
        "type": "BinarySink",
        "storeSettings": {
            "type": "AzureBlobFSWriteSettings"
        }
    },                    
    "skipErrorFile": {
        "fileForbidden": true,
        "dataInconsistency": true
    },
    "validateDataConsistency": true,
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
enableCopyActivityLog | Quand cette propriété est définie sur true, vous pouvez journaliser les fichiers copiés, les fichiers ignorés ou les lignes ignorées.  | True<br/>False (valeur par défaut) | Non
logLevel | Avec la valeur « Info », l’ensemble des fichiers copiés, des fichiers ignorés et des lignes ignorées sont journalisés. Avec la valeur « Warning », seuls les fichiers ignorés et les lignes ignorés sont journalisés.  | Informations<br/>Avertissement (par défaut) | Non
enableReliableLogging | Quand cette propriété est définie sur true, l’activité de copie en mode « reliable » (fiable) vide les journaux une fois que chaque fichier est copié dans la destination.  Lorsque vous copiez de grandes quantités de fichiers et que le mode de journalisation fiable est activé dans l’activité de copie, vous devez vous attendre à ce que le débit de copie en soit affecté, car la copie de chaque fichier est une opération de double écriture. Une demande est envoyée au magasin de destination et une autre au magasin de stockage des journaux.  Lorsqu’elle est en mode « best effort » (meilleur effort), l’activité de copie vide les journaux par lots d’enregistrements au cours d’une période, et le débit de copie s’en trouve beaucoup moins affecté. En revanche, le caractère exhaustif et opportun de la journalisation n’est pas garanti dans ce mode, car il est possible que le dernier lot d’événements de journal n’ait pas été vidé dans le fichier journal si le l’activité de copie a échoué. Dans ce cas, vous verrez que certains fichiers copiés dans la destination ne sont pas journalisés.  | True<br/>False (valeur par défaut) | Non
logLocationSettings | Groupe de propriétés qui peuvent être utilisées pour spécifier l’emplacement de stockage des journaux de session. | | Non
linkedServiceName | Service lié de [Stockage Blob Azure](connector-azure-blob-storage.md#linked-service-properties) ou [Azure Data Lake Storage Gen2](connector-azure-data-lake-storage.md#linked-service-properties) pour stocker les fichiers journaux de session. | Noms de services liés de type `AzureBlobStorage` ou `AzureBlobFS` faisant référence à l’instance que vous souhaitez utiliser pour stocker les fichiers journaux. | Non
path | Chemin d’accès des fichiers journaux. | Spécifiez le chemin d’accès que vous souhaitez utiliser pour stocker les fichiers journaux. Si vous ne spécifiez pas le chemin d’accès, le service crée un conteneur à votre place. | Non


## <a name="monitoring"></a>Surveillance

### <a name="output-from-copy-activity"></a>Sortie de l’activité de copie
Une fois l’exécution de l’activité de copie terminée, vous pouvez voir le chemin des fichiers journaux dans la sortie de chaque exécution de l’activité de copie. Vous pouvez trouver les fichiers journaux en suivant ce chemin : `https://[your-blob-account].blob.core.windows.net/[logFilePath]/copyactivity-logs/[copy-activity-name]/[copy-activity-run-id]/[auto-generated-GUID].csv`.  Les fichiers journaux seront les fichiers CSV. 

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

### <a name="the-schema-of-the-log-file"></a>Schéma du fichier journal

Le schéma d’un fichier journal est le suivant.

Colonne | Description 
-------- | -----------  
Timestamp | Timestamp correspondant au moment où ADF lit, écrit ou ignore l’objet.
Level | Niveau de journalisation de cet élément. Ses valeurs possibles sont « Warning » ou « Info ».
NomOpération | Comportement opérationnel de l’activité de copie d’ADF sur chaque objet. Les valeurs possibles sont « FileRead », « FileWrite », « FileSkip » ou « TabularRowSkip ».
OperationItem | Noms de fichiers ou lignes ignorées.
Message | Informations complémentaire pour indiquer si le fichier a été lu à partir du magasin source ou écrit dans le magasin de destination. Il peut aussi indiquer la raison pour laquelle le fichier ou les lignes ont été ignorés.

Voici un exemple de fichier journal. 
```
Timestamp, Level, OperationName, OperationItem, Message
2020-10-19 08:39:13.6688152,Info,FileRead,"sample1.csv","Start to read file: {""Path"":""sample1.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:39:56.3190846, Warning, FileSkip, "sample1.csv", "File is skipped after read 548000000 bytes: ErrorCode=DataConsistencySourceDataChanged,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Source file 'sample1.csv' is changed by other clients during the copy activity run.,Source=,'." 
2020-10-19 08:40:13.6688152,Info,FileRead,"sample2.csv","Start to read file: {""Path"":""sample2.csv"",""ItemType"":""File"",""Size"":104857620,""LastModified"":""2020-10-19T08:22:31Z"",""ETag"":""\""0x8D874081F80C01A\"""",""ContentMD5"":""dGKVP8BVIy6AoTtKnt+aYQ=="",""ObjectName"":null}"
2020-10-19 08:40:13.9003981,Info,FileWrite,"sample2.csv","Start to write file from source file: sample2.csv."
2020-10-19 08:45:17.6508407,Info,FileRead,"sample2.csv","Complete reading file successfully. "
2020-10-19 08:45:28.7390083,Info,FileWrite,"sample2.csv","Complete writing file from source file: sample2.csv. File is successfully copied."
```
Dans le fichier journal ci-dessus, vous pouvez voir que sample1.csv a été ignoré, car sa cohérence n’a pas été vérifiée entre les magasins source et de destination. Vous pouvez voir plus en détails que l’incohérence de sample1.csv est due au fait que le fichier a été modifié par d’autres applications alors que l’activité de copie ADF était en cours. Vous pouvez aussi constater que sample2.csv bien a été copié du magasin source vers le magasin de destination.

Vous pouvez utiliser plusieurs moteurs d’analyse pour approfondir l’analyse des fichiers journaux.  Vous trouverez ci-dessous quelques exemples d’utilisation d’une requête SQL en vue d’analyser le fichier journal en important un fichier journal csv dans la base de données SQL où le nom de la table peut être SessionLogDemo.  

-   Demande de la liste des fichiers copiés. 
```sql
select OperationItem from SessionLogDemo where Message like '%File is successfully copied%'
```

-   Demande de la liste des fichiers copiés au cours d’un intervalle de temps déterminé. 
```sql
select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%'
```

-   Demande d’un fichier déterminé avec l’heure à laquelle il a été copié et ses métadonnées. 
```sql
select * from SessionLogDemo where OperationItem='<file name>'
```

-   Demande d’une liste de fichiers dont les métadonnées ont été copiées au cours d’un intervalle de temps. 
```sql
select * from SessionLogDemo where OperationName='FileRead' and Message like 'Start to read%' and OperationItem in (select OperationItem from SessionLogDemo where TIMESTAMP >= '<start time>' and TIMESTAMP <= '<end time>' and Message like '%File is successfully copied%')
```

-   Demande de la liste des fichiers ignorés. 
```sql
select OperationItem from SessionLogDemo where OperationName='FileSkip'
```

-   Demande de la raison pour laquelle un fichier déterminé a été ignoré. 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip'
```

-   Demande de la liste des fichiers ignorés pour la même raison : « fichier blob inexistant ». 
```sql
select TIMESTAMP, OperationItem, Message from SessionLogDemo where OperationName='FileSkip' and Message like '%UserErrorSourceBlobNotExist%'
```

-   Demande du nom du fichier dont la copie prend le plus de temps.
```sql
select top 1 OperationItem, CopyDuration=DATEDIFF(SECOND, min(TIMESTAMP), max(TIMESTAMP)) from SessionLogDemo group by OperationItem order by CopyDuration desc
```


## <a name="next-steps"></a>Étapes suivantes
Voir les autres articles relatifs à l’activité de copie :

- [Vue d’ensemble des activités de copie](copy-activity-overview.md)
- [Tolérance de panne de l’activité de copie](copy-activity-fault-tolerance.md)
- [Cohérence des données dans l’activité de copie](copy-activity-data-consistency.md)
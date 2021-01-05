---
title: Créer une tâche de diffusion en continu T-SQL dans Azure SQL Edge
description: Apprenez-en davantage sur la création de tâches Stream Analytics dans Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 97189fd7a232c2467981b23dc20da51ebef08252
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656340"
---
# <a name="create-a-data-streaming-job-in-azure-sql-edge"></a>Créer une tâche de diffusion en continu de données dans Azure SQL Edge 

Cet article explique comment créer une tâche de diffusion en continu T-SQL dans Azure SQL Edge. Vous créez les objets d’entrée et de sortie du flux externe, puis vous définissez la requête de travail de streaming dans le cadre de la création de la tâche de streaming.

## <a name="configure-the-external-stream-input-and-output-objects"></a>Configurer les objets d’entrée et de sortie du flux externe

T-SQL Streaming utilise la fonctionnalité de source de données externe de SQL Server pour définir les sources de données associées aux entrées et sorties du flux externe de la tâche de streaming. Utilisez les commandes T-SQL suivantes pour créer un objet de sortie ou d’entrée de flux externe :

- [CREATE EXTERNAL FILE FORMAT (Transact-SQL)](/sql/t-sql/statements/create-external-file-format-transact-sql)

- [CREATE EXTERNAL DATA SOURCE (Transact-SQL)](/sql/t-sql/statements/create-external-data-source-transact-sql)

- [CREATE EXTERNAL STREAM (Transact-SQL)](#example-create-an-external-stream-object-to-azure-sql-database)

En outre, si Azure SQL Edge, SQL Server ou Azure SQL Database est utilisé comme flux de sortie, vous avez besoin de [CREATE DATABASE SCOPED CREDENTIAL (Transact-SQL)](/sql/t-sql/statements/create-database-scoped-credential-transact-sql). Cette commande T-SQL définit les informations d’identification pour accéder à la base de données.

### <a name="supported-input-and-output-stream-data-sources"></a>Sources de données de flux d’entrée et de sortie prises en charge

Actuellement, Azure SQL Edge prend uniquement en charge les sources de données suivantes en tant qu’entrées et sorties de flux.

| Type de source de données | Entrée | Output | Description |
|------------------|-------|--------|------------------|
| Hub Azure IoT Edge | O | O | Source de données pour lire et écrire des données de streaming dans un hub Azure IoT Edge. Pour plus d’informations, consultez l’article [IoT Edge Hub](../iot-edge/iot-edge-runtime.md#iot-edge-hub).|
| SQL Database | N | O | Connexion à la source de données pour écrire les données de streaming dans SQL Database. La base de données peut être une base de données locale dans Azure SQL Edge, ou une base de données distante dans SQL Server ou Azure SQL Database.|
| Kafka | O | N | Source de données pour lire les données de streaming à partir d’une rubrique Kafka. Cet adaptateur n’est actuellement disponible que pour les versions Intel ou AMD d’Azure SQL Edge. Il n’est pas disponible pour la version ARM64 d’Azure SQL Edge.|

### <a name="example-create-an-external-stream-inputoutput-object-for-azure-iot-edge-hub"></a>Exemple : Créer un objet d’entrée/de sortie de flux externe pour le hub Azure IoT Edge

L’exemple suivant crée un objet de flux externe pour Azure IoT Edge Hub. Pour créer une source de données d’entrée/de sortie de flux externe pour le hub Azure IoT Edge, vous devez d’abord créer un format de fichier externe pour la disposition des données en lecture/écriture.

1. Créez un format de fichier externe de type JSON.

    ```sql
    Create External file format InputFileFormat
    WITH 
    (  
       format_type = JSON,
    )
    go
    ```

2. Créez une source de données externe pour le hub Azure IoT Edge. Le script T-SQL suivant crée une connexion de source de données à un hub IoT Edge qui s’exécute sur le même hôte Docker qu’Azure SQL Edge.

    ```sql
    CREATE EXTERNAL DATA SOURCE EdgeHubInput 
    WITH 
    (
        LOCATION = 'edgehub://'
    )
    go
    ```

3. Créez l’objet de flux externe pour le hub Azure IoT Edge. Le script T-SQL suivant crée un objet de flux pour le hub IoT Edge. Dans le cas d’un objet de flux de hub IoT Edge, le paramètre LOCATION correspond au nom de la rubrique/du canal de hub IoT Edge en cours de lecture ou d’écriture.

    ```sql
    CREATE EXTERNAL STREAM MyTempSensors 
    WITH 
    (
        DATA_SOURCE = EdgeHubInput,
        FILE_FORMAT = InputFileFormat,
        LOCATION = N'TemperatureSensors',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    go
    ```

### <a name="example-create-an-external-stream-object-to-azure-sql-database"></a>Exemple : Créer un objet de flux externe vers Azure SQL Database

L’exemple suivant crée un objet de flux externe dans la base de données locale dans Azure SQL Edge. 

1. Créez une clé principale sur la base de données. C’est nécessaire pour chiffrer le secret des informations d’identification.

    ```sql
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '<<Strong_Password_For_Master_Key_Encryption>>';
    ```

2. Créez des informations d’identification incluses dans l’étendue de la base de données pour accéder à la source SQL Server. L’exemple suivant crée des informations d’identification pour la source de données externe avec IDENTITY = username et SECRET = password.

    ```sql
    CREATE DATABASE SCOPED CREDENTIAL SQLCredential
    WITH IDENTITY = '<SQL_Login>', SECRET = '<SQL_Login_PASSWORD>'
    go
    ```

3. Créez une source de données externe avec CREATE EXTERNAL DATA SOURCE. L’exemple suivant :

    * Crée une source de données externe pour nommée *LocalSQLOutput*.
    * Identifie la source de données externe (LOCATION = '<vendor>://<server>[:<port>]'). Dans l’exemple, elle pointe vers une instance locale d’Azure SQL Edge.
    * Elle utilise les informations d’identification créées précédemment.

    ```sql
    CREATE EXTERNAL DATA SOURCE LocalSQLOutput 
    WITH 
    (
        LOCATION = 'sqlserver://tcp:.,1433',
        CREDENTIAL = SQLCredential
    )
    go
    ```

4. Créez l’objet de flux externe. L’exemple suivant crée un objet de flux externe pointant vers une table *dbo.TemperatureMeasurements* dans la base de données *MySQLDatabase*.

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurements 
    WITH 
    (
        DATA_SOURCE = LocalSQLOutput,
        LOCATION = N'MySQLDatabase.dbo.TemperatureMeasurements',
        INPUT_OPTIONS = N'',
        OUTPUT_OPTIONS = N''
    );
    ```

### <a name="example-create-an-external-stream-object-for-kafka"></a>Exemple : Créer un objet de flux externe pour Kafka

L’exemple suivant crée un objet de flux externe dans la base de données locale dans Azure SQL Edge. Cet exemple suppose que le serveur Kafka est configuré pour l’accès anonyme. 

1. Créez une source de données externe avec CREATE EXTERNAL DATA SOURCE. L’exemple suivant :

    ```sql
    Create EXTERNAL DATA SOURCE [KafkaInput] 
    With
    (
        LOCATION = N'kafka://<kafka_bootstrap_server_name_ip>:<port_number>'
    )
    GO
    ```
2. Créer un format de fichier externe pour l’entrée Kafka. Dans l’exemple suivant, un format de fichier JSON a été créé avec la compression GZipped. 

   ```sql
   CREATE EXTERNAL FILE FORMAT JsonGzipped  
    WITH 
    (  
        FORMAT_TYPE = JSON , 
        DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec' 
    )
   ```

3. Créez l’objet de flux externe. L’exemple suivant crée un objet de flux externe pointant vers la rubrique Kafka `*TemperatureMeasurement*`.

    ```sql
    CREATE EXTERNAL STREAM TemperatureMeasurement 
    WITH 
    (  
        DATA_SOURCE = KafkaInput, 
        FILE_FORMAT = JsonGzipped,
        LOCATION = 'TemperatureMeasurement',
        INPUT_OPTIONS = 'PARTITIONS: 10' 
    ); 
    ```

## <a name="create-the-streaming-job-and-the-streaming-queries"></a>Créer la tâche et les requêtes de streaming

Utilisez la procédure stockée système `sys.sp_create_streaming_job` pour définir les requêtes de streaming et créer la tâche de streaming. La procédure stockée `sp_create_streaming_job` prend les paramètres suivants :

- `job_name`: Nom de la tâche de streaming. Les noms des tâches de streaming sont uniques sur l’ensemble de l’instance.
- `statement`: Instructions de requête de streaming basées sur le [langage de requête Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference).

L’exemple suivant crée une tâche de streaming simple avec une requête de streaming. Cette requête lit les entrées du hub IoT Edge et les écrit dans `dbo.TemperatureMeasurements` dans la base de données.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob1',
@statement= N'Select * INTO TemperatureMeasurements from MyEdgeHubInput'
```

L’exemple suivant crée une tâche de streaming plus complexe avec plusieurs requêtes différentes. Ces requêtes en incluent une qui utilise la fonction intégrée `AnomalyDetection_ChangePoint` pour identifier les anomalies dans les données de température.

```sql
EXEC sys.sp_create_streaming_job @name=N'StreamingJob2', @statement=
N' Select * INTO TemperatureMeasurements1 from MyEdgeHubInput1

Select * Into TemperatureMeasurements2 from MyEdgeHubInput2

Select * Into TemperatureMeasurements3 from MyEdgeHubInput3

SELECT
Timestamp as [Time],
[Temperature] As [Temperature],
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''Score'') as ChangePointScore,
GetRecordPropertyValue(AnomalyDetection_ChangePoint(Temperature, 80, 1200) OVER(LIMIT DURATION(minute, 20)), ''IsAnomaly'') as IsChangePointAnomaly
INTO TemperatureAnomalies FROM MyEdgeHubInput2;
'
go
```

## <a name="start-stop-drop-and-monitor-streaming-jobs"></a>Démarrer, arrêter, supprimer et surveiller des tâches de streaming

Pour démarrer une tâche de streaming dans Azure SQL Edge, exécutez la procédure stockée `sys.sp_start_streaming_job`. La procédure stockée requiert le nom de l’élément de tâche de streaming à démarrer, en tant qu'entrée.

```sql
exec sys.sp_start_streaming_job @name=N'StreamingJob1'
go
```

Pour arrêter une tâche de streaming, exécutez la procédure stockée `sys.sp_stop_streaming_job`. La procédure stockée requiert le nom de l’élément de tâche de streaming à arrêter, en tant qu'entrée.

```sql
exec sys.sp_stop_streaming_job @name=N'StreamingJob1'
go
```

Pour abandonner (ou supprimer) une tâche de streaming, exécutez la procédure stockée `sys.sp_drop_streaming_job`. La procédure stockée requiert le nom de l’élément de tâche de streaming à supprimer, en tant qu'entrée.

```sql
exec sys.sp_drop_streaming_job @name=N'StreamingJob1'
go
```

Pour obtenir l’état actuel d’une tâche de streaming, exécutez la procédure stockée `sys.sp_get_streaming_job`. La procédure stockée requiert le nom de l’élément de tâche de streaming à supprimer, en tant qu'entrée. Elle renvoie le nom et l’état actuel de la tâche de streaming.

```sql
exec sys.sp_get_streaming_job @name=N'StreamingJob1'
        WITH RESULT SETS
(
       (
       name nvarchar(256),
       status nvarchar(256),
       error nvarchar(256)
       )
)
```

La tâche de streaming peut se présenter dans l’un des états suivants :

| Statut | Description |
|--------| ------------|
| Date de création | La tâche de streaming a été créée, mais n’a pas encore été démarrée. |
| Démarrage en cours | La tâche de streaming est cours de démarrage. |
| Idle | La tâche de streaming est en cours d’exécution, mais il n’y a aucune entrée à traiter. |
| Traitement en cours | La tâche de streaming est en cours d’exécution et traite les entrées. Cet état indique un état sain de la tâche de streaming. |
| Détérioré | La tâche de streaming est en cours d’exécution, mais des erreurs non récupérables se sont produites pendant le traitement des entrées. La tâche d’entrée continuera de s’exécuter, mais supprimera les entrées rencontrant des erreurs. |
| Arrêté | La tâche de streaming a été arrêtée. |
| Échec | La tâche de streaming a échoué. Cela indique généralement une erreur irrécupérable lors du traitement. |

## <a name="next-steps"></a>Étapes suivantes

- [Afficher les métadonnées associées aux tâches de streaming dans Azure SQL Edge](streaming-catalog-views.md) 
- [Créer un flux externe](create-external-stream-transact-sql.md)
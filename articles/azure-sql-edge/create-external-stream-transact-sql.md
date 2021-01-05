---
title: CREATE EXTERNAL STREAM (Transact-SQL) - Azure SQL Edge
description: Découvrez l’instruction CREATE EXTERNAL STREAM dans Azure SQL Edge
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 07/27/2020
ms.openlocfilehash: 92658584030fa83da067eceab391d9bba2f034c0
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93392297"
---
# <a name="create-external-stream-transact-sql"></a>CREATE EXTERNAL STREAM (Transact-SQL)

L’objet EXTERNAL STREAM (flux externe) fait office à la fois de flux d’entrée et de flux de sortie. Il peut être utilisé comme entrée pour interroger les données de streaming auprès de services d’ingestion d’événements comme Azure Event, Azure IoT Hub (ou Edge Hub) ou Kafka, ou il peut être utilisé comme sortie pour spécifier où et comment stocker les résultats d’une requête de streaming.

Un FLUX EXTERNE peut également être spécifié et créé en tant que sortie et entrée pour des services tels que Event Hub ou Stockage Blob. Cela facilite les scénarios de chaînage où une requête de streaming conserve les résultats dans le flux externe en tant que sortie et une autre requête de streaming lit le même flux externe en tant qu'entrée.

Actuellement, Azure SQL Edge prend uniquement en charge les sources de données suivantes en tant qu’entrées et sorties de flux.

| Type de source de données | Entrée | Output | Description |
|------------------|-------|--------|------------------|
| Hub Azure IoT Edge | O | O | Source de données pour lire et écrire des données de streaming dans un hub Azure IoT Edge. Pour plus d’informations, consultez l’article [IoT Edge Hub](../iot-edge/iot-edge-runtime.md#iot-edge-hub).|
| SQL Database | N | O | Connexion à la source de données pour écrire les données de streaming dans SQL Database. La base de données peut être une base de données locale dans Azure SQL Edge, ou une base de données distante dans SQL Server ou Azure SQL Database.|
| Kafka | O | N | Source de données pour lire les données de streaming à partir d’une rubrique Kafka. La prise en charge de Kafka n’est pas disponible pour la version ARM64 d’Azure SQL Edge.|



## <a name="syntax"></a>Syntaxe

```sql
CREATE EXTERNAL STREAM {external_stream_name}  
( <column_definition> [, <column_definition> ] * ) -- Used for Inputs - optional 
WITH  ( <with_options> )

<column_definition> ::=
  column_name <column_data_type>

<data_type> ::=
[ type_schema_name . ] type_name
    [ ( precision [ , scale ] | max ) ]

<with_options> ::=
  DATA_SOURCE = data_source_name, 
  LOCATION = location_name, 
  [FILE_FORMAT = external_file_format_name], --Used for Inputs - optional 
  [<optional_input_options>],
  [<optional_output_options>], 
  TAGS = <tag_column_value>

<optional_input_options> ::= 
  INPUT_OPTIONS = '[<Input_options_data>]'

<Input_option_data> ::= 
      <input_option_values> [ , <input_option_values> ]

<input_option_values> ::=
  PARTITIONS: [number_of_partitions]
  | CONSUMER_GROUP: [ consumer_group_name ] 
  | TIME_POLICY: [ time_policy ] 
  | LATE_EVENT_TOLERANCE: [ late_event_tolerance_value ] 
  | OUT_OF_ORDER_EVENT_TOLERANCE: [ out_of_order_tolerance_value ]
  
<optional_output_options> ::= 
  OUTPUT_OPTIONS = '[<output_option_data>]'

<output_option_data> ::= 
      <output_option_values> [ , <output_option_values> ]

<output_option_values> ::=
   REJECT_POLICY: [ reject_policy ] 
   | MINIMUM_ROWS: [ row_value ] 
   | MAXIMUM_TIME: [ time_value_minutes] 
   | PARTITION_KEY_COLUMN: [ partition_key_column_name ] 
   | PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | SYSTEM_PROPERTY_COLUMNS: [ ( [ output_col_name ] ) ] 
   | PARTITION_KEY: [ partition_key_name ] 
   | ROW_KEY: [ row_key_name ] 
   | BATCH_SIZE: [ batch_size_value ] 
   | MAXIMUM_BATCH_COUNT: [ batch_value ] 
   | STAGING_AREA: [ blob_data_source ]
 
<tag_column_value> ::= -- Reserved for Future Usage
); 
```

## <a name="arguments"></a>Arguments

- [DATA_SOURCE](/sql/t-sql/statements/create-external-data-source-transact-sql/)
- [FILE_FORMAT](/sql/t-sql/statements/create-external-file-format-transact-sql/)
- **LOCATION**  : Spécifie le nom des données ou de l’emplacement réels dans la source de données. 
   - Pour les objets de flux Edge Hub ou Kafka, l’emplacement indique le nom de la rubrique Edge Hub ou Kafka où lire et écrire.
   - Pour les objets de flux SQL (SQL Server, Azure SQL Database ou Azure SQL Edge), l’emplacement indique le nom de la table. Si le flux est créé dans la même base de données et le même schéma que la table de destination, seul le nom de la table suffit. Dans le cas contraire, vous devez indiquer le nom entier de la table (<database_name.schema_name.table_name).
   - Pour un objet de flux Stockage Blob Azure, l’emplacement fait référence au modèle de chemin à utiliser dans le conteneur d’objets blob. Pour plus d’informations sur cette fonctionnalité, consultez (/articles/stream-analytics/stream-analytics-define-outputs.md#blob-storage-and-azure-data-lake-gen2)

- **INPUT_OPTIONS**  : Spécifie les options en tant que paires clé-valeur pour des services comme Kafka, IoT Edge Hub qui font office d’entrées pour les requêtes de streaming
    - PARTITIONS : Nombre de partitions définies pour une rubrique. Le nombre maximal de partitions utilisables est limité à 32.
      - S’applique aux flux d’entrée Kafka
    - CONSUMER_GROUP : Event Hub et IoT Hub limitent le nombre de lecteurs au sein d'un groupe de consommateurs (à 5). Laissez ce champ vide pour utiliser le groupe de consommateurs « $Default ».
      - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.  
    - TIME_POLICY : Indique s’il convient de supprimer des événements ou d'ajuster l’heure des événements lorsque des événements tardifs ou en désordre dépassent leur valeur de tolérance.
      - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
    - LATE_EVENT_TOLERANCE : Délai maximal acceptable. Ce délai correspond à la différence entre le timestamp de l’événement et l’horloge système.
      - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
    - OUT_OF_ORDER_EVENT_TOLERANCE : Des événements peuvent arriver en désordre après avoir transité de l'entrée vers la requête de streaming. Vous pouvez accepter ces événements tels quels ou choisir de les suspendre pendant une période définie afin de les mettre dans l'ordre souhaité.
      - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
        
- **OUTPUT_OPTIONS**  : Spécifie les options en tant que paires clé-valeur pour les services pris en charge correspondant à des entrées de requêtes de streaming 
  - REJECT_POLICY :  DROP | RETRY Spécifie les stratégies en matière de gestion des erreurs de données lorsque des erreurs de conversion de données se produisent. 
    - S’applique à toutes les sorties prises en charge. 
  - MINIMUM_ROWS :  
    Nombre minimal de lignes requises par lot écrit dans une sortie. Pour Parquet, chaque lot crée un nouveau fichier. 
    - S’applique à toutes les sorties prises en charge. 
  - MAXIMUM_TIME :  
    Délai d’attente maximal par lot en minutes. À l’issue de cette période, le lot est écrit dans la sortie même si l’exigence de lignes minimum n’est pas remplie. 
    - S’applique à toutes les sorties prises en charge. 
  - PARTITION_KEY_COLUMN :  
    Colonne utilisée pour la clé de partition. 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
  - PROPERTY_COLUMNS :  
    Liste de noms de colonne de sortie séparés par des virgules à attacher aux messages en tant que propriétés personnalisées, le cas échéant  
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge. 
  - SYSTEM_PROPERTY_COLUMNS :  
    Collection JSON de paires nom/valeur de noms de propriétés système et de colonnes de sortie à remplir sur les messages Service Bus. Par exemple, {« MessageId » : « Column1 », « PartitionKey » : « Column2 »} 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge. 
  - PARTITION_KEY :  
    Nom de la colonne de sortie contenant la clé de partition. La clé de partition est un identificateur unique pour la partition dans une table donnée qui constitue la première partie de la clé primaire d’une entité. C’est une valeur de chaîne qui peut atteindre 1 Ko. 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
  - ROW_KEY :  
    Nom de la colonne de sortie contenant la clé de ligne. La clé de ligne est un identificateur unique pour une entité dans une partition donnée. Elle constitue la deuxième partie de la clé primaire d'une entité. La clé de ligne est une valeur de chaîne qui peut atteindre 1 Ko. 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.
  - BATCH_SIZE :  
    Correspond au nombre de transactions pour le stockage de table avec un maximum de 100 enregistrements. Pour Azure Functions, il s’agit de la taille du lot en octets envoyé à la fonction par appel, la valeur (256 Ko par défaut). 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge. 
  - MAXIMUM_BATCH_COUNT :  
    Nombre maximal d'événements envoyés à la fonction par appel pour la fonction Azure (100 par défaut). Pour SQL Database, cela représente le nombre maximal d'enregistrements envoyés avec chaque transaction d'insertion en bloc (10 000 par défaut). 
    - S’applique à toutes les sorties basées sur SQL 
  - STAGING_AREA : Objet EXTERNAL DATA SOURCE vers Stockage Blob Zone de transit pour l’ingestion de données à haut débit dans Azure Synapse Analytics 
    - Réservé pour un usage ultérieur. Ne s’applique pas à Azure SQL Edge.


## <a name="examples"></a>Exemples

### <a name="example-1---edgehub"></a>Exemple 1 - EdgeHub

Tapez : Entrée ou Sortie<br>

Syntaxe :

```sql
CREATE EXTERNAL DATA SOURCE MyEdgeHub 
WITH  
(      
  LOCATION = 'edgehub://'       
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat  
WITH (  
   FORMAT_TYPE = 'JSON', 
); 
 
CREATE EXTERNAL STREAM Stream_A  
WITH    
(   
   DATA_SOURCE = MyEdgeHub, 
   FILE_FORMAT = myFileFormat, 
   LOCATION = '<mytopicname>', 
   OUTPUT_OPTIONS =   
     'REJECT_TYPE: Drop'
);
```

### <a name="example-2---azure-sql-database-azure-sql-edge-sql-server"></a>Exemple 2 - Azure SQL Database, Azure SQL Edge, SQL Server

Tapez : Output<br>

Syntaxe :

```sql
CREATE DATABASE SCOPED CREDENTIAL SQLCredName 
WITH IDENTITY = '<user>', 
SECRET = '<password>'; 
 
-- Azure SQL Database 
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = '<my_server_name>.database.windows.net', 
  CREDENTIAL = SQLCredName 
); 
 
--SQL Server or Azure SQL Edge
CREATE EXTERNAL DATA SOURCE MyTargetSQLTabl 
WITH 
(     
  LOCATION = ' <sqlserver://<ipaddress>,<port>', 
  CREDENTIAL = SQLCredName 
); 

CREATE EXTERNAL STREAM Stream_A 
WITH   
(  
    DATA_SOURCE = MyTargetSQLTable, 
    LOCATION = '<DatabaseName>.<SchemaName>.<TableName>' ,
   --Note: If table is contained in the database, <TableName> should be sufficient 
    OUTPUT_OPTIONS =  
      'REJECT_TYPE: Drop'
); 
```

### <a name="example-3---kafka"></a>Exemple 3 - Kafka

Tapez : Entrée<br>

Syntaxe :
```sql
CREATE EXTERNAL DATA SOURCE MyKafka_tweets 
WITH 
( 
  --The location maps to KafkaBootstrapServer 
  LOCATION = 'kafka://<kafkaserver>:<ipaddress>', 
  CREDENTIAL = kafkaCredName 
); 
 
CREATE EXTERNAL FILE FORMAT myFileFormat 
WITH ( 
    FORMAT_TYPE = JSON, 
    DATA_COMPRESSION = 'org.apache.hadoop.io.compress.GzipCodec'
); 

CREATE EXTERNAL STREAM Stream_A (user_id VARCHAR, tweet VARCHAR) 
WITH   
(  
    DATA_SOURCE = MyKafka_tweets, 
    LOCATION = '<KafkaTopicName>', 
    FILE_FORMAT = myFileFormat,  
    INPUT_OPTIONS =  
      'PARTITIONS: 5'
); 
```

## <a name="see-also"></a>Voir aussi

- [DROP EXTERNAL STREAM (Transact-SQL)](drop-external-stream-transact-sql.md)
---
title: Déplacer des données à partir d’une source HTTP – Azure
description: Découvrez comment déplacer des données à partir d’une source HTTP locale ou cloud à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 05/22/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: 653a53d6bb5c69cd95fd5e9a2483b51de8293b40
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97608576"
---
# <a name="move-data-from-an-http-source-by-using-azure-data-factory"></a>Déplacer des données à partir d’une source HTTP à l’aide d’Azure Data Factory

> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-http-connector.md)
> * [Version 2 (version actuelle)](../connector-http.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Azure Data Factory, consultez [Connecteur HTTP dans V2](../connector-http.md).

Cet article explique comment utiliser l’activité de copie dans Azure Data Factory pour déplacer des données d’un point de terminaison HTTP local ou cloud vers une banque de données réceptrice prise en charge. Il s’appuie sur l’article [Déplacer des données à l’aide de l’activité de copie](data-factory-data-movement-activities.md), qui présente une vue d’ensemble du déplacement des données avec l’activité de copie. L’article répertorie également les banques de données prises en charge en tant que sources et récepteurs par l’activité de copie.

Actuellement, Data Factory prend uniquement en charge le déplacement de données d’une source HTTP vers d’autres banques de données. Il ne prend pas en charge le déplacement de données à partir d’autres banques de données vers une destination HTTP.

## <a name="supported-scenarios-and-authentication-types"></a>Scénarios et types d’authentification pris en charge

Vous pouvez utiliser ce connecteur HTTP pour récupérer des données d’un *point de terminaison HTTP/S cloud et local* à l’aide des méthodes HTTP **GET** ou **POST**. Les types d’authentification suivants sont pris en charge : **Anonyme**, **De base**, **Digest**, **Windows** et **ClientCertificate**. Notez la différence entre ce connecteur et le [connecteur de table web](data-factory-web-table-connector.md). Le connecteur de table web extrait le contenu de tables d’une page web HTML.

Pour copier des données à partir d’un point de terminaison HTTP local, vous devez installer une passerelle de gestion des données dans l’environnement local/sur la machine virtuelle Azure. Pour en savoir plus sur la passerelle de gestion des données et obtenir des instructions détaillées sur la configuration de la passerelle, consultez l’article [Déplacement de données entre des emplacements locaux et le cloud](data-factory-move-data-between-onprem-and-cloud.md).

## <a name="get-started"></a>Bien démarrer

Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’une source HTTP à l’aide de différents outils ou API :

- Le moyen le plus simple de créer un pipeline consiste à utiliser l’Assistant Copier des données. Pour une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données, consultez la page [Tutoriel : Créez un pipeline à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md).

- Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, un **modèle Azure Resource Manager**, l’**API .NET** et l’**API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline ayant une activité de copie, consultez le [tutoriel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). Pour accéder à des exemples JSON sur la copie de données d’une source HTTP vers le stockage Blob Azure, consultez [Exemples JSON](#json-examples).

## <a name="linked-service-properties"></a>Propriétés du service lié

Le tableau suivant décrit les éléments JSON qui sont propres au service lié HTTP :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type | La propriété **type** doit être définie sur **Http**. | Oui |
| url | URL de base du serveur web. | Oui |
| authenticationType | Spécifie le type d’authentification. Les valeurs autorisées sont : **Anonyme**, **De base**, **Digest**, **Windows** et **ClientCertificate**. <br><br> Consultez les sections plus loin dans cet article pour accéder à d’autres propriétés et à des exemples JSON relatifs à ces types d’authentification. | Oui |
| enableServerCertificateValidation | Spécifie si la validation du certificat TLS/SSL du serveur doit être activée si la source est un serveur web HTTPS. Quand votre serveur HTTPS utilise un certificat auto-signé, définissez cette propriété sur **false**. | Non<br /> (la valeur par défaut est **true**) |
| gatewayName | Nom de l’instance de passerelle de gestion des données à utiliser pour se connecter à une source HTTP locale. | Oui, dans le cas de copie de données à partir d’une source HTTP locale |
| encryptedCredential | Informations d’identification chiffrées pour accéder au point de terminaison HTTP. La valeur est générée automatiquement quand vous configurez les informations d’authentification dans l’Assistant de copie ou la boîte de dialogue **ClickOnce**. | Non<br /> (s’applique uniquement pour la copie de données à partir d’un serveur HTTP local) |

Pour plus d’informations sur la définition des informations d’identification pour une source de données de connecteur HTTP local, consultez [Déplacement de données entre des sources locales et le cloud à l’aide de la passerelle de gestion des données](data-factory-move-data-between-onprem-and-cloud.md).

### <a name="using-basic-digest-or-windows-authentication"></a>Utilisation de l’authentification Basic (De base), Digest ou Windows

Définissez la valeur **authenticationType** sur **De base**, **Digest** ou **Windows**. Outre les propriétés de connecteur HTTP génériques décrites dans les sections précédentes, définissez les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| userName | Nom d’utilisateur à utiliser pour accéder au point de terminaison HTTP. | Oui |
| mot de passe | Mot de passe de l’utilisateur (**username**). | Oui |

**Exemple : Utilisation de l’authentification De base, Digest ou Windows**

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "basic",
            "url" : "https://en.wikipedia.org/wiki/",
            "userName": "user name",
            "password": "password"
        }
    }
}
```

### <a name="using-clientcertificate-authentication"></a>Utilisation de l’authentification ClientCertificate (Certificat client)

Pour utiliser l’authentification de base, définissez **authenticationType** sur **ClientCertificate**. Outre les propriétés de connecteur HTTP génériques décrites dans les sections précédentes, définissez les propriétés suivantes :

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| embeddedCertData | Contenu encodé en Base64 des données binaires du fichier PFX. | Spécifiez **embeddedCertData** ou **certThumbprint** |
| certThumbprint | Empreinte du certificat qui a été installé dans le magasin de certificats de votre ordinateur de passerelle. S’applique uniquement pour la copie de données à partir d’une source HTTP locale. | Spécifiez **embeddedCertData** ou **certThumbprint** |
| mot de passe | Mot de passe associé au certificat. | Non |

Si vous utilisez **certThumbprint** pour l’authentification et que le certificat est installé dans le magasin personnel de l’ordinateur local, accordez des autorisations de lecture au service de passerelle :

1. Ouvrez MMC (Microsoft Management Console). Ajoutez le composant logiciel enfichable **Certificats** ciblant **Ordinateur local**.
2. Développez **Certificats** > **Personnel**, puis sélectionnez **Certificats**.
3. Cliquez avec le bouton droit sur le certificat du magasin personnel, puis sélectionnez **Toutes les tâches** >**Gérer les clés privées**.
3. Sous l’onglet **Sécurité**, ajoutez le compte d’utilisateur sous lequel le service hôte de la passerelle de gestion des données s’exécute avec l’accès en lecture au certificat.  

**Exemple : Utilisation d’un certificat client**

Ce service lié lie votre fabrique de données à un serveur web HTTP local. Il utilise un certificat client installé sur l’ordinateur doté de la passerelle de gestion des données.

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "certThumbprint": "thumbprint of certificate",
            "gatewayName": "gateway name"
        }
    }
}
```

**Exemple : Utilisation d’un certificat client dans un fichier**

Ce service lié lie votre fabrique de données à un serveur web HTTP local. Il utilise un fichier de certificat client sur l’ordinateur doté de la passerelle de gestion des données.

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "ClientCertificate",
            "url": "https://en.wikipedia.org/wiki/",
            "embeddedCertData": "Base64-encoded cert data",
            "password": "password of cert"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données

Certaines sections d’un fichier JSON de jeu de données, comme la structure, la disponibilité et la stratégie, sont similaires pour tous les types de jeux de données (par exemple, Azure SQL Database, stockage Blob Azure et stockage Table Azure).

Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md).

La section **typeProperties** est différente pour chaque type de jeu de données. La section **typeProperties** fournit des informations sur l’emplacement des données dans la banque de données. La section **typeProperties** pour un jeu de données du type **Http** a les propriétés suivantes :

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type | La propriété **type** du jeu de données doit être définie sur **Http**. | Oui |
| relativeUrl | URL relative de la ressource qui contient les données. Quand le chemin n’est pas spécifié, seule l’URL indiquée dans la définition du service lié est utilisée. <br><br> Pour construire une URL dynamique, vous pouvez utiliser les [variables système et fonctions Data Factory](data-factory-functions-variables.md). Exemple : **relativeUrl**: **$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)** . | Non |
| requestMethod | Méthode HTTP. Les valeurs autorisées sont **GET** et **POST**. | Non <br />(la valeur par défaut est **GET**) |
| additionalHeaders | En-têtes de requête HTTP supplémentaires. | Non |
| requestBody | Corps de la requête HTTP. | Non |
| format | Si vous souhaitez *récupérer les données d’un point de terminaison HTTP en l’état*, sans les analyser, ignorez le paramètre de **format**. <br><br> Si vous souhaitez analyser le contenu de la réponse HTTP pendant la copie, les types de formats suivants sont pris en charge : **TextFormat**, **JsonFormat**, **AvroFormat**, **OrcFormat** et **ParquetFormat**. Pour plus d’informations, consultez [Format Texte](data-factory-supported-file-and-compression-formats.md#text-format), [Format JSON](data-factory-supported-file-and-compression-formats.md#json-format), [Format Avro](data-factory-supported-file-and-compression-formats.md#avro-format), [Format Orc](data-factory-supported-file-and-compression-formats.md#orc-format) et [Format Parquet](data-factory-supported-file-and-compression-formats.md#parquet-format). |Non |
| compression | Spécifiez le type et le niveau de compression pour les données. Types pris en charge : **GZip**, **Deflate**, **BZip2** et **ZipDeflate**. Niveaux pris en charge : **Optimal** et **Fastest**. Pour plus d’informations, consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md#compression-support). |Non |

**Exemple : Utilisation de la méthode GET (par défaut)**

```json
{
  "name": "HttpSourceDataInput",
    "properties": {
    "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
          "relativeUrl": "XXX/test.xml",
          "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

**Exemple : Utilisation de la méthode POST**

```json
{
    "name": "HttpSourceDataInput",
    "properties": {
        "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "/XXX/test.xml",
       "requestMethod": "Post",
            "requestBody": "body for POST HTTP request"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```

## <a name="copy-activity-properties"></a>Propriétés de l’activité de copie

Les propriétés comme le nom, la description, les tables d’entrée et de sortie, et la stratégie, sont disponibles pour tous les types d’activités.

Pour obtenir la liste complète des sections et des propriétés permettant de définir des activités, consultez [Création de pipelines](data-factory-create-pipelines.md). 

Les propriétés disponibles dans la section **typeProperties** de l’activité varient pour chaque type d’activité. Pour une activité de copie, les propriétés dépendent des types de sources et de récepteurs.

Actuellement, quand la source de l’activité de copie est de type **HttpSource**, les propriétés suivantes sont prises en charge :

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| httpRequestTimeout | Délai d’expiration (valeur **TimeSpan**) pour l’obtention d’une réponse par la requête HTTP. Il s’agit du délai d’expiration pour l’obtention d’une réponse, et non du délai d’expiration pour la lecture des données de la réponse. | Non<br />(Valeur par défaut : **00:01:40**) |

## <a name="supported-file-and-compression-formats"></a>Formats de fichier et de compression pris en charge

Consultez [Formats de fichiers et de compression pris en charge dans Azure Data Factory](data-factory-supported-file-and-compression-formats.md) pour plus d’informations.

## <a name="json-examples"></a>Exemples JSON

Les exemples suivants présentent des exemples de définitions JSON que vous pouvez utiliser pour créer un pipeline à l’aide de [Visual Studio](data-factory-copy-activity-tutorial-using-visual-studio.md) ou d’[Azure PowerShell](data-factory-copy-activity-tutorial-using-powershell.md). Les exemples montrent comment copier des données d’une source HTTP vers le stockage Blob Azure. Toutefois, les données peuvent être copiées *directement* d’une source quelconque vers l’un des récepteurs [pris en charge](data-factory-data-movement-activities.md#supported-data-stores-and-formats) à l’aide de l’activité de copie d’Azure Data Factory.

**Exemple : Copier des données d’une source HTTP vers Stockage Blob Azure**

La solution Data Factory pour cet exemple contient les entités Data Factory suivantes :

*   Un service lié de type [HTTP](#linked-service-properties).
*   Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
*   Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [Http](#dataset-properties).
*   Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
*   Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [HttpSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie des données d’une source HTTP vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections qui suivent les exemples.

### <a name="http-linked-service"></a>Service lié HTTP

Cet exemple utilise le service lié HTTP avec l’authentification anonyme. Pour connaître les différents types d’authentification que vous pouvez utiliser, consultez [Service lié HTTP](#linked-service-properties).

```json
{
    "name": "HttpLinkedService",
    "properties":
    {
        "type": "Http",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

### <a name="azure-storage-linked-service"></a>Service lié de stockage Azure

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<account name>;AccountKey=<account key>"
    }
  }
}
```

### <a name="http-input-dataset"></a>Jeu de données d’entrée HTTP

La définition de **external** : **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory et non produit par une activité dans Data Factory.

```json
{
  "name": "HttpSourceDataInput",
    "properties": {
    "type": "Http",
        "linkedServiceName": "HttpLinkedService",
        "typeProperties": {
            "relativeUrl": "$$Text.Format('/my/report?month={0:yyyy}-{0:MM}&fmt=csv', SliceStart)",
        "additionalHeaders": "Connection: keep-alive\nUser-Agent: Mozilla/5.0\n"
    },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}

```

### <a name="azure-blob-output-dataset"></a>Jeu de données de sortie d’objet Blob Azure

Les données sont écrites dans un nouvel objet blob toutes les heures (**fréquence** : **heure**, **intervalle** : **1**).

```json
{
    "name": "AzureBlobOutput",
    "properties":
    {
        "type": "AzureBlob",
        "linkedServiceName": "AzureStorageLinkedService",
        "typeProperties":
        {
            "folderPath": "adfgetstarted/Movies"
        },
        "availability":
        {
            "frequency": "Hour",
            "interval": 1
        }
    }
}
```

### <a name="pipeline-that-uses-a-copy-activity"></a>Pipeline qui utilise une activité de copie

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d’entrée et de sortie. L’activité de copie est planifiée pour s’exécuter toutes les heures. Dans la définition JSON du pipeline, le type **source** est défini sur **HttpSource** et le type **sink** sur **BlobSink**.

Pour obtenir la liste des propriétés prises en charge par le type **HttpSource**, consultez [HttpSource](#copy-activity-properties).

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with a copy activity",
    "activities":[  
      {
        "name": "HttpSourceToAzureBlob",
        "description": "Copy from an HTTP source to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "HttpSourceDataInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "HttpSource"
          },
          "sink": {
            "type": "BlobSink"
          }
        },
       "scheduler": {
          "frequency": "Hour",
          "interval": 1
        },
        "policy": {
          "concurrency": 1,
          "executionPriorityOrder": "OldestFirst",
          "retry": 0,
          "timeout": "01:00:00"
        }
      }
      ]
   }
}
```

> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source à des colonnes d’un jeu de données récepteur, voir [Mappage des colonnes d’un jeu de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Performances et réglage

Consultez le [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.

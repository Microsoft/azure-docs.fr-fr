---
title: Déplacer des données à partir d’une table web avec Azure Data Factory
description: Découvrez comment transférer des données à partir d’une table dans une page web à l’aide d’Azure Data Factory.
services: data-factory
documentationcenter: ''
author: linda33wj
manager: shwang
ms.assetid: f54a26a4-baa4-4255-9791-5a8f935898e2
ms.service: data-factory
ms.workload: data-services
ms.topic: conceptual
ms.date: 01/05/2018
ms.author: jingwang
robots: noindex
ms.openlocfilehash: d2ea038c7d7212529185d77a6ba9e64deacb1c9e
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96012848"
---
# <a name="move-data-from-a-web-table-source-using-azure-data-factory"></a>Déplacer des données depuis une source de table web à l’aide d’Azure Data Factory
> [!div class="op_single_selector" title1="Sélectionnez la version du service Data Factory que vous utilisez :"]
> * [Version 1](data-factory-web-table-connector.md)
> * [Version 2 (version actuelle)](../connector-web-table.md)

> [!NOTE]
> Cet article s’applique à la version 1 de Data Factory. Si vous utilisez la version actuelle du service Data Factory, consultez [Connecteur de table web dans V2](../connector-web-table.md).

Cet article explique comment utiliser l’activité de copie d’Azure Data Factory afin de déplacer les données d’une table dans une page web vers un magasin de données récepteur pris en charge. Cet article s’appuie sur l’article des [activités de déplacement des données](data-factory-data-movement-activities.md) qui présente une vue d’ensemble du déplacement des données avec l’activité de copie et la liste de magasins de données pris en charge comme sources/récepteurs.

Actuellement, Data Factory prend uniquement en charge le déplacement de données depuis une table web vers d’autres magasins de données, mais pas l’inverse.

> [!IMPORTANT]
> Pour l’instant, ce connecteur web prend uniquement en charge l’extraction du contenu d’une table à partir d’une page HTML. Pour récupérer des données à partir d’un point de terminaison HTTP/S, utilisez plutôt le [Connecteur HTTP](data-factory-http-connector.md).

## <a name="prerequisites"></a>Prérequis

Pour utiliser ce connecteur Table web, vous devez configurer un runtime d’intégration auto-hébergé (également appelé passerelle de gestion des données) et la propriété `gatewayName` dans le service lié récepteur. Par exemple, pour effectuer une copie à partir de Table web vers le stockage d’objets Blob Azure, configurez le service lié Stockage Azure comme suit :

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>",
      "gatewayName": "<gateway name>"
    }
  }
}
```

## <a name="getting-started"></a>Prise en main
Vous pouvez créer un pipeline avec une activité de copie qui déplace les données d’un magasin de données Cassandra local à l’aide de différents outils/API. 

- Le moyen le plus simple de créer un pipeline consiste à utiliser **l’Assistant Copie**. Voir le [tutoriel : Créer un pipeline avec l’activité de copie à l’aide de l’Assistant Copie](data-factory-copy-data-wizard-tutorial.md) pour obtenir une procédure pas à pas rapide sur la création d’un pipeline à l’aide de l’Assistant Copie de données. 
- Vous pouvez également utiliser les outils suivants pour créer un pipeline : **Visual Studio**, **Azure PowerShell**, **modèle Azure Resource Manager**, **.NET API** et **API REST**. Pour obtenir des instructions détaillées sur la création d’un pipeline avec une activité de copie, consultez le [didacticiel sur l’activité de copie](data-factory-copy-data-from-azure-blob-storage-to-sql-database.md). 

Que vous utilisiez des outils ou des API, la création d’un pipeline qui déplace les données d’un magasin de données source vers un magasin de données récepteur implique les étapes suivantes :

1. Création de **services liés** pour lier les magasins de données d’entrée et de sortie à votre fabrique de données.
2. Création de **jeux de données** pour représenter les données d’entrée et de sortie de l’opération de copie. 
3. Création d’un **pipeline** avec une activité de copie qui utilise un jeu de données en tant qu’entrée et un jeu de données en tant que sortie. 

Lorsque vous utilisez l’Assistant, les définitions JSON de ces entités Data Factory (services liés, jeux de données et pipeline) sont automatiquement créées pour vous. Lorsque vous utilisez des outils/API (à l’exception de l’API .NET), vous devez définir ces entités Data Factory au format JSON.  Pour un exemple contenant des définitions JSON pour les entités Data Factory servant à copier des données à partir d’une table web, consultez la section [Exemple JSON : copier des données d’une table web vers Stockage Blob Azure](#json-example-copy-data-from-web-table-to-azure-blob) de cet article. 

Les sections suivantes fournissent des informations sur les propriétés JSON utilisées pour définir les entités Data Factory spécifiques d’une table web :

## <a name="linked-service-properties"></a>Propriétés du service lié
Le tableau suivant fournit la description des éléments JSON spécifiques du service lié Web.

| Propriété | Description | Obligatoire |
| --- | --- | --- |
| type |La propriété type doit être définie sur : **Web** |Oui |
| Url |URL de la source web |Oui |
| authenticationType |Anonyme |Oui |

### <a name="using-anonymous-authentication"></a>Utilisation de l’authentification anonyme

```json
{
    "name": "web",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

## <a name="dataset-properties"></a>Propriétés du jeu de données
Pour obtenir une liste complète des sections et propriétés disponibles pour la définition de jeux de données, consultez l’article [Création de jeux de données](data-factory-create-datasets.md). Les sections comme la structure, la disponibilité et la stratégie d'un jeu de données JSON sont similaires pour tous les types de jeux de données (SQL Azure, Azure Blob, Azure Table, etc.).

La section **typeProperties** est différente pour chaque type de jeu de données et fournit des informations sur l’emplacement des données dans le magasin de données. La section typeProperties pour le jeu de données de type **WebTable** a les propriétés suivantes

| Propriété | Description | Obligatoire |
|:--- |:--- |:--- |
| type |Type du jeu de données. Doit avoir la valeur **WebTable** |Oui |
| path |URL relative de la ressource qui contient la table. |Non. Quand le chemin d’accès n’est pas spécifié, seule l’URL spécifiée dans la définition du service lié est utilisée. |
| index |Index de la table dans la ressource. Pour savoir comment obtenir l’index d’une table dans une page HTML, consultez la section [Obtenir l’index d’une table dans une page HTML](#get-index-of-a-table-in-an-html-page) . |Oui |

**Exemple :**

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
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
Pour obtenir la liste complète des sections et des propriétés disponibles pour la définition des activités, consultez l’article [Création de pipelines](data-factory-create-pipelines.md). Les propriétés comme le nom, la description, les tables d’entrée et de sortie et la stratégie sont disponibles pour tous les types d’activités.

En revanche, les propriétés disponibles dans la section typeProperties de l’activité varient pour chaque type d'activité. Pour l’activité de copie, elles dépendent des types de sources et récepteurs.

Actuellement, lorsque la source de l’activité de copie est de type **WebSource**, aucune propriété supplémentaire n’est prise en charge.


## <a name="json-example-copy-data-from-web-table-to-azure-blob"></a>Exemple JSON : copier des données d’une table web vers Stockage Blob Azure
L’exemple suivant montre :

1. Un service lié de type [Web](#linked-service-properties).
2. Un service lié de type [AzureStorage](data-factory-azure-blob-connector.md#linked-service-properties).
3. Un [jeu de données](data-factory-create-datasets.md) d’entrée de type [WebTable](#dataset-properties).
4. Un [jeu de données](data-factory-create-datasets.md) de sortie de type [AzureBlob](data-factory-azure-blob-connector.md#dataset-properties).
5. Un [pipeline](data-factory-create-pipelines.md) avec une activité de copie qui utilise [WebSource](#copy-activity-properties) et [BlobSink](data-factory-azure-blob-connector.md#copy-activity-properties).

L’exemple copie des données d’une table web vers un objet blob Azure toutes les heures. Les propriétés JSON utilisées dans ces exemples sont décrites dans les sections suivant les exemples.

L’exemple suivant indique comment copier des données à partir d’une table web vers un objet blob Azure. Toutefois, les données peuvent être copiées directement vers l’un des récepteurs indiqués dans l’article [Activités de déplacement des données](data-factory-data-movement-activities.md) , par le biais de l’activité de copie d’Azure Data Factory.

**Service lié Web** Cet exemple utilise le service lié Web avec l’authentification anonyme. Consultez la section [Service lié Web](#linked-service-properties) pour connaître les différents types d’authentification que vous pouvez utiliser.

```json
{
    "name": "WebLinkedService",
    "properties":
    {
        "type": "Web",
        "typeProperties":
        {
            "authenticationType": "Anonymous",
            "url" : "https://en.wikipedia.org/wiki/"
        }
    }
}
```

**Service lié Azure Storage**

```json
{
  "name": "AzureStorageLinkedService",
  "properties": {
    "type": "AzureStorage",
    "typeProperties": {
      "connectionString": "DefaultEndpointsProtocol=https;AccountName=<accountname>;AccountKey=<accountkey>",
      "gatewayName": "<gateway name>"
    }
  }
}
```

**Jeu de données d’entrée WebTable** La définition de **external** sur **true** informe le service Data Factory qu’il s’agit d’un jeu de données qui est externe à Data Factory et non produit par une activité dans Data Factory.

> [!NOTE]
> Pour savoir comment obtenir l’index d’une table dans une page HTML, consultez la section [Obtenir l’index d’une table dans une page HTML](#get-index-of-a-table-in-an-html-page) .  
>
>

```json
{
    "name": "WebTableInput",
    "properties": {
        "type": "WebTable",
        "linkedServiceName": "WebLinkedService",
        "typeProperties": {
            "index": 1,
            "path": "AFI's_100_Years...100_Movies"
        },
        "external": true,
        "availability": {
            "frequency": "Hour",
            "interval":  1
        }
    }
}
```


**Jeu de données de sortie d'objet Blob Azure**

Les données sont écrites dans un nouvel objet blob toutes les heures (fréquence : heure, intervalle : 1).

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



**Pipeline avec activité de copie**

Le pipeline contient une activité de copie qui est configurée pour utiliser les jeux de données d'entrée et de sortie, et qui est planifiée pour s'exécuter toutes les heures. Dans la définition du pipeline JSON, le type **source** est défini sur **WebSource** et le type **sink** est défini sur **BlobSink**.

Pour obtenir la liste des propriétés prises en charge par WebSource, consultez propriétés du type WebSource.

```json
{  
    "name":"SamplePipeline",
    "properties":{  
    "start":"2014-06-01T18:00:00",
    "end":"2014-06-01T19:00:00",
    "description":"pipeline with copy activity",
    "activities":[  
      {
        "name": "WebTableToAzureBlob",
        "description": "Copy from a Web table to an Azure blob",
        "type": "Copy",
        "inputs": [
          {
            "name": "WebTableInput"
          }
        ],
        "outputs": [
          {
            "name": "AzureBlobOutput"
          }
        ],
        "typeProperties": {
          "source": {
            "type": "WebSource"
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

## <a name="get-index-of-a-table-in-an-html-page"></a>Obtenir l’index d’une table dans une page HTML
1. Lancez **Excel 2016** et basculez vers l’onglet **Données**.  
2. Cliquez sur **Nouvelle requête** dans la barre d’outils, pointez sur **À partir d’autres sources** et cliquez sur **À partir du web**.

    ![Menu Power Query](./media/data-factory-web-table-connector/PowerQuery-Menu.png)
3. Dans la boîte de dialogue **À partir du web**, entrez l’**URL** que vous utiliseriez dans le service lié JSON (par exemple https://en.wikipedia.org/wiki/), ainsi que le chemin à spécifier pour le jeu de données (par exemple : AFI%27s_100_Years...100_Movies), puis cliquez sur **OK**.

    ![Boîte de dialogue À partir du web](./media/data-factory-web-table-connector/FromWeb-DialogBox.png)

    URL utilisée dans cet exemple : https://en.wikipedia.org/wiki/AFI%27s_100_Years...100_Movies
4. Si la boîte de dialogue **Accéder au contenu web** apparaît, sélectionnez **l’URL** et **l’authentification** adéquates, puis cliquez sur **Se connecter**.

   ![Boîte de dialogue Accéder au contenu web](./media/data-factory-web-table-connector/AccessWebContentDialog.png)
5. Cliquez sur un élément de **table** dans l’arborescence pour afficher le contenu de la table, puis sur le bouton **Modifier** du bas.  

   ![Boîte de dialogue Navigateur](./media/data-factory-web-table-connector/Navigator-DialogBox.png)
6. Dans la fenêtre **Éditeur de requête**, cliquez sur **Éditeur avancé** dans la barre d’outils.

    ![Bouton Éditeur avancé](./media/data-factory-web-table-connector/QueryEditor-AdvancedEditorButton.png)
7. Dans la boîte de dialogue Éditeur avancé, le numéro en regard de « Source » est l’index.

    ![Éditeur avancé - Index](./media/data-factory-web-table-connector/AdvancedEditor-Index.png)

Si vous utilisez Excel 2013, utilisez [Microsoft Power Query pour Excel](https://www.microsoft.com/download/details.aspx?id=39379) pour obtenir l’index. Pour plus d’informations, consultez l’article [Se connecter à une page web](https://support.office.com/article/Connect-to-a-web-page-Power-Query-b2725d67-c9e8-43e6-a590-c0a175bd64d8) . Les étapes sont identiques si vous utilisez [Microsoft Power BI Desktop](https://powerbi.microsoft.com/desktop/).

> [!NOTE]
> Pour savoir comment mapper des colonnes d’un jeu de données source sur des colonnes d’un jeu de données récepteur, consultez [Mappage de colonnes des jeux de données dans Azure Data Factory](data-factory-map-columns.md).

## <a name="performance-and-tuning"></a>Performances et réglage
Consultez l’article [Guide sur les performances et le réglage de l’activité de copie](data-factory-copy-activity-performance.md) pour en savoir plus sur les facteurs clés affectant les performances de déplacement des données (activité de copie) dans Azure Data Factory et les différentes manières de les optimiser.
